## 第 9 集：高级 CPU 设计  
00:24  早期是加快晶体管切换速度，来提升 CPU 速度  
01:20  给 CPU 专门的除法电路 + 其他电路来做复杂操作，比如游戏，视频解码  
02:28  给 CPU 加缓存，提高数据存取速度，更快喂给 CPU，用计算餐馆销售额举例  
05:13  脏位 -  Dirty bit  
05:33  流水线设计，用 1 个洗衣机和 1 个干燥机举例  
06:01  并行处理 -  parallelize  
07:33  乱序执行 -  out-of-order execution  
08:21  推测执行 -  speculative execution  
08:50  分支预测 -  branch prediction  
09:34  多个 ALU  
09:54  多核 (Core)  
10:11  多个独立 CPU  
10:52  超级计算机，中国的&quot;神威 太湖之光&quot;  

Hi, I’m Carrie Anne and welcome to CrashCourse Computer Science!
(｡･∀･)ﾉﾞ嗨，我是 Carrie Anne，欢迎收看计算机科学速成课！

随着本系列进展，我们知道计算机进步巨大

从 1 秒 1 次运算，到现在有千赫甚至兆赫的CPU

你现在看视频的设备八成也有 GHz 速度————1秒十亿条指令

这是很大的计算量！

早期计算机的提速方式是  减少晶体管的切换时间.

————晶体管组成了逻辑门，ALU 以及前几集的其他组件
但这种提速方法最终会碰到瓶颈，所以处理器厂商 发明各种新技术来提升性能，不但让简单指令运行更快 也让它能进行更复杂的运算

上集我们写了个做除法的程序，给 CPU 执行 方法是做一连串减法，比如16除4 会变成 16-4 -4 -4 -4 碰到 0 或负数才停下.

但这种方法要多个时钟周期，很低效 所以现代 CPU 直接在硬件层面设计了除法，可以直接给 ALU除法指令 这让 ALU 更大也更复杂一些

但也更厉害 - \N  复杂度 vs 速度 的平衡在计算机发展史上经常出现

举例，现代处理器有专门电路来处理 \N 图形操作, 解码压缩视频, 加密文档 等等

如果用标准操作来实现，要很多个时钟周期. 你可能听过某些处理器有 MMX, 3DNOW, SSE，它们有额外电路做更复杂的操作 用于游戏和加密等场景

指令不断增加，人们一旦习惯了它的便利就很难删掉

所以为了兼容旧指令集，指令数量越来越多

英特尔 4004，第一个集成CPU，有 46 条指令 - 这说明46条指令就足够做一台能用的计算机

但现代处理器有上千条指令，有各种巧妙复杂的电路

超高的时钟速度带来另一个问题 - 如何快速传递数据给 CPU

就像有强大的蒸汽机  但无法快速加煤

RAM 成了瓶颈 RAM 是 CPU 之外的独立组件

意味着数据要用线来传递，叫"总线"

总线可能只有几厘米————别忘了电信号的传输接近光速

但 CPU 每秒可以处理上亿条指令– 很小的延迟也会造成问题

RAM 还需要时间找地址 \N 取数据，配置，输出数据

一条"从内存读数据"的指令可能要多个时钟周期 CPU在这期间就是空等数据

解决延迟的方法之一是 \N 给 CPU 加一点 RAM - 叫"缓存" 因为处理器里空间不大，所以缓存一般只有 KB 或 MB 而 RAM 都是 GB 起步 

缓存提高了速度 CPU 从 RAM 拿数据时 \N RAM 不用传一个，可以传一批
虽然花的时间久一点，但数据可以存在缓存
这很实用，因为数据常常是一个个按顺序处理

举个例子，算餐厅的当日收入
先取 RAM 地址 100 的交易额
RAM 与其只给1个值，直接给一批值
把地址100到200都复制到缓存
当处理器要下一个交易额地址 101时，缓存会说："我已经有了，现在就给你"

不用去 RAM 取数据

因为缓存离 CPU 近, 一个时钟周期就能给数据 - CPU 不用空等！

比反复去 RAM 拿数据快得多

如果想要的数据已经在缓存，叫 缓存命中 如果想要的数据不在缓存，叫 缓存未命中

缓存也可以当临时空间，存一些中间值，适合长/复杂的运算

继续餐馆的例子，假设 CPU 算完了一天的销售额

想把结果存到地址 150

就像之前，数据不是直接存到 RAM

而是存在缓存，这样不但存起来快一些 如果还要接着算，取值也快一些

但这样带来了一个有趣的问题

- 缓存和 RAM 不一致了.

这种不一致必须记录下来，之后要同步
因此缓存里每块空间  有一个特殊标记
叫 "脏位" -- - 这可能是计算机科学家取的最贴切的名字

同步一般发生在 当缓存满了而 CPU 又要缓存时
在清理缓存腾出空间之前，会先检查 "脏位"

如果是"脏"的, 在加载新内容之前, 会把数据写回 RAM

另一种提升性能的方法叫 "指令流水线" 想象下你要洗一整个酒店的床单

