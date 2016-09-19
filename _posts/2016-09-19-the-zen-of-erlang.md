---
layout: post
title: Erlang 之禅
cn: true
---

> 本文是在原作者 Fred Hebert 先生的许可下, 对 [_The Zen of Erlang_](http://ferd.ca/the-zen-of-erlang.html) 的简体中文翻译.
>
> 我从原文中获益良多, 也曾受其启发在 [Elixir Shanghai](http://www.meetup.com/Elixir-Shanghai/) 的[第二次聚会](http://www.meetup.com/Elixir-Shanghai/events/232775992/)上分享了一个题为 [_Defensive Programming vs. Let It Crash_ 的演讲](https://speakerdeck.com/aquarhead/defensive-programming-vs-let-it-crash), 从另一个角度切入分享了我的一些感悟.
>
> 虽说本文对所有想了解 Erlang 的人都十分值得一读, 我个人认为对那些接触了 Erlang 或 Elixir 却觉得没有以 Erlang/OTP 的思路在架构程序的人们或许更有用.
>
> 文中原作者的一些比喻也许有些冗长 - 我的翻译也深受文字功底所限, 很是拙劣生硬, 所以那些部分随便看看就好不必较真 - 但有关 Erlang/OTP 的部分都很有仔细研读的价值.
>
> 衷心希望这篇译文能对中文世界的｢编程者｣们有所帮助.

原文发表于 2016年02月08日

The Zen of Erlang

# Erlang 之禅

This is a loose transcript (or long paraphrasing?) of a presentation given at ConnectDev'16, a conference organized by Genetec in which I was invited to speak.

我在 Genetec 组织的 ConnectDev'16 上做过一个演讲, 本文是那次演讲不怎么精确的文本化(或许该说是一篇更详尽的注解?)

![The Zen of Erlang](/static/zen_of_erlang/001.png)

I assume most people here have never used Erlang, have possibly heard of it, maybe just the name. As such, this presentation will only cover the high level concepts of Erlang, in such a way that it may be useful to you in your work or side projects even if you never touch the language.

那时我假定听讲的人大多从没用过 Erlang, 也许有人听说过这个语言, 又或者就知道有这么个名字. 所以这篇演讲只涵盖了 Erlang 一些高层次的概念, 不过其中的一些｢禅理｣或许也会对你们的日常工作或是自己的业余项目有所帮助, 哪怕你并不使用这门语言.

![Let It Crash](/static/zen_of_erlang/002.png)

If you've ever looked at Erlang before, you've heard about that "Let it crash" motto. My first encounter with it had me wondering what the hell this was about. Erlang was supposed to be great for concurrency and fault tolerance, and here I was being told to let things crash, the entire opposite of what I actually want to happen in a system. The proposition is surprising, but the "Zen" of Erlang is related to it directly nonetheless.

假如你曾经看过 Erlang, 你肯定听说过那句｢随他崩溃｣("Let it crash", 下文保持英文)的座右铭. 我第一次听到这句话的时候就在想这什么鬼. Erlang 不是强在高并发, 高容错性等等这些方面吗, 怎么能允许随便崩溃这种事, 这不是我最不希望看到的事故么. 的确, 这句话听起来很出人意料, 不过 Erlang 之｢禅｣还真和它脱不开干系.

![Blow it Up](/static/zen_of_erlang/003.png)

In some ways it would be as funny to use 'Let it Crash' for Erlang as it would be to use 'Blow it up' for rocket science. 'Blow it up' is probably the last thing you want in rocket science — the Challenger disaster is a stark reminder of that. Then again, if you look at it differently, rockets and their whole propulsion mechanism is about handling dangerous combustibles that can and will explode (and that's the risky bit), but doing it in such a controlled manner that they can be used to power space travel, or to send payloads in orbit.

在某些方面 Erlang 的 "Let it crash" 听起来就和搞火箭科学的人说｢让它爆炸吧｣一样滑稽. 那本该是你最不希望发生的事 - 想想｢挑战者｣号曾发生的悲剧吧. 但话说回来, 若从另一个角度来看, 火箭本身以及所有这些推进原理正是关于如何控制那些极其危险的易燃易爆物质, 进而才能利用它们驱动太空旅行和运输.

The point here is really about control; you can try and see rocket science as a way to properly harness explosions — or at least their force — to do what we want with them. Let it crash can therefore be seen under the same light: it's all about fault tolerance. The idea is not to have uncontrolled failures everywhere, it's to instead transform failures, exceptions, and crashes into tools we can use.

其中的关键就在于控制; 你可以试着将火箭科学看作一门通过合理利用爆炸 - 起码有推力 - 来达成目的的学问. 所以 "Let it crash" 也是类似的: 这正是关乎容错率的信条. 我们真正的目标不是去随处制造那些不受控制的崩溃, 而是将故障, 异常还有崩溃等等这些变成我们可以利用的工具.

![Fight Fire with Fire](/static/zen_of_erlang/004.png)

Back-burning and controlled burns are a real world example of fighting fire with fire. In Saguenay–Lac-Saint-Jean, the region I come from, blueberry fields are routinely burnt down in a controlled manner to help encourage and renew their growth. To prevent forest fires, it is fairly frequent to see unhealthy parts of a forest cleaned up with fire, so that it can be done under proper supervision and control. The main objective there is to remove combustible material in such a way an actual wildfire cannot propagate further.

向后引燃法和受控引燃法 [^1] 是现实世界中以火制火的实例. 在我的家乡, 人们会定期燃烧蓝莓田, 用一种可控的方式来促进其生长. 为了预防森林大火, 树林中不健康的部分也会时常被人们主动烧掉, 当然也是在专门的监管和控制之下进行的. 这样就算大火发生了也会因为缺少可燃物而无法扩散太远.

In all of these situations, the destructive power of fire going through crops or forests is being used to ensure the health of the crops, or to prevent a much larger, uncontrolled destruction of forested areas.

在上面所有这些情况里, 火焰那本来极具破坏性的能量却可以用于保障作物的健康, 或是预防规模更大, 亦无法控制的森林大火.

I think this is what 'Let it crash' is about. If we can embrace failures, crashes and exceptions and do so in a very well-controlled manner, they stop being this scary event to be avoided and instead become a powerful building block to assemble large reliable systems.

我想这差不多就是 "Let it crash" 的含义. 如果我们能以一种更有掌控力的姿态来面对故障, 异常或是崩溃, 它们便不再是什么让人害怕的事件. 我们便无需竭力避免其发生, 反而可以以其为基石, 构建更大并且更稳定的系统.

![Processes / Bees](/static/zen_of_erlang/005.png)

So the question becomes to figure out how do we ensure that crashes are enablers rather than destructors. The basic game piece for this in Erlang is the process. Erlang's processes are fully isolated, and they share nothing. No process can go and reach into another one's memory, or impact the work it's doing by corrupting the data it operates on. This is good because it means that a process dying is essentially guaranteed to keep its issues to itself, and that provides very strong fault isolation into your system.

那我们怎么让崩溃之类的｢弃恶从善｣呢? Erlang 提供了最基本的拼板: Erlang 进程 [^2]. Erlang 进程彼此完全独立, 不共享任何数据. 一个进程无法查看或是篡改其他进程的内存, 也就无法干扰其他进程正在进行的工作. 这样的好处是基本上保证了一个进程出问题时只会影响到自己, 同时这就为你的系统提供了很强的故障隔离性.

Erlang's processes are also extremely lightweight, so that you can have thousands and thousands of them without problem. The idea is to use as many processes as you need, rather than as many as you can. The common comparison there is to say that if you had an object-oriented language where you could only have 32 objects running at a any given time, you'd rapidly find it overly constraining and quite ridiculous to build programs in that language. Having many small processes does ensure a higher granularity in how thing break, and in a world where we want to harness the power of these failures, this is good!

Erlang 进程也非常轻量, 启动成千上万个稀松平常. 重要的是你可以想用多少进程就用多少, 而非受限于系统可以承受多少. 你可以这样试着比较一下, 假如在一门面向对象的语言中你最多只能同时有 32 个对象, 你肯定很快就会觉得完全不够用了. 使用大量且轻量的进程使我们能以更细的粒度处理故障, 对于我们想要反过来利用故障和崩溃之类的也非常有用.

Now it can be a bit weird to picture how these processes work exactly. When you write a C program, you have one big main() function that does a lot of stuff. This is your entry point into the program. In Erlang, there is no such thing. No process is the designated master of the program. Every one of them runs a function, and that function plays the role of main() within that single process.

你可能会对这么多进程究竟是如何一起工作的有所疑问. 比方说用 C 语言写程序的时候, 常常会在一个大的 `main()` 函数里面写很多代码. 我们称之为程序入口. Erlang 里面没有这样的概念. 没有哪个进程是控制整个系统的. 虽然每个进程还是在运行某个函数, 但这个函数只在那个进程本身当中扮演 `main()` 函数的角色.

We now have that swarm of bees, but it is probably very hard to direct them to strengthen the hive if they cannot communicate in any way. Where bees dance, Erlang processes pass messages.

我们现在有了蜂群, 但如果它们无法彼此交流就很难指挥它们一同构筑蜂巢. 蜜蜂通过舞蹈来交流, Erlang 进程则通过传递｢消息｣.

![Message Passing](/static/zen_of_erlang/006.png)

Message passing is the most intuitive form of communication in a concurrent environment there is. It's the oldest one we've worked with, from the days where we wrote letters and sent them via couriers on horse to find their destination, to fancier mechanisms like the Napoleonic semaphores shown on the slide. In this case you'd just take a bunch of guys up into towers, give them a message, and they'd wave flags around to pass data over long distances in ways that were faster than horses, which could tire. This eventually got replaced by the telegraph, which got replaced by the phone and the radio, and now we have all the fancy technologies to pass messages really far and really fast.

消息传递是并发环境下最为直观的通讯方式. 这也是人类最古老的通讯方式, 从骑着马的信使带着我们的信件奔向目的地, 到一些更酷的像是图片里面的｢拿破仑旗语｣之类的方法. 马匹终究会疲惫, 但旗语却可以快速地将一条消息传递很长的距离. 当然了, 后来这些渐渐被电报所取代, 然后是无线电和电话, 如今我们有了更多更先进的技术, 能够以前所未有的速度向难以想像的远方传递消息.

A critical aspect of all this message passing, especially in the olden days, is that everything was asynchronous, and with the messages being copied. No one would stand on their porch for days waiting for the courier to come back, and no one (I suspect) would sit by the semaphore towers waiting for responses to come back. You'd send the message, go back to your daily activities, and eventually someone would tell you you got an answer back.

这种消息传递的方式很重要的一点 - 特别是在旧时代 - 就是大家都是｢异步｣的, 只有消息被｢复制｣了. 一般没人会一直站在门口等待回信, 也不会有人一直守在信号塔上等待回复. 你发出一条消息, 接着干平常的事情, 当有回复的时候自会有人通知你.

This is good because if the other party doesn't respond, you're not stuck doing nothing but waiting on your porch until you die. Conversely, the receiver on the other end of the communication channel does not see the freshly arrived message vanish or change as by magic if you do die. Data should be copied when messages are sent. These two principles ensure that failure during communication does not yield a corrupted or unrecoverable state. Erlang implements both of these.

这种方式的好处在于如果对方迟迟没有回应, 你不会什么也不做一直等到世界末日. 同样的, 即使发信人突然死掉了, 收信人所收到的信息也不会因此消失或者被什么魔法篡改. 这两条准则可以保证通信时的故障不会产生损坏的或无法恢复的状态. Erlang 将这两条准则都实现了.

To read messages, each process has a single mailbox. Everyone can write to a process' mailbox, but only the owner can look into it. The messages are by default read in the order they arrived, but it is possible, through features such as pattern matching [which were discussed in a prior talk during the day] to only or temporarily focus on one kind of message and drive various priorities around.

每个进程都会从自己的｢信箱｣中读取消息. 一个进程可以向任意其他进程的信箱中写入消息, 但只能读取自己的信箱中收到的消息. 默认情况下进程会按照消息被收到的顺序依次读取, 但也可以通过模式匹配 (pattern matching [^3]) 之类的方式来重点关注某些消息.

![Links & Monitors](/static/zen_of_erlang/007.png)

Some of you will have noticed something in what I have mentioned so far; I keep repeating that isolation and independence are great so components of a system are allowed to die and crash without influencing others, but I did also mention having conversations across many processes or agents.

可能在座的有些人从我刚刚所讲过的内容中听出了什么端倪. 我一直反复地讲隔离性和独立性多么多么的好, 它们能让一个系统的各个组件崩溃挂掉却不会影响到其他部分. 但同时我又讲了这么多进程之间该如何通信.

Every time two processes start having a conversation, we create an implicit dependency between them. There is some implicit state in the system that binds both of them together. If process A sends a message to process B and B dies without responding, A can either wait forever, or give up on having a conversation after a while. The latter is a valid strategy, but it's a very vague one: it is unclear if the remote end has died or is just taking long, and off-band messages can land in your mailbox.

每当两个进程之间有所通信的时候, 我们实际上就隐式地在两者之间创建了｢依赖｣. 这种依赖可以说在系统中也留下了隐式的｢状态｣. 如果进程 A 发给进程 B 一条消息, 然而 B 还没回应就挂掉了, 这时候 A 有两种选择, 要么一直等下去, 要么等一段时间后就放弃. 一般来说后面这种比较可取, 但这种策略还不够明确: 即我们无法确切地得知究竟 B 真的挂掉了还是只是这一条消息要花比较久才能够回复, 如果只是计算较慢可能在 A 放弃后 B 才发来回复, 这种情况下这条回复就会永远留在 A 的信箱里无人处理了.

Instead Erlang gives us two mechanisms to deal with this: monitors and links.

不过无需担心, Erlang 提供了两种机制来处理这种情况: 监控 (monitor) 和链接 (link).

Monitors are all about being an observer, a creeper. You decide to keep an eye on a process, and if it dies for whatever reason, you get a message in your mailbox telling you about it. You can then react to this and make decisions with your newly found information. The other process will never have had an idea you were doing all of this. Monitors are therefore fairly decent if you're an observer or care about the state of a peer.

监控诚如其名, 就是让一个进程变成观察者. 你可以一直关注某个进程, 一旦它挂掉了, 你就会收到一条包含具体原因的消息. 之后你就可以根据消息中提供的信息决定要采取些什么对策. 被监控的进程完全不会知晓你所在做的这些事. 所以如果你在意某个进程的死活, 监控是一种蛮不错的手段.

Links are bidirectional, and setting one up binds the destiny of the two processes between which they are established. Whenever a process dies, all the linked processes receive an exit signal. That exit signal will in turn kill the other processes.

链接则是双向的, 设置了链接的两个进程的｢命运｣将紧密相连. 一旦其中一个进程挂掉, 所有与其建立了链接的进程都会收到退出信号, 从而跟随这个进程一同死掉.

Now this gets to be really interesting because I can use monitors to quickly detect failures, and I can use links as an architectural construct letting me tie multiple processes together so they fail as a unit. Whenever my independent building blocks start having dependencies among themselves, I can start codifying that into my program. This is useful because I prevent my system from accidentally crashing into unstable partial states. Links are a tool letting developers ensure that in the end, when a thing fails, it fails entirely and leaves behind a clean slate, still without impacting components that are not involved in the exercise.

现在我们可以用监控来快速检测故障, 还可以用链接作为一种组建架构的手段, 将若干个进程绑定在一起, 有故障时一齐杀掉. 当系统中原本独立的模块开始彼此产生依赖时, 我可以将这种依赖写入程序代码中. 而这可以防止系统由于意外的崩溃进入不稳定的或不完整的状态. 链接作为一种工具可以让开发者确保当系统的一部分有故障时, 这一部分会整个关闭而不会污染整个系统的状态, 同时也不会殃及与这部分本不相关的其他组件.

For this slide, I picked a picture of mountain climbers roped together. Now if mountain climbers only had links between them, they would be in a sorry state. Every time a climber from your team would slip, the rest of the team would instantly die. Not a great way to go about things.

这一页的图片是几个以绳子互相绑在一起的登山者们. 假如说这些登山者们只有链接的话那就太悲剧了. 因为一旦一个人打滑了可能整个团队都要死于非命. 这听起来可不怎么好.

Erlang instead lets you specify that some processes are special and can be flagged with a `trap_exit` option. They can then take the exit signals sent over links and transform them into messages. This lets them recover faults and possibly boot a new process to do the work of the former one. Unlike mountain climbers, a special process of that kind cannot prevent a peer from crashing; that is the responsibility of that peer by using `try ... catch` expressions, for example. A process that traps exits still has no way to go play in another one's memory and save it, but it can avoid dying of it.

实际上 Erlang 可以让你指定一些特殊的进程, 通过一个叫 `trap_exit` 的选项. 这些特殊进程可以捕获链接所导致的退出信号, 并把它们转换为消息. 从而它们可以修复一些故障, 比如说启动新的进程来继续挂掉的进程的工作. 不像真正的登山者那样, 这样的特殊进程并不能阻止与其链接的进程崩溃 - 要阻止崩溃你可能需要写类似 `try ... catch` 之类的语句, 当然是在可能会崩溃的那个进程里 - 一个激活了｢捕获退出信号｣的特殊进程不能进入其他进程的内存然后阻止其崩溃, 但却可以避免它｢灭亡｣.

This turns out to be a critical feature to implement supervisors. If you haven't heard of them, we'll get to them soon enough.

这是实现｢监督者｣ (supervisor) 的一个重要特性. 别着急我们马上就会聊到它们了.

![Preemptive Scheduling](/static/zen_of_erlang/008.png)

Before going to supervisors, we still have a few ingredients to be able to successfully cook a system that leverages crashes to its own benefit. One of them is related to how processes are scheduled. For this one, the real world use case I want to refer to is Apollo 11's lunar landing.

讲解监督者之前, 我们还需要再聊几样东西, 少了它们还是没法真正地｢化崩溃为神奇｣. 首先是进程如何调度, 我就拿阿波罗11的月球登陆来举例吧.

Apollo 11 is the mission that went to the moon in '69. In the slide right there, we see the lunar module with Buzz Aldrin and Neil Armstrong on board, with a photo taken by a person I assume to be Michael Collins, who stayed in the command module for the mission.

阿波罗11是69年的一项登月任务. 图片中我们看到巴兹·奥尔德林和尼尔·阿姆斯特朗所乘坐的登月舱, 照片本身应该是留在指令舱的迈克尔·科林斯拍下的.

While on their way to land on the moon, the lunar module was being guided by the Apollo PGNCS (Primary Guidance, Navigation and Control System). The guidance system had multiple tasks running on it, taking a carefully accounted for number of cycles. NASA had also specified that the processor was only to be used to 85% capacity, leaving 15% free.

登月舱飞往月球时应该是由 Apollo PGNCS (Primary Guidance, Navigation and Control System) 所引导的. 导航系统要同时运行若干个任务, 每个任务都有严格分配的循环数. NASA 特别指定了处理器只能用85%的运算力, 剩余的15%以应对紧急情况.

Now because the astronauts in this case wanted a decent backup plan in case they needed to abort, they had left a rendezvous radar up in case it would come in handy. That took up a decent chunk of the capacity the CPU had left. As Buzz Aldrin started entering commands, error messages would pop up about overflow and basically going out of capacity. If the system was going haywire on this, it possibly couldn't do its job and we could end up with two dead astronauts.

宇航员们想制订一个比较稳妥的备用计划来应对任务中止的情况, 他们预留了一个也许会有用的交会雷达. 那家伙用了 CPU 剩余性能的一大部分. 当巴兹·奥尔德林开始输入命令的时候, 各种关于溢出和资源耗尽的错误信息层出不穷. 假如这时候系统失去控制的话, 可能其他组件就无法完成它们的任务, 最后我们就要面对两具宇航员的尸体了.

This was mostly because the radar had known hardware bugs causing its frequency to be mismatched with the guidance computer, and caused it to steal far more cycles than it should have had otherwise. Now NASA people weren't idiots, and they reused components with which they knew the rare bugs they had rather than just greenfielding new tech for such a critical mission, but more importantly, they had devised priority scheduling.

这主要是因为那个雷达有已知的硬件问题, 导致其频率和导航电脑的不符, 进而导致其使用了远多于应该使用的 CPU 循环. 不过 NASA 的人也不是笨蛋, 他们没有为这种重量级的任务去开发过多的新技术, 而是再利用了以前的组件, 那些就算是稀有故障他们也了如指掌的东西. 更重要的是, 他们发明了｢优先级调度｣.

This meant that even in the case where either this radar or possibly the commands entered were overloading the processor, if their priority were too low compared to the absolutely life-critical stuff, the task would get killed to give CPU cycles to what really, really needed it. That was in 1969; today there's still plenty of languages or frameworks that give you only cooperative scheduling and nothing else.

这意味着导致了处理器过载的不论是这个有故障的雷达或者甚至是宇航员所输入的命令, 只要其优先级与那些绝对重要的东西比起来足够低的话, 系统就会杀掉这些任务, 而给那些真正需要的任务腾出 CPU 循环. 那还是 1969 年; 时至今日还有好多语言或框架只提供了协同调度.

Erlang is not a language you'd use for life-critical systems — it only respects soft-real time constraints, not hard real time ones and it just wouldn't be a good idea to use it in these scenarios. But Erlang does provide you with preemptive scheduling, and with process priorities. This means that you do not have to care, as a developer or system designer, about making sure that absolutely everyone carefully counts all the CPU usage they're going to be doing across all their components (including libraries you use) to ensure they don't stall the system. They just won't have that capacity. And if you need some important task to always run when it must, you can also get that.

虽然你不应该用 Erlang 去编写这些关乎性命的系统 - Erlang 只遵循了软实时系统的约束, 而非硬实时, 所以这些情景下不要用 Erlang. 但 Erlang 确实提供了抢占式调度, 以及进程优先级. 这意味着对于系统设计者来说你无需仔细确认每个组件(包括你用的库)所使用的 CPU 资源, 它们不会导致整个系统挂起. 同时假如你真的需要, 也可以让一些重要的任务要优先运行.

This may not seem like a big or common requirement, and people still ship really successful projects only with cooperative scheduling of concurrent tasks, but it certainly is extremely valuable because it protects you against the mistakes of others, and also against your own mistakes. It also opens up the door to mechanisms like automated load-balancing, punishing or rewarding good and bad processes or giving higher priorities to those with a lot of work waiting for them. Those things can end up giving you systems that are fairly adaptive to production loads and unforeseen events.

这些可能听起来不是什么很重要的需求, 有很多成功的项目构建在只有协同调度的并发任务之上, 但抢占式调度也有很大的价值, 它可以从别人乃至你自己的错误中保护整个系统. 它还可以帮助你构建类似自动负载均衡, 好坏进程的惩罚奖赏或是提高那些任务很多的进程的优先级等等. 这些功能可以让你的系统更加从容地面对生产环境中变幻莫测的负载或无法预见的问题.

![Network Aware](/static/zen_of_erlang/009.png)

The last ingredient I want to discuss in getting decent fault tolerance is network awareness. In any system we develop that we need to stay up for long periods of time, having more than one computer to run it on quickly becomes a prerequisite. You don't want to be sitting there with your own golden machine locked behind titanium doors, unable to tolerate any disruption with effecting your users in major ways.

有关构建高容错性系统我最后想说的一点是｢网络认知｣ (network awareness). 在我们想要构建的任何要保持长期在线的系统中, 多台机器构建的网络都可说已成为了前提条件. 哪怕你有一台性能超强的机器, 当它出现问题的时候也就会不可避免地影响到你的用户.

So you eventually need two computers so one can survive a broken other, and maybe a third one if you want to deploy with broken computers part of your system.

所以你最少也要两台机器, 这样一台有问题的时候另一台可以顶上. 假如说部署的时候就有坏掉的机器, 那你可能就要三台才能保证系统的正常运转.

The plane on the slide is the F-82 twin mustang, an aircraft that was designed during the second world war to escort bombers over ranges most other fighters just couldn't cover. It had two cockpits so that pilots could take over and relay each other over time when tired; at some point they also fit it so one would pilot while the other would operate radars in an interceptor role. Modern aircrafts still do something similar; they have countless failovers, and often have crew members sleeping in transit during flight time to make sure there's always someone who's alert ready to pilot the plane.

这一页上的飞机是一架 F-82 ｢双生野马｣, 这种二战时期设计出来的重型战斗机用于掩护长距离的轰炸机, 当时绝大多数其他战斗机都没法非这么长的距离. 它有两个驾驶舱, 因此飞行员们可以轮流休息; 或者可以一组操纵飞机, 另一组操作雷达, 变成类似拦截机的角色. 现代的飞机其实也有类似之处, 它们有数不胜数的故障转移机制, 也常常有可以换班的飞行员在舱内休息, 以便随时有清醒的人可以应对紧急情况.

When it comes to programming languages or development environments, most of them are designed ignoring distribution altogether, even though people know that if you write a server stack, you need more than one server. Yet, if you're gonna use files, there's gonna be stuff in the standard library for that. The furthest most languages go is giving you a socket library or an HTTP client.

尽管人们都知道写服务器的话怎么也要几台机器, 很多语言或平台依然忽略着分布式的功能. 不像文件操作, 大多数语言都有标准库之类的可以直接使用, 大多数语言最多就是提供套接字库或者 HTTP 客户端.

Erlang acknowledges the reality of distribution and gives you an implementation for it, which is documented and transparent. This lets people set up fancy logic for failing over or taking over applications that crash to provide more fault tolerance, or even lets other languages pretend they are Erlang nodes to build polyglot systems.

Erlang 不但认识到分布式的重要性, 还提供了一整套文档完善且透明的实现. 你可以在此之上控制如何故障转移, 或由另一个节点接管某个崩溃的应用等等从而实现更高的容错性. 甚至还可以让其他语言接入 Erlang 的分布式系统中, 构建一个多种语言混合的系统.

![Let it crash.](/static/zen_of_erlang/010.png)

So those are all the basic ingredients in the recipe for Erlang Zen. The whole language is built with the purpose of taking crashes and failures, and making them so manageable it becomes possible to use them as a tool. Let it crash starts making sense, and the principles seen here are for the most part things that can be reused as inspiration in non-Erlang systems.

那么这些就是 Erlang ｢禅宗｣的一些基本要素. 整个语言构建在处理崩溃之上, 将它们变的如此可控从而可以当作一种组建系统的工具. Let it crash 开始有那么点儿道理了, 这里面的一些原理也可以作为其他非 Erlang 系统的灵感.

How to compose them together is the next challenge.

如果将所有这些要素融合在一起是下一个挑战.

![Supervision Trees](/static/zen_of_erlang/011.png)

Supervision trees is how you impose structure to your Erlang programs. They start with a simple concept, a supervisor, whose only job is to start processes, look at them, and restart them when they fail. By the way, supervisor are one of the core components of 'OTP', the general development framework used in the name 'Erlang/OTP'.

监督树是在 Erlang 程序中添加结构的方式. 它们从监督者这个简单的概念开始, 还记得吧, 监督者唯一的工作就是启动子进程, 监控它们, 当它们出故障时重启. [^4]

The objective of doing that is to create a hierarchy, where all the important stuff that must be very solid accumulate closer to the root of the tree, and all the fickle stuff, the moving parts, accumulate at the leaves of the tree. In fact, that's what most trees look like in real life: the leaves are mobile, there's a lot of them, they can all fall down in autumn, the tree stays alive.

如此, 我们可以构建这样的一个层级关系, 那些非常重要的, 必须十分可靠的东西靠近树根, 而那些易变的部分聚集在树叶附近. 这其实跟现实世界中的大多数树很像: 树叶是活动的, 树叶很多, 树叶秋天的时候全都掉下来了, 然而树本身一直活着.

That means that when you structure Erlang programs, everything you feel is fragile and should be allowed to fail has to move deeper into the hierarchy, and what is stable and critical needs to be reliable is higher up.

所以当你组织 Erlang 程序的时候, 那些你感觉不稳固的和可以出错的部分要尽可能放在较低的层级, 在高的层级上放那些需要稳定性的重要的部分.

![Supervisors](/static/zen_of_erlang/012.png)

Supervisors can do that through usage of links and trapping exits. Their job begins with starting their children in order, depth-first, from left to right. Only once a child is fully started does it go back up a level and start the next one. Each child is automatically linked.

监督者通过使用链接和捕获退出信号工作. 它们的任务首先是从左至右, 以深度优先逐次启动其子进程. 只有前一个子进程完全启动成功时它才会进一步启动下一个子进程. 每一个子进程都会自动链接到监督者.

Whenever a child dies, one of three strategies is chosen. The first one on the slide is 'one for one', enacted by only replacing the child process that died. This is a strategy to use whenever the children of that supervisor are independent from each other.

当一个子进程挂掉时, 你有三种策略可以选择. 首先是 `one_for_one`, 具体方法是只重启挂掉的那一个进程. 这种策略适合子进程相互独立的监督者使用.

The second strategy is 'one for all'. This one is to be used when the children depend on each other. When any of them dies, the supervisor then kills the other children before starting them all back. You would use this when losing a specific child would leave the other processes in an uncertain state. Let's imagine a conversation between three processes that ends with a vote. If one of the process dies during the vote, it is possible that we have not programmed any code to handle that. Replacing that dead process with a new one would now bring a new peer to a table that has no idea what is going on either!

第二种策略是 `one_for_all`. 子进程互相都有依赖时就应当使用这种策略. 任何一个子进程挂掉时, 监督者首先杀掉其他的所有子进程, 然后再重新启动它们. 具体来说, 当一个子进程出问题会导致其他进程进入异常状态的时候就应当用这种策略. 比方说三个进程在讨论和投票. 我们可能没有编写处理投票时某个进程挂掉的代码. 这时如果仅仅重启挂掉的那个进程, 新的进程根本无法继续完成之前的任务.

This inconsistent state is possibly dangerous to be in if we haven't really defined what goes on when a process wreaks havoc through a voting procedure. It is probably safer to just kill all processes, and start afresh from a known stable state. By doing so, we're limiting the scope of errors: it is better to crash early and suddenly than to slowly corrupt data on a long-term basis.

假如我们没有定义投票时一个进程搞破坏的情况, 类似这样的不一致状态会是非常危险的. 这时杀掉所有相关的进程, 从已知的稳定状态开始也许更安全. 同时这也可以限制出错的范围: 尽可能早的崩溃要比让坏掉的进程慢慢破坏数据要好的多.

The last strategy happens whenever there is a dependency between processes according to their booting order. Its name is 'rest for one' and if a child process dies, only those booted after it are killed. Processes are then restarted as expected.

最后一种策略适用于进程间的依赖与启动顺序相关的情况. 名字是 `rest_for_one`, 当某个子进程挂掉时, 只有那些在这个进程之后启动的进程才会被杀掉重启.

Each supervisor additionally has configurable controls and tolerance levels. Some supervisors may tolerate only 1 failure per day before aborting while others may tolerate 150 per second.

每个监督者还可以单独配置容错度. 比如一些重要层级上的监督者可能一天最多允许出现一次故障, 而其他的也许一秒种出现150次也没关系.

![Heisenbugs](/static/zen_of_erlang/013.png)

The comment that usually comes right after I mention supervisors is usually to the tune of "but if my configuration file is corrupted, restarting won't fix anything!"

通常我解释了监督者之后第一个问题都类似这种: ｢可是如果我的配置文件被破坏了, 重启了也无济于事啊!｣

That is entirely right. The reason restarting works is due to the nature of bugs encountered in production systems. To discuss this, I have to refer to the terms 'Bohrbug' and 'Heisenbug' coined by Jim Gray in 1985 (I do recommend you read as many Jim Gray papers as you can, they're pretty much all great!)

这么说当然没错. 重启之所以能解决一部分问题关系到生产环境中所遇到的问题的本质. 为了解释这个问题, 我需要引用詹姆斯·格雷先生[^5]于1985年提出的一组概念: ｢玻尔 Bug｣ 和 ｢海森堡 Bug｣ (Bohrbug, Heisenbug, 下文不再翻译[^6])

Basically, a bohrbug is a bug that is solid, observable, and easily repeatable. They tend to be fairly simple to reason about. Heisenbugs by contrast, have unreliable behaviour that manifests itself under certain conditions, and which may possibly be hidden by the simple act of trying to observe them. For example, concurrency bugs are notorious for disappearing when using a debugger that may force every operation in the system to be serialised.

简单的说, Bohrbug 是那种固定会发生的, 比较明显的, 很容易重现的错误. 通常也很容易找出错误的原因. 相对的, Heisenbug 的行为很不规律, 往往只有在特殊条件下才会发生, 当你想观测它们的时候又消失不见了. 举例来说, 当你用调试器想要调试一个并发错误时往往它就不发生了, 因为调试器可能强制让整个系统序列化地运行.

Heisenbugs are these nasty bugs that happen once in a thousand, million, billion, or trillion times. You know someone's been working on figuring one out for a while once you see them print out pages of code and go to town on them with a bunch of markers.

Heisenbug 就是这种发生率只有千万甚至亿万分之一的错误. 假如你看到有人带着一叠打印出来的代码, 上面还有成堆的标记, 那十有八九这个人就是在调试一个 Heisenbug.

With these terms defined, let's look at what should be their frequencies.

现在你知道它们的含义了, 我们先来看看它们发生的频率.

![Ease of Finding Bugs in Production](/static/zen_of_erlang/014.png)

Here I'm classifying bohrbugs as repeatable, and heisenbugs as transient.

这里我把 Bohrbug 标记成可重复的 (repeatable), Heisenbug 则是临时的 (transient).

If you have bohrbugs in your system's core features, they should usually be very easy to find before reaching production. By virtue of being repeatable, and often on a critical path, you should encounter them sooner or later, and fix them before shipping.

存在于系统主要特性中的 Bohrbug 往往在部署到生产环境之前就能很容易地发现. 因为它们很容易重现, 又在整个系统的关键部位上, 你早晚都会在发布之前遇到它们并一一修复.

Those that happen in secondary, less used features, are far more of a hit and miss affair. Everyone admits that fixing all bugs in a piece of software is an uphill battle with diminishing returns; weeding out all the little imperfections takes proportionally more time as you go on. Usually, these secondary features will tend to gather less attention because either fewer customers will use them, or their impact on their satisfaction will be less important. Or maybe they're just scheduled later and slipping timelines end up deprioritising their work.

那些在次要特性中的错误, 更多的是看碰不碰的上. 人们都承认要修复一个软件中所有的错误是不太可能的; 那些越是最后剩下的细微处的问题越是会消耗成倍增长的时间. 通常这些次要特性中的错误不会那么受人关注, 要么是用户数量很少, 要么是对整体体验的影响不大. 或者这些问题本来优先级就没那么高.

In any case, they are somewhat easy to find, we just won't spend the time or resources doing so.

不管怎么说, 这些错误都还是很简单就可以找到的, 只是我们没打算耗费很多时间或精力去修复它们.

Heisenbugs are pretty much impossible to find in development. Fancy techniques like formal proofs, model checking, exhaustive testing or property-based testing may increase the likelihood of uncovering some or all of them (depending on the means used), but frankly, few of us use any of these unless the task at hand is extremely critical. A once in a billion issue requires quite a lot of tests and validation to uncover, and chances are that if you've seen it, you won't be able to generate it again just by luck.

Heisenbug 基本上在开发环境中完全不会出现. 类似[公式证明](https://www.wikiwand.com/en/Formal_proof), [模型检查](https://www.wikiwand.com/en/Model_checking), 极其详尽的测试或者[属性测试](https://www.wikiwand.com/en/QuickCheck)这样的高级技术或许能提高发现这类问题的概率, 不过坦白地讲, 我们通常都不会用刚刚提过的任何一种方法, 除非是写什么特别特别重要的东西. 一个亿万分之一概率的问题需要写大量的测试和验证才能发现, 而且很多情况下就算你偶尔看见了一次, 再跑一次也不一定会重现同样的问题.

![Bugs that Happen in production](/static/zen_of_erlang/015.png)

The next connection I want to make is regarding the frequency each of these types of bugs have in production (in my experience). There's no obvious proof that there is any connection between the use of finding bugs and their incidence in production systems, but my gut feeling would tell me that such a connection does exist.

接下来我想讨论(根据我的经验)这两种错误在生产环境中发生的频率. 虽然没办法证明, 直觉告诉我发现错误的难易和它们在生产环境中发生的频率还是有某种联系的.

First of all, easy repeatable bugs in core features should just not make it to production. If they do, you have essentially shipped a broken product and no amount of restarting or support will help your users. Those require modifying the code, and may be the result of some deeply entrenched issues within the organisation that produced them.

首先, 主要特性中反复出现的错误根本不应该出现在生产环境中. 如果有的话你就等于发布了一个有本质缺陷的产品, 不论怎么重启都没用的. 因为这类错误就是代码质量的问题, 也或许是公司里更深层的什么问题导致的.

Repeatable bugs in side-features will pretty often make it to production. I do think this is a result of not taking the time to test them properly, but there's also a strong possibility that secondary features often get left behind when it comes to partial refactorings, or that the people behind their design do not fully consider whether the feature will coherently fit with the rest of the system.

次要特性中我们还是常常会看到那些重复出现的错误. 我也认为这是没有花足够的时间充分测试所导致的, 但也有很大可能是这些次要特性在重构的时候没人关注, 或者设计它们的人并没有完整地考虑过在各种情况下它们如何与系统的系统部分合作.

On the other hand, transient bugs will show up all the damn time. Jim Gray, who coined these terms, reported that on 132 bugs noted at a given set of customer sites, only one was a Bohrbug. 131/132 of errors encountered in production tended to be heisenbugs. They're hard to catch, and if they're truly statistical bugs that may show once in a million times, it just takes some load on your system to trigger them all the time; a once in a billion bug will show up every 3 hours in a system doing 100,000 requests a second, and a once in a million bug could similarly show up once every 10 seconds on such a system, but their occurrence would still be rare in tests.

另一方面, 临时性的问题什么时候都有可能发生. 提出这两个概念的格雷先生曾表示在某个客户遇到的132个错误中, 只有一个是 Bohrbug. 在这个生产环境里, 总共132个错误里131个都是 Heisenbug. 你很难捕捉这类错误, 假如说统计学上它们发生的概率是几百万分之一, 那么当你的负载达到一定的量级就会时常发生这类问题; 一个十亿分之一概率的错误, 每三个小时就会在一个每秒处理十万次请求的系统中发生一次, 而一个百万分之一概率的错误每10秒就会在这样的系统中发生一次, 然而这样的频率在(绝对数量没那么多的)测试中可以说太稀有了.

That's a lot of bugs, and a lot of failures if they are not handled properly.

听起来这可是很多很多的错误, 并且如果没有正确处理的话每一个都有可能导致系统故障.

![Bugs Handled by Restarts](/static/zen_of_erlang/016.png)

So really, how efficient is restarting as a strategy?

那么回到本来的问题, 重启进程这种策略能不能解决这些问题呢?

Well for repeatable bugs on core features, restarting is useless. For repeatable bugs in less frequently used code paths, it depends; if the feature is a thing very important to a very small amount of users, restarting won't do much. If it's a side-feature used by everyone, but to a degree they don't care much about, then restarting or ignoring the failure altogether can work well. For example, if the facebook 'poke' feature were to be broken (would it still exist), not too many users would notice or see their experience ruined by its failure.

首先对于主要特性中的重复性错误, 重启应该是没什么用的. 对于次要特性中的这类问题, 可能要取决于具体情况; 如果这个功能对于一小部分用户来说很重要, 重启进程同样是没什么用的. 如果大多数人都不在乎这个功能好不好用, 那么重启或者忽略这个错误就还好. 比方说, 假如 Facebook 的｢戳｣功能失效了(假如现在还有这么个功能的话), 可能大多数人不会觉得他们的体验受到了什么影响.

For transient bugs though, restarting is extremely effective, and they tend to be the majority of bugs you'll meet live. Because they are hard to reproduce, that their showing up is often dependent on very specific circumstances or interleavings of bits of state in the system, and that their appearance tends to be in a very small fraction of all operations, restarting tends to make them disappear altogether.

而对于临时性的错误来说, 重启是十分有效的, 并且这类错误才是你在生产环境中主要会遇到的. 这类错误之所以难以重现, 往往是因为它们只有在非常特定的条件下才会触发, 或者是系统处于什么中间状态时. 同时它们只会在很少的一部分操作里才会发生, 重启同时可以彻底｢解决｣(或者说隐藏)这一类问题.

Rolling back to a known stable state and trying again is unlikely to hit the same weird context that causes them. And just like that, what could have been a catastrophe has become little more than a hiccup for the system, something users quickly learn to live with.

从一个已知的稳定的状态重新开始, 很可能这个操作就不会遇到发生了问题的那种上下文. 重启可以让这种本来可能导致系统级灾难的错误变成一个小问题, 用户也不会注意到什么不同.

You can then make use of logging, tracing, or a variety of introspection tools (which all come out of the box in Erlang) to later find, understand, and fix the issues so they stop happening. Or you could just decide to tolerate them were the effort required to fix the issues too large.

在这之后, 你可以通过使用日志, 追踪, 或者其他 Erlang 提供的分析工具来查找, 理解最后修复这一类问题. 假如这个错误不值得花那么多时间去修复, 你也完全可以选择就让它们出错时被监督者重启.

![notorious bsd](/static/zen_of_erlang/017.png)

This question was asked to me on a forum where I was discussing programming stuff and discussing the Erlang model. I copied it verbatim because it's a great example of a question a lot of people ask when they hear about restarting and Erlang's features.

这个问题 [^7] 是我在某个论坛上讨论编程相关的话题和 Erlang 模型时有人问我的. 我直接把原文贴过来, 因为很多人在听了 Erlang 的特性和重启之后都会问类似的问题.

I want to address it specifically by giving a realistic example of how a system could be designed in Erlang, which will highlight its peculiarities.

我想通过一个用 Erlang 设计的系统的实际例子来回答这个问题, 这可以突出一些 Erlang 独特的地方.

![supervision tree demo](/static/zen_of_erlang/018.png)

With supervisors (rounded squares), we can start creating deep hierarchies of processes. Here we have a system for elections, with two trees: a tally tree and a live reports tree. The tally tree takes care of counting and storing results, and the live reports tree is about letting people connect to it to see the results.

我们可以通过监督者(图中以圆角矩形表示)将进程组织成很深的层次关系. 这里我们看到一个选举系统, 大体上分成两个树: 一个管理选票, 另一个负责实时报告. 选票树负责计数和储存结果, 实时报告树可以让人们看到结果.

By the order the children are defined, the live reports will not run until the tally tree is booted and functional. The district subtree (about counting results per district) won't run unless the storage layer is available. The storage's cache is only booted if the storage worker pool (which would connect to a database) is operational.

按照子进程定义的顺序, 只有当选票树完成启动开始工作时实时报告的服务才会启动. 负责分区计数的地区子树要等存储层上线了才能启动. 存储层中的缓存也要等实际连接到数据库的存储工作池可以用了才会启动.

The supervision strategies I mentioned earlier let us encode these requirements in the program structure, and they are still respected at run time, not just at boot time. For example, the tally supervisor may be using a one for one strategy, meaning that districts can individually fail without effecting each other's counts. By contrast, each district (Quebec and Ontario's supervisors) could be employing a rest for one strategy. This strategy could therefore ensure that the OCR process can always send its detected vote to the 'count' worker, and it can crash often without impacting it. On the other hand, if the count worker is unable to keep and store state, its demise interrupts the OCR procedure, ensuring nothing breaks.

我前面讲过的监督策略让我们可以把这些需求直接转化为程序的结构, 无论启动时还是运行时系统都会遵循这些规则. 例如, 分区树的根监督者可能会用 `one_for_one` 的策略, 这样某一个分区出了问题不会影响到其他的分区子树. 而具体一个分区的监督者可以用 `rest_for_one` 的策略, 这样可以保证 OCR 进程在检测选票并发送给 `count` 进程时, 自身的崩溃不会影响到计数. 而一旦计数进程出现错误时, 监督者会同时关闭 OCR 进程, 保证没有进一步的崩溃.

The OCR process itself here could be just monitoring code written in C, as a standalone agent, and be linked to it. This would further isolate the faults of that C code from the VM, for better isolation or parallelisation.

OCR 进程可能只是一段监控代码, 此时具体的工作可以是用 C 写的另一个程序. 这可以进一步将 C 代码中的错误与 Erlang VM 分离.

Another thing I should point out is that each supervisor has a configurable tolerance to failure; the district supervisor might be very tolerant and deal with 10 failures a minute, whereas the storage layer could be fairly intolerant to failure if expected to be correct, and shut down permanently after 3 crashes an hour if we wanted it to.

正如前面提过的, 每个监督者可以配置不同的容忍度; 分区的监督者可以很宽松, 允许一分钟内出现10次崩溃, 而存储层为了保证正确性就会更严格, 比如一个小时内如果有3次崩溃我们就要关闭整个程序去除错了.

In this program, critical features are closer to the root of the tree, unmoving and solid. They are unimpacted by their siblings' demise, but their own failure impacts everyone else. The leaves do all the work and can be lost fairly well — once they have absorbed the data and operated their photosynthesis on it, it is allowed to go towards the core.

像这个程序里, 关键的部分靠近树根, 不会变动. 他们不会受其他进程影响, 但如果他们自己出问题了会影响所有人(子进程等). 叶子上的进程负责实际的工作, 有些就算失败或崩溃了也没关系, 只要他们｢吸收｣了数据, 完成了｢光合作用｣就好.

So by defining all of that, we can isolate risky code in a worker with a high tolerance or a process that is being monitored, and move data to stabler process as information matures into the system. If the OCR code in C is dangerous, it can fail and safely be restarted. When it works, it transmits its information to the Erlang OCR process. That process can do validation, maybe crash on its own, maybe not. If the information is solid, it moves it to the Count process, whose job is to maintain very simple state, and eventually flush that state to the database via the storage subtree, safely independent.

通过定义这些层级关系等等, 我们就可以将较为危险的代码放入容忍度较高的工作进程中, 把处理得出的数据存放到更加稳定的进程中. 如果用 C 写的 OCR 代码不够稳定, 我们可以允许这些进程失败并重启它们. 当它们成功的时候会将数据传给 Erlang 系统中的 OCR 进程. 这个进程可能会做一些验证, 也可能在这一部分崩溃. 不过只要数据没问题, 它就会将其传给计数进程, 这个进程只要维护一个很简单的状态, 并在最后(通过存储层的进程)写入到数据库中即可, 这样数据就安全存放在系统之外了.

If the OCR process dies, it gets restarted. If it dies too often, it takes its own supervisor down, and that bit of the subtree is restarted too — without affecting the rest of the system. If that fixes things, great. If not, the process is repeated upwards until it works, or until the whole system is taken down as something is clearly wrong and we can't cope with it through restarts.

如果 OCR 进程挂掉了, 它会被重启. 如果它挂的太频繁, 它的监督者会挂掉, 然后这一部分子树也一样会被重启 - 注意这些都不会影响到系统的其他部分. 如果重启后没问题了, 那万事大吉. 如果还不行, 进程就会一层一层地向上重复崩溃重启的步骤, 直到系统恢复正常, 当然如果真的遇到无法解决的问题, 最后整个系统也可能都关掉了.

There's enormous value in structuring the system this way because error handling is baked into its structure. This means I can stop writing outrageously defensive code in the edge nodes — if something goes wrong, let someone else (or the program's structure) dictate how to react. If I know how to handle an error, fine, I can do that for that specific error. Otherwise, just let it crash!

这种架构系统的方式有着巨大的价值, 因为我们是通过**结构**在进行错误处理. 这意味着在叶子节点上我们无需再去写那些粗暴的｢防御式｣代码, 出了问题时让其他的进程(或是程序的结构)来决定采取什么措施. 如果我们确定地知道某些特定错误需要如何处理, 我们依然可以去编写针对这些情况的代码. 但除此之外, **让他崩溃就好了!**

This tends to transform your code. Slowly you notice that it no longer contains these tons of if/else or switches or try/catch expressions. Instead, it contains legible code explaining what the code should do when everything goes right. It stops containing many forms of second guessing, and your software becomes much more readable.

这也会影响你所写的代码. 慢慢地你会发现代码里不再有成堆的 `if/else` 或是 `try/catch` 之类的语句. 相反, 你的代码只会关注当一切照常的时候怎样完成它的工作. 你不再写各式各样的代码试图处理你无法预测的错误, 代码的可读性也会显著提高.

![supervision subtrees](/static/zen_of_erlang/019.png)

When taking a step back and looking at our program structure, we may in fact find that each of the subtrees encircled in yellow seem to be mostly independent from each other in terms of what they do; their dependency is mostly logical: the reporting system needs a storage layer to query, for example.

再退开一步观察我们的程序结构, 你或许会发现黄色圈出来的这几个子树在功能上基本是彼此独立的; 他们的依赖关系只是逻辑上的, 比方说报告系统要从某个存储层里查询数据.

It would also be great if I could, for example, swap my storage implementation or use it independently in other systems. It could be neat, too, to isolate the live reports system into a different node or to start providing alternative means (such as SMS for example).

这样组织之后还有其他的好处, 比如说换另一个存储层的实现或者将这个存储层独立地用于其他的系统. 假如我们能把实时报告系统放到另一个节点上或者是提供更多通知方法比如短信啊之类的就更棒了.

What we now need is to find a way to break up these subtrees and turn them into logical units that we can compose, reuse together, and that we can otherwise configure, restart, or develop independently.

我们需要的某种可以将这些子树拆分更若干个逻辑单元, 可以独立开发, 配置, 重启, 同时又可以任意组合, 重用的方法.

![OTP apps](/static/zen_of_erlang/020.png)

OTP applications are what Erlang uses as a solution here. OTP applications are pretty much the code to construct such a subtree, along with some metadata. This metadata contains basic stuff like version numbers and descriptions of what the app does, but also ways to specify dependencies between applications. This is useful because it lets me keep my storage app independent from the rest of the system, but still encode the tally app's need for it to be there when it runs. I can keep all the information I had encoded in my system, but now it is built out of independent blocks that are easier to reason about.

Erlang 为此提供的方案是 OTP 应用 (Application). OTP 应用基本上就包括构建这样的子树的一点代码以及一些元数据. 元数据中包括了一些基本信息, 比如版本号, 一段关于这个应用的描述, 还包括了这个应用和其他应用的依赖关系等等. 这可以保证虽然存储应用是与系统的其他部分独立的, 但其他应用依然可以将其编入自己的依赖列表中, 保证运行时这些应用都可用. 构建整个系统的方法其实差不多, 不过现在每一部分更加独立, 从而可以更好地管理和分析.

In fact, OTP applications are what people consider to be libraries in Erlang. If your code base isn't an OTP application, it isn't reusable in other systems. [Sidenote: there are ways to specify OTP libraries that do not actually contain subtrees, just modules to be reused by other libraries]

事实上人们常常把 OTP 应用看作是 Erlang 世界中的库. 如果你的代码没有组织成一个 OTP 应用, 那就无法在其他系统中重用. [^8]

With all of this done, our Erlang system now has all of the following properties defined:

有了这些, 我们的 Erlang 可以定义下面的这些属性:

- what is critical or not to the survival of the system
- what is allowed to fail or not, and at which frequency it can do so before it is no longer sustainable
- how software should boot according to which guarantees, and in what order
- how software should fail, meaning it defines the legal states of partial failures you find yourself in, and how to roll back to a known stable state when this happens
- how software is upgraded (because it can be upgraded live, based on the supervision structure)
- how components interdepend on each other

- 对于整个系统来说哪些是绝对重要的, 哪些是不太重要的
- 哪些是允许出错的, 又允许以何种频率出错
- 程序的启动有何需求, 又应当以何种顺序
- 程序应如何应对故障, 即一部分出错时怎样的状态是合法的, 以及如何回滚到一个已知的稳定状态
- 程序应如何升级 (通过监督结构我们可以实现在线升级)
- 各个组件之间如何相互依赖

This is all extremely valuable. What's more valuable is forcing every developer to think in such terms from early on. You have less defensive code, and when bad things happen, the system keeps running. All you have to do is go look at the logs or introspect the live system state and take your time to fix things, if you feel it's worth the time.

所有这些都很有价值. 比这些更有价值的是迫使每个开发者尽早地以这种方式思考(系统构建的方式). 你减少了｢防御式｣的代码, 而系统出错时依然可以继续运行. 如果你觉得某个错误值得花时间修复, 你可以查看日志或是直接观察生产环境上系统的状态, 而且没有时间上的压力.

![sleep at night](/static/zen_of_erlang/021.png)

With all of this done, I should be able to sleep at night, right? Hopefully yes. What I included here is a small pixelated diagram from a new software deploy we ran at Heroku a couple of years ago.

如果我做到了这些, 我该可以在睡个安稳觉了吧? 说不定真的可以哟. 这张图来自若干年前我们在 Heroku 部署的一个新程序.

The leftmost side of the diagram is around September. By that time, our new proxying layer (vegur) had been in production for maybe 3 months, and we had ironed out most of the kinks in it. Users had no problem, the transition was going smoothly and new features were being used.

这张图最左边是大概九月份的时候. 那个时候我们新写的代理层 (vegur) 已经在生产环境使用了大概3个月了, 我们也基本上修复了所有的已知问题. 用户也没再反映什么问题, 迁移过程也很平稳, 新的特性开始投入使用.

At some point, a team member got a very expensive credit card bill for the logging service we were using to aggregate exceptions. That's when we took a look at it and saw the horror on the leftmost side of the diagram: we were generating between 500,000 to 1,200,000 exceptions a day! Holy cow, that was a lot. But was it? If the issue was a heisenbug, and our system was seeing, say 100,000 requests a second, what were the odds of it happening? Something between 1/17000 and 1/7000. Somewhat very frequent, but because it had no impact on service, we didn't notice it until the bandwidth and storage bill came through.

后来有一天, 一个同事从我们使用的日志服务那儿收到了一笔高额账单. 那时候我们才关注到这张图, 发现那时候每天都有50万到120万左右的异常发生! 天啊, 竟然有那么多. 但真的有很多么? 如果这是个 Heisenbug, 然后我们的系统比如说每秒会处理10万左右的请求, 那么它发生的概率是多少? 也就是 1/17000 到 1/7000 左右. 也可以说蛮频繁的, 但因为它对主要的服务没什么影响, 我们直到看到带宽和存储相关的账单时才注意到这个问题.

It took us a short while to figure out the error, and we fixed it. You can see that there is still a low rate of exceptions after that, maybe a few dozen thousands a day. They're all things we know of, but are impact-free. Two years later and we haven't bothered to fix it because the system works fine despite that.

修复这个问题花了点时间. 你可以看到那之后其实每天还会发生几千个异常左右. 这是已知问题, 而且不会影响到我们的服务. 两年过去了我们也没有耗费精力去完全修复这个问题, 因为整个系统依然运转良好.

![expect failure](/static/zen_of_erlang/022.png)

At the same time, you can't always just sleep at night. Failures can be out of your control despite your best design efforts.

可惜的是, 你不能真的什么都不管. 有些故障源自你无法控制的东西, 即便是再好的设计也无法幸免.

A couple of years ago I was on a flight to Vancouver starting on its descent when the pilot used the intercom to tell us something a bit like this: "this is your captain speaking, we will be landing shortly. Do not be alarmed as we will stay on the tarmac for a few minutes while the fire department looks over the plane. We have lost some hydraulic component and they want to ensure there is no risk of fire. There are two backup systems for the broken one, and we should be fine."

若干年前我坐飞机去温哥华, 飞机开始下降时机长通过广播说了类似这样的内容: ｢我是机长, 我们即将着陆. 我们会在跑道上等待消防部门对飞机进行检查, 到时别惊慌. 有几个液压组件失灵了, 他们需要确保飞机没有起火的危险. 我们还有两套备用的系统, 不会有事的.｣

And we were fine. In this case the airplane was amazingly well designed.

我们确实没事. 这再次表明了飞机的设计多么完备.

The image for this slide isn't that flight though, it's another one I was on two weeks ago, while the Eastern US were being burrowed under 24 inches of snow. The plane (flight United 734), which I'm sure was as reliable, landed on the runway. When it came time to break though, it made a loud noise, what I assume is the ABS equivalent of aircrafts, but it still kept going.

这张照片并不是那次航班, 而是两周前我飞往积雪24寸的美国东部时所乘坐的飞机. 那架 - 我敢说一样可靠的 - 飞机开始在跑道上降落. 然而刹车时, 飞机发出了巨大的声响, 我猜是飞机上类似 ABS 的机制, 然而飞机没有及时停下来.

We ran over the red lights at the end of the runway you see on the picture, and at the end of the tarmac, the plane skid off the runway, missed the onramp, and the front wheel ended up in the grass. Everyone was fine, but this is an example of why great engineering cannot save the day every time.

我们冲出了标志跑道尽头的红灯, 飞机一路滑出跑道, 错过了斜坡弯道, 最后前轮停在了草地里. 大家都没事, 但这正说明了即使是杰出的工程设计也无法保证绝对的安全.

![danger zone](/static/zen_of_erlang/023.png)

In fact, operations will always remain a huge factor in successful systems being deployed. This slide is heavily inspired (pretty much stolen in fact) from presentations by Richard Cook. If you don't know him, I urge you to go watch videos of his talks on youtube, they're pretty much all fantastic.

事实上, 运维 [^9] 工作总是会极大地影响一个系统能否成功部署. 这张图来自理查德·库克先生的一次演讲. 假如你没听说过他, 我强烈建议你去 YouTube 上看看他的视频, 都很不错.

Proper system architecture and development practices can still not replace, or can be broken by inadequate operations; the efficiency and usefulness of tools, playbooks, monitoring, automation, and so on, all tend to implicitly rely on the knowledge and respect of well-defined operating conditions (throughput, load, overload management, etc.) If defined at all, these operational limits let you know when things are about to go bad, and when they are good again.

良好的系统架构和开发实践依然无法完全取代运维工作, 或者说不好的运维会直接毁掉一个设计和实现良好的系统; 你所依赖的开发, 监控, 自动化工具等等, 它们的实用性和效率全都会收到运维状况的影响 (带宽, 负载, 过载管理等等). 只有当你了解这些运维限制时才能了解什么时候系统要开始出问题了, 以及什么时候系统恢复正常了.

The problem with these limits is that as operators get used to them, and get used to frequently breaking them without negative consequences, there is a risk of slowly pushing the envelope towards the edge of the danger zone, where nasty large-scale failures happen. Your reaction time and margin to adapt to higher loads erodes, and you eventually end in a position where things are constantly broken with no respite in sight.

问题在于运维人员会慢慢习惯于这些, 当偶尔破坏了某些限制而没有影响的时候他们会渐渐开始忽略这些条件, 就好像一点点地逼近危险范围的临界点, 超过了这个临界点就会产生复杂的大规模系统故障. 在这个过程里你对于高危情况的嗅觉也被慢慢侵蚀, 最后你可能发现系统的各个部分持续的崩溃, 完全没有停下来的意思.

So we have to be careful and aware of this kind of thing, and to the importance that people using and operating the software has on it. It is always harder to scale up a good team than it is to scale up a program. Plan for emergencies even if they don't happen; they will some day and you'll be happy you ran simulations and have recipes to follow to fix it all up.

所以我们还是要注意这些事情, 并且那些使用和运维这些软件的人也必须有所警觉. 想要扩大一个好的团队要远远比给一个软件扩容要难. 在紧急情况出现之前就有所准备, 这样当问题真的出现时你就有现成的方案来修复整个系统.

![plane emergency measures](/static/zen_of_erlang/024.png)

In the case of my flight, as I said, nobody was injured. Still, this is the circus that was deployed for it all: busses to escort passengers back to the terminal, since moving a stranded plane could be risky. Pick-up trucks to escort the busses safely from the runway to the terminal. Police cars, a whole lot of fire trucks, and that black car that I don't know what it does but I'm sure it's super useful.

回到我刚刚的例子, 就像我说的并没有人受伤什么的. 但是机场仍然派出了所有这些装备来确保万无一失: 护送乘客返回航站楼的大巴. 护送大巴的车辆. 警车, 很多辆消防车, 还有我猜相当重要的那台黑色车辆.

They deploy all of that despite everyone being fine, despite planes being super reliable. They do things right.

尽管人和飞机都没事, 他们依然采取了这么多应对措施. 这个做法值得借鉴.

![other goodies](/static/zen_of_erlang/025.png)

Here's another bunch of things you gain by using Erlang. I don't really have much to say about them, just that I do tend to have some kind of interest in you switching to use it, so here it is.

这一页 [^10] 列出了 Erlang 带来的其他一些功能. 这次我不会具体解释它们, 如果你有兴趣使用 Erlang 的话可以去看看这些.

The last point is worth commenting though. One of the risks that happen in languages that are very flexible in their approach in system design is that libraries you use may not want to do things the way you feel would be appropriate in your case, and you're left either not using the lib, or having to operate codebases with an incoherent design. This doesn't happen in Erlang as everyone uses the same proven approach to do things.

关于最后一点我想再多说几句. 那些更为灵活的语言常常遇到的问题是你想使用的某个库和你做事的｢风格｣不符, 结果就是要么不用这个库, 要么就得忍受同一个代码库中存在着不一致的设计. Erlang 不存在这样的情况, 因为大家都遵循同样的准则 [^11] 去解决问题.

![how things interact](/static/zen_of_erlang/026.png)

In a nutshell, the Zen of Erlang and 'let it crash' is really all about figuring out how components interact with each other, figuring out what is critical and what is not, what state can be saved, kept, recomputed, or lost. In all cases, you have to come up with a worst-case scenario and how to survive it. By using fail-fast mechanisms with isolation, links & monitors, and supervisors to give boundaries to all of these worst-case scenarios' scale and propagation, you make it a really well-understood regular failure case.

总的来说, 所谓 Erlang 之禅和 "Let it crash" 实际上就是要理解各个组件之间究竟是如何互动的, 理解哪些是绝对重要和不那么重要的, 哪些状态可以被保存, 或是可以暂时保留, 或是可以重新计算, 又或者是可以扔掉的. 对于任何问题, 你都要想出最坏可能的情况, 然后思考如何摆脱困境. 通过使用由进程独立, 监控与链接以及监督者所构建的尽早崩溃的机制, 你可以控制那些最坏情况所影响的范围, 进而有可能将其转化为很普通的故障.

That sounds simple, but it's surprisingly good; if you feel that your well-understood regular failure case is viable, then all your error handling can fall-through to that case. You no longer need to worry or write defensive code. You write what the code should do and let the program's structure dictate the rest. Let it crash.

这一切听起来很简单, 却也非常实用; 如果你觉得管理自己已经完全理解的, 有规律可循的崩溃是一条可行之路, 那么你可以把所有的错误处理都归结于此. 你不再需要担心未曾处理的意外情况, 不再去写｢防御式｣的代码. 你只需写真正干活的那部分代码, 其他的情况交给程序的结构来处理. 不要惧怕崩溃, Let it crash.

![zen of erlang](/static/zen_of_erlang/027.png)

That’s the Zen of Erlang: building interactions first, making sure the worst that can happen is still okay. Then there will be few faults or failures in your system to make you nervous (and when it happens, you can introspect everything at run time!) You can sit back and relax.

这便是 Erlang 之禅: 首先构建整个系统如何交互, 保护好可能发生的最坏的情况. 系统中不再有大量需要你担心的错误(当真的出现问题时, 你也可以直接查看运行时的所有状态!) 如此你就能放轻松了.

[^1]: 实在是不了解这些术语是怎么翻译的...
[^2]: Erlang 的进程不同于一般概念中由操作系统提供的｢进程｣, 下文若非明确提及, ｢进程｣皆特指 Erlang 进程
[^3]: Pattern Matching 是 Erlang 很｢独特｣同时也非常强大的一个特性, 其直接导致了 Erlang 中函数的写法有别于很多更为常见的语言. 若想详细了解建议阅读 Erlang 或 Elixir 相关的书籍或在线教程等
[^4]: 监督者是 OTP 的一个核心组件, OTP 是 Erlang/OTP 这个常常写在一起的名字里面表示一个通用开发平台的那部分. (虽然全称是 Open Telecom Platform, 但现在一般不在意这层意思, 只称为 OTP)
[^5]: Jim Gray, [wiki](https://www.wikiwand.com/en/Jim_Gray_(computer_scientist)), Fred 还建议多阅读他的论文, 基本上都写的很好
[^6]: 对量子力学有所了解的读者看到这两个名字应该会会心一笑吧 :)
[^7]: ｢我喜欢静态类型的语言. 遇到没有处理的异常时我会直接重启整个 daemon. Erlang 有什么更好的方案来提供高容错性么?｣
[^8]: OTP 应用完全可以不包含需要运行起来的监督树, 而只包含一些模块代码
[^9]: Operation, 我一直觉得翻译成｢运营｣或者是｢运维｣都怪怪的...
[^10]: 
    - 模式匹配
    - 函数式编程
    - 可选的类型检查
    - 基于属性的测试工具
    - 在线代码升级
    - 遵循同一套准则的社区

[^11]: OTP
