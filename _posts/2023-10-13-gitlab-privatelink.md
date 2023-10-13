---
layout: post
title: "Building MITM to secure™ GitLab CI traffic"
---

Disclaimer: I don't like GitLab, not one bit. But if you're unfortunately in a team called "DevOps" where the company uses GitLab, chances are, you are tasked to run it anyway.

---

MITM often describes an [attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) where an attacker inspects or even modifies the traffic between two parties without them noticing.

Why does an attack technique have anything to do with securing GitLab CI? Well, this post exists exactly to explore the reasons that gravitate us towards this solution, and explain how each piece fits together to achieve this (terrible) solution.

## How we secure GitLab today

As the story often goes, our GitLab deployment was publicly accessible from the internet, then it got compromised due to a RCE vulnerability (it was also way out-of-date), so we switched to the current setup where GitLab runs in Kubernetes with tightened security group allowing only "trusted" IPs including office and VPNs. We also prioritize applying security patches and upgrades to GitLab.

Using a security group to protect one service is trivial, but in reality we have many security groups spanning many AWS accounts, and during the period when we were constantly adding new offices and different VPNs, it's simply unmanageable to manually maintain all of them in all the different places.

This leads to the creation of our [cidr-controller](https://github.com/controlant-org/ctrl-cidr), which allows the mapping of a name to a set of CIDRs to be maintained in a single place. The controller then automatically applies the appropriate rules according to the tags declared on security groups (and other supported resources).

The controller works well so far, but from the very beginning we knew there exists [an upper limit](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html#vpc-limits-security-groups) on how many rules a Security Group can have. Given our current scale and the typical time needed to set up new offices, we won't hit the limit on SG rules anytime soon.

However, we also use the cidr-controller to manage access between different AWS accounts, this is especially relevant for GitLab (and some other shared services) which itself runs in one account, while runners in other accounts -for integration tests and DB migrations etc... - also need to access GitLab. So far, this is enabled by tracking their NAT gateway IPs in cidr-controller, which then adds them to the SG GitLab uses. "Fortunately" the existing accounts only has 1 NAT gateway, but still, this multiplies because we need to open multiple ports on GitLab, so these add up to ~20 rules at least.

So in the current state, our GitLab load balancer allows access from our office and VPNs, as well as from any other environments (AWS accounts) in terms of networking. Of course, any such access still requires authentication and authorization.

## Domain accounts and PrivateLink

Due to various reasons, we have started creating (a set of) new accounts for each new team, instead of sharing one (the same set of) account between all teams. While discussing the pros and cons, the limit mentioned above is one of the main talking points, and specifically GitLab is a service new accounts likely need access to.

When creating thew new accounts, we always create 3 NAT gateways, this is because cross-AZ traffic has cost and AWS has suggested to avoid that by having NAT gateways in each AZ. Combine that with the fact each domain has 4 accounts, the numbers quickly explode. Not to mention, if we continue to use cidr-controller for this purpose, we have to manually track each new NAT gateway's IP and add them to the list of command line arguments. So finding an alternative way without public traffic is quite high on our priority list.

A classic solution for internal networking on AWS is [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html), however this requires careful planning of CIDR assignments and ultimately is not scalable. Further research pointed us towards [AWS PrivateLink](https://aws.amazon.com/privatelink/), which provides the same connectivity but is not limited by L3 routing. This gave us confidence in continuing with overlapped VPC CIDR ranges across all accounts.

We fairly quickly managed to set it up successfully for Vault. This allows office and VPN IPs to reach Vault via `https://vault.example.com`, while in any VPC connected by PrivateLink to access through `https://vault.privatelink.example.com`.

The different domain is a requirement for PrivateLink, and as shown next, presents a myriad of challenges for GitLab.

## GitLab Ingress strikes first

While our deployment of Vault only uses the AWS NLB to terminate TLS and use HTTP behind it. GitLab creates another load balancer in the form of [nginx-ingress](https://docs.gitlab.com/charts/charts/nginx/), and also uses self-signed certificates so traffic between nginx and GitLab is "secured" with TLS.

Both factors contribute to making it more complex to use PrivateLink, with Vault the NLB terminates the TLS and the Vault server pods do not care what domain the request is asking originally but just look at the body instead. With GitLab, the nginx-ingress also terminates TLS as well as having various rules to match domains.

In fact, there is [a long-standing open issue on Gitlab](https://gitlab.com/gitlab-org/gitlab/-/issues/21319) about supporting multiple domains, GitLab also responds with redirects in many situations.

All said, our understanding is that we will need to somehow rewrite the incoming request so that GitLab _thinks_ it's requesting `gitlab.example.com` instead of the PrivateLink domain.

While Nginx should have such capabilities, due to it being completely managed as part of the GitLab Helm chart, we eventually pivoted to use a different solution altogether, enter Caddy.

[Caddy](https://caddyserver.com/) is a brilliant tool that provides totally secure hosting with very simple configurations. After years of fronting my own applications (outside work) with Nginx, switching to Caddy made everything so much easier.

I found out it also has [specific support](https://caddyserver.com/docs/quick-starts/reverse-proxy#https-from-proxy-to-backend) for rewriting the `Host` header which is what we're after here.

```
{$DOWNSTREAM_ADDRESS} {
	reverse_proxy {$UPSTREAM_ADDRESS} {
		header_up Host "gitlab.example.com"
		header_down Host "gitlab.privatelink.example.com"
		transport http {
			tls
			tls_insecure_skip_verify
		}
	}
}
```

This configuration now allows us to run `curl https://gitlab.privatelink.example.com` successfully. But remember, our goal is to allow GitLab runners to function while communicating to GitLab via PrivateLink, turns out, this is but just a small step towards that goal.

## Runner forces us to the dark side

When we started to look into enabling runner usage, I quickly realized we cannot simply change the URL it talks to, again not simply like Vault. Even if we can [configure the runner itself to use a different URL](https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-section), for one thing, it won't affect all the [predefined variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) which are populated by the server - `CI_REGISTRY` is of special notice, as we at least use it to pull build time images from GitLab container registry.

Besides the configurations, it's actually more so that we cannot control the "content", i.e. the CI jobs created by other teams. Even if we can require certain changes to be made to accommodate connecting via PrivateLink domains, it would be very messy to identify where it applies and very hard to maintain in the long run.

This puts us in an awkward position, PrivateLink requires a unique domain, we can't change that, but we want to maintain the "illusion" that CI jobs are just talking to `gitlab.example.com` like in any other environment.

We figured the only viable solution is by messing with DNS, the idea is that when anything tries to talk to `gitlab.example.com`, the DNS responds with entries of the PrivateLink domain (`gitlab.privatelink.example.com`) instead. We're fully aware of how terrible this solution is, altering DNS responses can easily result in very obscure bugs, and more often than not very confusing to debug. The only justification is that since we only intend to do this for CI purposes, it (hopefully) won't break any production workloads, but I guess only time will tell.

With a solution in mind, we searched around, AWS has [Route53 Resolver](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html) but eventually we found [CoreDNS supports rewrite](https://coredns.io/plugins/rewrite/). CoreDNS is used by Kubernetes as DNS servers for pods, so that applies to GitLab runners (as we intend to support). The rewrite rule is easy enough to configure, so we just added a few lines to the CoreDNS configmap:

```diff
 data:
   Corefile: |
     .:53 {
         errors
         health
+        rewrite stop {
+          name substring gitlab.example.com gitlab.privatelink.example.com answer auto
+        }
+        rewrite stop {
+          name substring registry.example.com gitlab-registry.privatelink.example.com answer auto
+        }
         kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
         }
         prometheus :9153
         forward . /etc/resolv.conf
         cache 30
         loop
         reload
         loadbalance
     }
```

Around this time we also realized our GitLab registry is using a different domain, so we created a second Caddy / PrivateLink setup similiar to the one for GitLab itself, but for `registry.example.com`, that's why there's an additional rewrite rule.

Now we can `curl https://gitlab.example.com` just fine from a debug pod. Time to try with real™ jobs.

## Container registry stabs the nodes in the back

It's straightforward to add a new runner in our development environment, and we added a simple "hello world" job to test the new runner setup.

Even though the runner controller comes up fine, the job soon fails:

> WARNING: Failed to pull image with policy "": image pull failed: rpc error: code = DeadlineExceeded desc = failed to pull and unpack image "registry.example.com/example/container:v1": failed to resolve reference "registry.example.com/example/container:v1": failed to do request: Head "https://registry.example.com/v2/example/container/manifests/v1": dial tcp 1.2.3.4:443: i/o timeout

At first I was a bit confused because several test on `registry.example.com` works fine, but then my colleague pointed out that IP is the public IP of the registry, so it's not using our modified DNS server, and I soon realized this is not from a pod, this is from a Kubernetes node itself! Of course, the node needs to pull the image first, before firing it up for a pod.

So we know the node itself does not use CoreDNS, which makes some sense as otherwise it could be some chicken and egg problems. At this point we thought about Route53 Resolvers again, but given we run [Bottlerocket OS](https://bottlerocket.dev/) and it offers several configurations that can be easily modified as part of [user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-add-user-data.html), I decide to shop around there first.

The "obvious" solution is to set the [servers](https://bottlerocket.dev/en/os/1.15.x/api/settings/dns/#name-servers) directly, this may seem counter-intuitive as I just talked about potential chicken and egg problem, however, we create dedicated [provisioner](https://karpenter.sh/docs/concepts/provisioners/) to run runners - basically a separate pool of Kubernetes nodes, and since CoreDNS runs on a different set of nodes, it should not be a problem. What turns out to be a problem, though, is that the nodes actually cannot boot up, I suspect this is because the node needs to run some pods to configure the networking before it can actually reach the CoreDNS service, so I guess it's a different ordering issue in the end. However, I did not fully confirm this.

At this point we kind of hit a dead-end, we do not want to use the Resolver as it's potentially affecting the entire VPC, and also it's not clear how that actually works. Looking around the docs a bit more, I noticed [another option for configuring container registry mirrors](https://bottlerocket.dev/en/os/1.15.x/api/settings/container-registry/#mirrors), starring at it a bit more and I thought it might work by setting the mirror the the registry's PrivateLink domain. So adding the user data:

```
[[settings.container-registry.mirrors]]
registry = "registry.example.com"
endpoint = ["https://gitlab-registry.privatelink.example.com"]
```

And now we get a different error!


> 0s          Warning   Failed                 Pod/runner-jwcvnpwno-project-3719-concurrent-0-8zpbi5qt   Failed to pull image "registry.example.com/example/container:v1": rpc error: code = DeadlineExceeded desc = failed to pull and unpack image "registry.example.com/example/container:v1": failed to resolve reference "registry.example.com/example/container:v1": failed to authorize: failed to fetch oauth token: Post "https://gitlab.example.com/jwt/auth": dial tcp 1.2.3.4:443: i/o timeout


Even though it might look almost the same, the last part - the "root error" - is entirely different, this was another step towards success so I was naturally excited, even though I'm also confused by the error at the same time - why would it try to authenticate, shouldn't it already done so with the runner token?

Some quick research (a.k.a. Google search) soon revealed this is the [intended auth flow](https://docs.docker.com/registry/spec/auth/token/#how-to-authenticate) with "V2 Registry". So now the node was instructed by GitLab (container registry) to authenticate, obviously GitLab itself is unaware of the fact we're sneaking through via PrivateLink, so it just uses the domain it thinks it's serving.

Again, note that we're still on the Kubernetes node, which does NOT use CoreDNS, so it does not resolve `gitlab.example.com` to the PrivateLink domain, and thus the request fails.

However, since the "instruction" from GitLab is present in a header, it's "easy" to [modify that with Caddy](https://caddyserver.com/docs/caddyfile/directives/header) - remember, the node contacts GitLab container registry through our PrivateLink which is ultimately handled by Caddy, adding these:

```diff
 {$DOWNSTREAM_ADDRESS} {
+	header {
+		>Www-Authenticate gitlab.example.com gitlab.privatelink.example.com
+	}
 	reverse_proxy {$UPSTREAM_ADDRESS} {
 		header_up Host "registry.example.com"
 		header_down Host "gitlab-registry.privatelink.example.com"
 		transport http {
 			tls
 			tls_insecure_skip_verify
 		}
 	}
 }
```

And it all works!

## Is it worth it?

So there you have it, all the hacks just to make it possible to reach GitLab without having to track public NAT gateway IPs.

I must say at each step, we have been asking the question "is it worth it??" with increasing numbers of question marks. Header rewriting is maybe a necessary evil, but DNS rewriting is definitely against our will, even at such a limited scale.

Given the technical limitation that exists on AWS side, I still think this is a viable solution, for the sake of scale. However, I'd struggle to say this is a sustainable solution, say if more systems resembles GitLab when using PrivateLink, then we should look for other ways.

An "obvious solution" is to open GitLab back to the public internet. Even though we ourselves advocate for zero-trust, my justification for not doing so is that we do not develop GitLab, and we don't have enough man-power to operate GitLab around the clock. This means we are at the mercy of when/if GitLab releases security fixes so until we have dedicated resources to proactively fix potential security issues, it's better to reduce the exposure.

It's quite possible we're missing something very obvious here, and don't need many of these hacks? Let me know if you have any ideas!