但只有 1 个洗衣机, 1 个干燥机

选择1：按顺序来，放洗衣机等 30 分钟洗完
然后拿出湿床单，放进干燥机等 30 分钟烘干

这样1小时洗一批

另外一说：如果你有 30 分钟就能烘干的干燥机
请留言告诉我是什么牌子，我的至少要 90 分钟.

即使有这样的神奇干燥机,  \N 我们可以用"并行处理"进一步提高效率

就像之前，先放一批床单到洗衣机等 30 分钟洗完 然后把湿床单放进干燥机

但这次，与其干等 30 分钟烘干，\N 可以放另一批进洗衣机

让两台机器同时工作

30 分钟后，一批床单完成, 另一批完成一半 另一批准备开始

效率x2！

处理器也可以这样设计

第7集，我们演示了 CPU 按序处理

and in a continuous loop: Fetch-decode-execute, fetch-decode-execute, fetch-decode-execute, and so on
取指 → 解码 → 执行, 不断重复

这种设计，三个时钟周期执行 1 条指令

但因为每个阶段用的是 CPU 的不同部分

意味着可以并行处理！

While one instruction is getting executed, the next instruction could be getting decoded,
"执行"一个指令时，同时"解码"下一个指令

and the instruction beyond that fetched from memory.
"读取"下下个指令

All of these separate processes can overlap
不同任务重叠进行，同时用上 CPU 里所有部分.

so that all parts of the CPU are active at any given time.
不同任务重叠进行，同时用上 CPU 里所有部分.

In this pipelined design, an instruction is executed every single clock cycle
这样的流水线  每个时钟周期执行1个指令

which triples the throughput.
吞吐量 x 3

But just like with caching this can lead to some tricky problems.
和缓存一样，这也会带来一些问题

A big hazard is a dependency in the instructions.
第一个问题是 指令之间的依赖关系

For example, you might fetch something that the currently executing instruction is just about to modify,
举个例子，你在读某个数据 \N 而正在执行的指令会改这个数据

which means you’ll end up with the old value in the pipeline.
也就是说拿的是旧数据

To compensate for this, pipelined processors have to look ahead for data dependencies,
因此流水线处理器  要先弄清数据依赖性

and if necessary, stall their pipelines to avoid problems.
必要时停止流水线，避免出问题

High end processors, like those found in laptops and smartphones,
高端 CPU，比如笔记本和手机里那种

go one step further and can dynamically reorder instructions with dependencies
会更进一步，动态排序 有依赖关系的指令

in order to minimize stalls and keep the pipeline moving,
最小化流水线的停工时间

which is called out-of-order execution.
这叫 "乱序执行"

As you might imagine, the circuits that figure this all out are incredibly complicated.
和你猜的一样，这种电路非常复杂

Nonetheless, pipelining is tremendously effective and almost all processors implement it today.
但因为非常高效，几乎所有现代处理器都有流水线

Another big hazard are conditional jump instructions -- we talked about one example, a JUMP NEGATIVE,last episode.
第二个问题是 "条件跳转"，比如上集的 JUMP NEGATIVE

These instructions can change the execution flow of a program depending on a value.
这些指令会改变程序的执行流

A simple pipelined processor will perform a long stall when it sees a jump instruction,
简单的流水线处理器，看到 JUMP 指令会停一会儿 \N 等待条件值确定下来

waiting for the value to be finalized.
简单的流水线处理器，看到 JUMP 指令会停一会儿 \N 等待条件值确定下来

Only once the jump outcome is known, does the processor start refilling its pipeline.
一旦 JUMP 的结果出了，处理器就继续流水线

But, this can produce long delays, so high-end processors have some tricks to deal with this problem too.
因为空等会造成延迟，所以高端处理器会用一些技巧

Imagine an upcoming jump instruction as a fork in a road - a branch.
可以把 JUMP 想成是 "岔路口"

Advanced CPUs guess which way they are going to go, and start filling their pipeline with
高端 CPU 会猜哪条路的可能性大一些

instructions based off that guess – a technique called speculative execution.
然后提前把指令放进流水线，这叫 "推测执行"

When the jump instruction is finally resolved, if the CPU guessed correctly,
当 JUMP 的结果出了，如果 CPU 猜对了

then the pipeline is already full of the correct instructions and it can motor along without delay.
流水线已经塞满正确指令，可以马上运行

However, if the CPU guessed wrong, it has to discard all its speculative results and
如果 CPU 猜错了，就要清空流水线

perform a pipeline flush - sort of like when you miss a turn and have to do a u-turn to
就像走错路掉头

get back on route, and stop your GPS’s insistent shouting.
让 GPS 不要再！叫！了！

To minimize the effects of these flushes, CPU manufacturers have developed sophisticated
为了尽可能减少清空流水线的次数，CPU 厂商开发了复杂的方法

ways to guess which way branches will go, called branch prediction.
来猜测哪条分支更有可能，叫"分支预测"

Instead of being a 50/50 guess, today’s processors can often guess with over 90% accuracy!
现代 CPU 的正确率超过 90%

In an ideal case, pipelining lets you complete one instruction every single clock cycle,
理想情况下，流水线一个时钟周期完成 1 个指令

but then superscalar processors came along
然后"超标量处理器"出现了，一个时钟周期完成多个指令

which can execute more than one instruction per clock cycle.
然后"超标量处理器"出现了，一个时钟周期完成多个指令

During the execute phase even in a pipelined design,
即便有流水线设计，在指令执行阶段

whole areas of the processor might be totally idle.
处理器里有些区域还是可能会空闲

For example, while executing an instruction that fetches a value from memory,
比如，执行一个 "从内存取值" 指令期间

the ALU is just going to be sitting there, not doing a thing.
ALU 会闲置

So why not fetch-and-decode several instructions at once, and whenever possible, execute instructions
所以一次性处理多条指令（取指令+解码） 会更好.

that require different parts of the CPU all at the same time
如果多条指令要 ALU 的不同部分，就多条同时执行

But we can take this one step further and add duplicate circuitry
我们可以再进一步，加多几个相同的电路 \N 执行出现频次很高的指令

for popular instructions.
我们可以再进一步，加多几个相同的电路 \N 执行出现频次很高的指令

For example, many processors will have four, eight or more identical ALUs,
举例，很多 CPU 有四个, 八个甚至更多 完全相同的ALU

so they can execute many mathematical instructions all in parallel!
可以同时执行多个数学运算

Ok, the techniques we’ve discussed so far primarily optimize the execution throughput
好了，目前说过的方法，都是优化 1 个指令流的吞吐量

of a single stream of instructions,
好了，目前说过的方法，都是优化 1 个指令流的吞吐量

but another way to increase performance is to run several streams of instructions at once
另一个提升性能的方法是 同时运行多个指令流

with multi-core processors.
用多核处理器

You might have heard of dual core or quad core processors.
你应该听过双核或四核处理器

This means there are multiple independent processing units inside of a single CPU chip.
意思是一个 CPU 芯片里，有多个独立处理单元

In many ways, this is very much like having multiple separate CPUs,
很像是有多个独立 CPU

but because they’re tightly integrated, they can share some resources,
但因为它们整合紧密，可以共享一些资源

like cache, allowing the cores to work together on shared computations.
比如缓存，使得多核可以合作运算

But, when more cores just isn’t enough, you can build computers with multiple independent CPUs!
但多核不够时，可以用多个 CPU

High end computers, like the servers streaming this video from YouTube’s datacenter, often
高端计算机，比如现在给你传视频的 Youtube 服务器

need the extra horsepower to keep it silky smooth for the hundreds of people watching simultaneously.
需要更多马力，让上百人能同时流畅观看

Two- and four-processor configuration are the most common right now,
2个或4个CPU是最常见的

but every now and again even that much processing power isn’t enough.
但有时人们有更高的性能要求

So we humans get extra ambitious and build ourselves a supercomputer!
所以造了超级计算机！

If you’re looking to do some really monster calculations
如果要做怪兽级运算

– like simulating the formation of the universe - you’ll need some pretty serious compute power.
比如模拟宇宙形成，你需要强大的计算能力

A few extra processors in a desktop computer just isn’t going to cut it.
给普通台式机加几个 CPU 没什么用

You’re going to need a lot of processors.
你需要很多处理器！

No.. no... even more than that.
不…不…还要更多

A lot more!
更多

When this video was made, the world’s fastest computer was located in
截止至视频发布，世上最快的计算机在

The National Supercomputing Center in Wuxi, China.
中国无锡的国家超算中心

The Sunway TaihuLight contains a brain-melting 40,960 CPUs, each with 256 cores!
神威·太湖之光有 40960 个CPU，每个 CPU 有 256 个核心

Thats over ten million cores in total... and each one of those cores runs at 1.45 gigahertz.
总共超过1千万个核心，每个核心的频率是 1.45GHz

In total, this machine can process 93 Quadrillion -- that’s 93 million-billions
每秒可以进行 9.3 亿亿次浮点数运算

floating point math operations per second, knows as FLOPS.
也叫 每秒浮点运算次数 (FLOPS)

And trust me, that’s a lot of FLOPS!!
相信我  这个速度很可怕

No word on whether it can run Crysis at max settings, but I suspect it might.
没人试过跑最高画质的《孤岛危机》但我估计没问题

So long story short, not only have computer processors gotten a lot faster over the years,
长话短说，这些年处理器不但大大提高了速度

but also a lot more sophisticated, employing all sorts of clever tricks to squeeze out
而且也变得更复杂，用各种技巧

more and more computation per clock cycle.
榨干每个时钟周期 做尽可能多运算

Our job is to wield that incredible processing power to do cool and useful things.
我们的任务是利用这些运算能力，做又酷又实用的事

That’s the essence of programming, which we’ll start discussing next episode.
编程就是为了这个，我们下集说

See you next week.
下周见

