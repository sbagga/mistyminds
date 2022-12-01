---
layout: post
title:  全异步的Mobile Super Runtime构想
categories: [Rust, Async, Runtime, mobile]
comments_id: 10
excerpt: 新的异步并行系统性硬件设计方法和架构的变革对软件系统提出了全新的挑战，尤其是如何有效的支持并行异构计算。

---


# 全异步的Mobile Super Runtime构想




## 高性能并行并发系统软件是支持下一代硬件创新的基础


移动SoC未来技术发展有两种方式，第一种是延长线创新，通过先进的2/3nm制程技术持续获得功耗和性能的改善。另一种通过domain specific 软件+芯片co-design架构，chiplet组合和先进3D封装带来的面积换性能等系统性的创新，在制程技术不变的情况下获得性能和功耗的改善。这种新的系统性硬件设计方法和架构的变革对软件系统提出了全新的挑战，尤其是如何有效的支持并行异构计算。



LLVM编译器和SWIFT语言的发明人Chris Lattner也系统阐述了这种异构并行硬件对编程语言提出的挑战。（https://gist.github.com/yxztj/7744e97eaf8031d673338027d89eea76）

多核CPU（几十到上百核）已经是趋势，传统的OS和编程语言对硬件的抽象是基于共享内存的SMP多核计算模式的，在这种模式下，CPU被抽象为一台单处理器串行处理模式的逻辑硬件，从而使软件开发者仍然可以使用串行的控制流方式来编程，为此OS和编程语言提供了锁、原子化指令等管理共享内存的机制，但是这些软件工具限制了硬件处理性能，也没有简化开发者的负担，并行并发应用开发仍然是非常困难的任务。

多核并发CPU管理cache一致性时，也需要实现硬件锁来提供原子化的内存访问，从而把并行访问变成了串行，硬件系统的内存访问机制如果从强一致性（strong consistancy）变为弱一致性（relaxed consistancy），访问速度可以提升20-100倍。GPU就是一个例子，它的编程模式鼓励开发者使用本地的快速内存而避免使用全局的共享内存。


未来的软件系统应该摆脱这种内存共享范式的限制，而让开发者可以做数据抽象，并发抽象和并很好的理解他们系统，而这样的软件系统可以在运行在多核甚至多机系统上，而不需要每次重新编写程序。

云计算是如何从大型服务器扩展SMP计算的scale up模式走向机群扩展方式的scale out模式的演化历程，和Chris的这些观点遥相呼应，这这个过程中，引入了面向应用的多种数据一致性存储的实现方式，例如：提供事实一致性的NOSQL数据库，从而极大提高了电商应用的交易处理速度，引入了go语言，提供了良好的并发应用抽象，成为高性能云计算基础设施开发的主力语言。

新一代的移动OS需要在编程语言、操作系统和编程框架做系统性的并行并发支持，才能完全释放新一代移动SoC硬件的能力。

## 移动OS并行和并发的场景(需求)和趋势


### 异步并发并行的概念
* Concurrent - 并发

    * 任务可以交替执行（alternatively），计算目标是单处理器，计算任务是重复执行的低占空比（duty cycle，任务处理时间和任务周期时间的比值）计算
    * 典型的并发应用是I/O，网络，数据库。文件系统等所谓的I/O bound任务，即任务的吞吐量取决于数据进出的流量，性能取决于计算系统的数据接口和内存带宽，平均每单位数据上的计算耗时占任务处理时间的比重低

* Parallel - 并行
    * 并行的计算目标是多个处理单元，可以是一个多核处理器，也可以是计算集群系统
    * 典型的并行计算任务是数据处理，数据可以切分到不同的处理单元，不同单元数据处理的指令/程序类似。DSP，GPU等支持SIMD（Single Instruction Multiple Data）的处理器是典型的可并行的处理器。map/reduce等大数据处理算法是典型的可大规模并行的算法。在AI/ML学习任务里，训练模型时候可以采取数据切分，模型切分，参数切分等并行化策略，也是典型的并行计算任务。
* Asynchronous - 异步
    * 和异步处理对应的是同步处理，同步程序的控制流是顺序执行的，上一条指令没有执行完之前不会执行下一条指令，异步程序的控制流不是顺序执行的，程序编写的先后顺序和程序执行的先后顺序不一致。
    * 异步处理的困难主要是如何描述异步处理的逻辑，因为传统的编程语言是配合单处理器，串行控制而设计的，例如：Fortune等，C语言等，程序员员编写的串行程序就是硬件执行的顺序，是一致的。而现代处理器的处理速度非常快，而I/O设备的数据传输速度是一个从低速几KB/s到高速几百GB/s的巨大区间，处理器速度和I/O速度不匹配时，同步阻塞式I/O会浪费大量的处理能力。异步编程是为了解耦处理器处理速度和I/O数据传输速度，提高处理器的利用率。
    * 多核处理器要求对计算任务做切分，导致了数据同步的需求，因为不同处理器对内存系统的访问速度差异，处理任务速度的差异，数据同步过程必然是异步的，核数越多、内存访问速度差异越大，异步处理能带来的计算效率收益越大。
    * 异构并行处理器也是芯片技术发展的趋势，异构计算体系包含了不同处理速度的面向应用的加速器，主CPU的控制程序对这些异构处理器的访问也是异步的，类似对I/O的访问，按照异构处理器处理速度异步地及时提供数据输入和输出管理，可以充分利用主CPU和异构加速器的处理能力。

### 移动OS的程序控制类型
1. 传统的同步串行控制流程
    * 大部分程序员熟悉的顺序执行的编程模式
    * 移动应用对处理器功耗和利用效率有极高要求，移动应用有大量异步并行和并发的场景，串行编程和执行方式难以让异步I/O和并行异构处理器获得满意的运行效率。
2. 异步并发控制流程
    * 移动应用的异步计算场景包括：I/O访问，数据库访问，UI前端开发，2D并行图形渲染，游戏脚本开发等
    * 异步编程方式破坏了程序的顺序执行顺序，编程语言必须引入新的语意来描述程序并发、数据同步、错误处理等问题，提供给用户调试异步应用的工具，解决异步并发应用难以编写和调试，用户体验差的问题。
    * 不仅仅是用户程序，移动OS也需要重构，提供全异步的I/O模式的设备驱动程序，甚至内核完全使用异步并发并行的微内核方式编写，这样才能构成一个全异步并发的完整软件栈。例如：Linux最近引入的异步IO_URING机制。
    * Async/Await 成为异步并发程序的标准语意，可以以串行方式写并发应用，主流语言均已支持（Swift，JS，Python，Kotlin，Rust）。Async/Await的完整实现需要基于语言自带异步Runtime或者生态实现的Runtime框架来提供的异步任务调度机制。
3. 消息传递和数据隔离
    * 主要解决需要可变的共享内存状态的场景，例如：前端UI开发，高异步并行化的I/O bound和CPU bound的混合应用，例如：2D/3D渲染业务
    * 通过消息机制来实现数据隔离，提供对共享状态的安全并发访问，所谓的actor模式，是实现并发编程的另一种主流模式
    * 发端于erlang语言，Golang的最大特色就是实现了基于Actor的goroutine，取得了很好的效果，Swift也全面支持Actor
4. 分布式数据和计算
    * 使用场景：音视频媒体处理、GPU图像处理、AI/ML，物理引擎等
    * 主要是数据密集型的SIMD计算模式，解决数据切分，计算切分问题
    * 通常这类问题可以通过声明式编程方式解决，即用户提供基于DAG等抽象数据结构的计算图，由Compiler和Runtime共同来完成数据的分割和任务的调度，例如：Android的RenderScript，Taskgraph，Halide等声明式的并行计算框架。

### 异步并发软件框架的几个概念
1. CPU core vs OS thread vs User Space Thread （Green Task， 或者fiber，light-weigh user sapce thread）

    * CPU core是OS对多核CPU硬件的抽象，OS调度程序运行到CPU core上去
    * OS thread是OS提供给程序的CPU抽象，对程序员而言，thread是可以运行代码的虚拟CPU，thread有自己的寄存器、stack和heap内存，可以被OS调度到物理CPU上运行；OS支持程序创建多个thread，创建每个thread都要消耗计算机的内存资源，例如：在linux创建一个thread对象需要分配8MB的虚拟内存，主要用于stack，OS可以运行的thread远远大于实际的物理CPU核数，OS可以随时打断正在运行的thread，调度一个新的thread运行，thread被中断时需要存储寄存器和stack，再次被OS调度运行的时候，thread需要恢复上次被打断时候的运行场景，包括stack内存和寄存器，以便从上次被打断的点开始继续运行。
    * OS process是对硬件资源的抽象和隔离，对程序员而言，process相当于一台虚拟计算机，有自己独立的内存空间，驱动程序，文件系统，网络系统等，OS提供了资源隔离机制来支持在单个活多个CPU Core上同时运行多个process，这些process在OS的调度下轮换和并发使用计算机硬件资源。同一个process可以运行多个thread，这些thread共享process的资源，thread有独立的stack内存，但是共享heap内存，thread之间可以通过共享内存来通信。thread运行出现的问题，例如内存越界访问，可能导致process内的其他thread崩溃，因为它们之间没有内存和资源隔离机制，但是不会影响其他process的运行。
    * thread的运行取决于外部条件，例如：读写I/O，或者等待数据同步，多thread设计可以让OS根据thread的运行状态来调度准备好的thread在CPU上运行，等待状态的thread被挂起，不占用CPU资源，这样可以充分利用CPU资源。OS还提供支持异构并行并发的数据同步API，例如：POSIX的mutex，semaphore等原子操作，select和epoll等事件驱动的I/O API，配合thread实现异步并发并行能力。
    * 不同的编程语言都提供使用OS原生的多线程异步并发并行能力的语言API，例如：POSIX API本身就是C语言实现的，其他语言都有POSIX API 的wrapper
    * 供给开发者一个安全易用的异步并行并发语言表达和API是编程语言的主要功能，例如：函数调度为基于线程的异步执行，多线程函数之间的数据同步，线程的生命周期管理。
    * 现代编程语言，如 go，C#，javascript，python都带有一个虚拟机或者runtime，用于支持安全动态的内存使用，也能完整控制用户程序的运行行为，所以发展出了所谓的用户态线程和用户态的任务调度机制，为了区别 OS thread，这类用户态的thread一般称为green task（coroutine）。green task提供给用户和编程语言特性融合的异步多任务控制流表达能力，编程语言Runtime则实现了green task到OS thread和CPU core的映射，这种映射中，OS thread可以和CPU Core做绑定，叫做CPU affinity，这样OS就不会把已经映射的thread调度到其他CPU core上。这种映射可以是1:1映射，即green task不会被调度到多于一个CPU core，也可以是N:M模式，即green task可以被调度到多于一个CPU core上运行。Runtime的映射策略可以是静态的也可以是动态的。

* Preemptive vs Cooperative scheduling, Stackful vs Stackless task
    * 语言Runtime实现green task的调度有两种策略，一种是协作式的（cooperative），即green task的运行不会被Runtime打断，会一直运行到结束为止，即run to complete模式，这种情况下Runtime不需要为green task做function stack的完全备份和恢复，因为任务总是会正常退出的，这样runtime的设计会比较简单，缺点是green task必须自觉把CPU让出给Runtime（yield），如果不让出，任务可以一直运行，导致其他任务得不到运行时间，甚至锁死系统。而且，任务调度需要实现一些资源共享策略，比如平均使用CPU时间，或者某类QoS SLA，Runtime如果不能随时切换green task，则不能保证任务的QoS。
    * 另一种是抢夺式的任务调度（preemptive），即Runtime可以中断当前运行的green task，置换为另一个green task运行。因此，Runtime需要对green task的呼叫现场，包括stack和registers做备份和恢复，Runtime可以直接利用OS thread的conext swtich能力（作为cooperative task scheduling的补充），这种方法随时可以切换green task，缺点是OS thread的调度代价比较高。也可以通过编译器在特定的程序控制部分插入Runtime的指令实现Runtime对用户green task的接管，而实现调度策略，优点是Runtime自己实现context switch，可以优化备份内存的使用，缺点是需要程序知道何处可以插入Runtime调度点，对使用者不透明，而且并不能完全实现preemptive调度的优点。goroutine最新的调度实现中使用OS signal来中断用户green task，Runtime通过signal handler来插入调度算法，因为GO有GC，所以还可以判断目前的中断位置是否是GC的safe-points，如果不是则退出中断，继续运行用户green task，如果可以调度，则Runtime保存现有任务现场，调度入新green task，这是一种cooperative和preemptive调度机制的折中实现（non-cooperative scheduling），即达到了可以随时调度任务，又兼顾了尽可降低保存任务现场的代价。(https://github.com/golang/proposal/blob/master/design/24543-non-cooperative-preemption.md)。

    
    * 下表比较了基于OS thread和async的green task的创建和调度开销。 OS thread需要MB级别的存储空间，而green task得内存是KB级别或者更少，所以同样的内存green task的数目可以是CPU tread的上千倍，async task的任务粒度更小，调度的代价和时延也更小，可以更好的利用CPU资源。
https://kerkour.com/cooperative-vs-preemptive-scheduling

        | operation           | async            | thread           |
        |----------------|------------------|------------------|
        | Creation       | 0.3 microseconds | 17 microseconds  |
        | Context switch | 0.2 microseconds | 1.7 microseconds |

* Stackful and Stackless

    * Stackless Runtime模式是指Runtime不需要为green task动态分配stack，主要依赖编译器对代码调用关系的分析，把异步控制流程转换为状态机实现，分析异步调用closure的上下文使用的局部变量，做保存和恢复，这种实现方式下Runtime不管理程序stack，stack的尺寸是静态分配好的，不能动态变化，会对程序编写有一定的限制，例如：不支持递归，不支持在stack上动态分配内存(https://lunatic.solutions/blog/rust-without-the-async-hard-part/)。
    * Stackful Runtime模式是指Runtime具备动态管理green task stack的能力，如果green task使用的stack内存超过设计的门限，Runtime会动态增长stack大小，设计的重点是如何实现dynamic stack growth（出现了不同的设计思路，segmented stack，guarded stack等），即如何最经济的使用stack空间，比较小的stack size也意味着更快的任务切换时间，更好的计算利用率，运行比OS thread更多的green task，这种方式下green task的使用更类似OS thread。


* Language idiom and runtime
    * 编程语言提供的fork/join，async/await，actor等语意的目的都是为开发者提供更好的人机交互（ergonomics），而且也逐渐形成了业界通行的标准，任何一个新的语言都需要考虑兼容这些语言设计上的事实标准
    * 语言实现这些语意需要编译器和runtime的配合，甚至需要进一步提供异步的设备驱动和协议栈实现，才能真正实现异步并发并行。Runtime因为在用户态实现，所以不依赖OS kernel的限制和约束，可以快速创新，是语言核心竞争力的体现。例如：go语言的goroutine基本借鉴了erlang相应的语言设计，但是其高性能核心是其runtime实现的N:M的preemptive scheduling，和HTTP等核心协议库。

## Rust带来异步并行并发Mobile OS弯道超车的机会

> 
> C++ implementations obey the zero-overhead principle: What you don't use, you don't pay for [Stroustrup, 1994]. And further: What you do use, you couldn't hand code any better.
> 
> -- Stroustrup
> 


1. ### zero-overhead abstraction，尽可能在编译时间优化，而不要付出运行时的代价

* 内存ownership的机制实现内存不可能越界访问，因此运行时不需要做内存越界检查（C/C++没有这个特性，WASM必须付出运行时的代价）
* 内置并行并发特性，共享内存的场景使用严格受控，避免无必要的内存锁设计造成的性能损失
* 语言核心特性严格管理，通过traits扩展功能（inheritance vs composition，C++ OOP失败的设计），形成良好内核抽象稳定、library生态活跃创新的格局，Chris Lattner特别强调的small things that compose的格局，避免了微软 fiber，Apple GCD等中心化创新的困境（无法通过大规模试验获取经验，迭代创新），出现了tokio这样的活跃library生态

* 基于future trait实现的Async/Await被编译器静态展开为有限状态机，避免运行时动态申请内存的开销


2. ### 基于Rust的异步并发的Runtime生态创新极其活跃

Christ Lattner列出的三种并发并行控制流，在Rust生态都有对应的项目

21.  #### 异步并发控制流：Rust在2019年发布的1.39版正式内置了Async/Await语意支持，自此发展出非常活跃蓬勃的runtime生态



如下表所示，async/await已经被主流编程语言内置支持，成为异步并发应用编写的标准。



https://shahbhat.medium.com/structured-concurrency-in-modern-programming-languages-part-iv-kotlin-and-swift-7bf0e08de1dd



| Feature                                 |      Typescript (NodeJS)      |                         GO                        |                                              Rust |                       Kotlin                      |                                             Swift |
|-----------------------------------------|:-----------------------------:|:-------------------------------------------------:|--------------------------------------------------:|:-------------------------------------------------:|--------------------------------------------------:|
| Structured scope                        |            Built-in           |                      manually                     |                                          Built-in |                      Built-in                     |                                          Built-in |
| Asynchronous Composition                |              Yes              |                         No                        |                                               Yes |                        Yes                        |                                               Yes |
| Error Handling                          |   Natively using Exceptions   |        Manually storing errors in Response        |                         Manually using Result ADT |             Natively using Exceptions             |                         Natively using Exceptions |
| Cancellation                            |    Cooperative Cancellation   | Built-in Cancellation or Cooperative Cancellation | Built-in Cancellation or Cooperative Cancellation | Built-in Cancellation or Cooperative Cancellation | Built-in Cancellation or Cooperative Cancellation |
| Timeout                                 |               No              |                        Yes                        |                                               Yes |                        Yes                        |                                               Yes |
| Customized Execution Context            |               No              |                         No                        |                                                No |                        Yes                        |                                               Yes |
| Race Conditions                         | No due to NodeJS architecture |            Possible due to shared state           |                        No due to strong ownership |            Possible due to shared state           |                      Possible due to shared state |
| Value Types                             |               No              |                        Yes                        |                                               Yes |                        Yes                        |                                               Yes |
| Concurrency paradigms                   |           Event loop          |              Go-routine, CSP channels             |                              OS-Thread, coroutine |         OS-Thread, coroutine, CSP channels        |     OS-Thread, GCD queues, coroutine, Actor model |
| Type Checking                           |             Static            |             Static but lacks generics             |               Strongly static types with generics |        Strongly static types with generics        |               Strongly static types with generics |
| Suspends Async code using Continuations |               No              |                         No                        |                                               Yes |                        Yes                        |                                               Yes |
| Zero-cost based abstraction ( async)    |               No              |                         No                        |                                               Yes |                         No                        |                                                No |
| Memory Management                       |               GC              |                         GC                        |            (Automated) Reference counting, Boxing |                         GC                        |                      Automated reference counting |


GO语言的goroutine实现，不仅包括了语意语法设计，也实现了完整的async runtime来完成preemptive的任务调度，同时提供了http这类急基础lib的实现。同样，swift的asyc/await实现包括也包括了基于软件task cooperative调度模式的runtime，这些都是所谓battery included 语言设计。

相比其他语言，Rust语言的Runtime只提供对OS适配的基础std library，并不自带一个支持async/await的Runtime，Rust的async/await只是语法糖实现，底层的支持是future trait，用户需要提供自己的async Runtime来支持异构并发，如果没有用户提供的Runtime，Rust编译器自动展开的async/await和普通的串行函数调用是一样的。

这样做的出发点还是zero-overhead abstraction，如果用户程序不使用异步并发，则不需要付出异步Runtime的开销，如果需要支持异步并发，则可以通过生态实现，可以针对应用的特点实现不同的Runtime调度机制，例如：基于I/O bound和CPU bound任务可以使用的不同的 Runtime，Runtime可以实现基于cooperative task的调度机制，也可以实现基于preemptive的调度。缺点是，可能会出现多个竞争性的生态，用户选择太多导致无所适从。目前已经出现的async/await runtime就有6-7种
tokio，async-std，smol，monoio，glommio，lunatic

如下图所示，一个完整的Rust Async Runtime实现需要包含3各部分

async function被编译器转化为state machine，需要实现future trait的poll function，要把自己注册到runtime的executor上，executor会调用poll，同时要注册事件给Runtime的reactor，reactor是一个基于内核epoll和队列实现的事件引擎，事件到达之后reactor会通知executor，executor会调用async function的poll，驱动状态机状态迁移。


![](https://i.imgur.com/BYguphF.png)


Rust提供的基于软件task的异步并发特性被用于domain specific的library开发。例如：tokio，lunatic，

2. #### 消息传递和数据隔离：级所谓的actor模式。

Rust内置支持msg channel概念，例如：MPSC（mulitple producer single consumer） channel。生态中，Lunatic Runtime试图对标goroutine实现完整的actor模式和preemtive任务调度系统。Rust最初的版本包括preemptive调度的green task特性，非常类似Microsoft的用户态任务调度体系Fiber，后续版本删除了这个特性，因为希望由生态Runtime Library来实现，而不需要在语言层实现。

3. #### 分布式数据和计算

Rust的Ryaon并行计算库实现了声明式的，基于fork/join模式的并行计算，可以用于排序、map/reduce类型的数据并行化处理。

Rust社区已经在考虑是否要进一步标准化Runtime来避免目前比较分化的格局，如何在标准化和鼓励创新上取舍。
另外，当前I/O bound和CPU bound的runtime由不同的生态实现，API不一致，容易造成开发困难，是否可以进一步统一这两个不同的runtime社区。

3. ### Rust大举进入操作系统。即将成为开发OS的主要语言
* Rust正式成为Linux Kernel的开发语言，关键的kernel library会重写，包括ISRG Prossimo(https://www.memorysafety.org/about/)项目赞助的使用Rust重写Linux Kernel和关键library（TLS，NTP等），OpenSSF提出的用安全语言（Rust，Go）重写C/C++b编写的基础软件的项目，是增大kernel话语权的机会。
* 基于Rust开发的新型OS，例如：theseus OS利用Rust zero-overhead abstraction带来的内存隔离，摆脱了传统OS通过kernel和用户态的内存隔离，用户态进程之间的虚拟内存隔离带来的开销，实现了single address space的内存使用机制
* 基于Rust开发的应用I/O Kernel，例如：quark，配合Linux Kernel的新异步I/O机制IO_URING实现了高性能的I/O虚拟化

![](https://i.imgur.com/vfOq1j6.png)


![](https://i.imgur.com/D46NvXT.png)


综上所述，Rust zero-overhead abstraction 的设计理念强力支持了OS开发需要的安全、可靠、高效、高性能等需求，实现了对C/C++的跨代式超越，无论在Linux还是新型OS领域都会获得大量使用，是改变既有OS格局的重要武器。Rust提供的内存ownership模式，改变了操作系统传统基于硬件或者runtime实现的资源隔离，同时提供安全可靠和有优秀用户体验的异步并发软件开发能力，可以实现从OS到应用层的全栈异步并行开发，已经证明可以更加有效利用多核处理器，为颠覆性的OS创新提供了沃土。


## 基于Rust的实现的Super Runtime

### 需求

1. 对既有mobile OS生态的支持，完全不继承既有生态和代码资产不现实
2. 又需要实现全栈式（编程语言、OS、应用框架、开发者体验）的颠覆式创新，才能配合新的面积换性能硬件架构
    * 同样内存条件下软件task数量是OS线程的1000倍，任务调度切换速度比OS线程快100倍，也就是基于编程语言实现的异步并发比传统移动OS基于线程的异步并行并发计算更能提升CPU的使用效率
    * 

### 分析
1. 采取虚拟化的技术，新旧架构并存，逐步切换
2. 既有的虚拟化技术开销巨大，例如：VM，container，而且很难保证新旧系统在用户体验上的一致性
3. 采用Rust zero-overhead abstraction 提供的“虚拟化”，和既有OS生态共存，但是又可以overlay在既有的OS之上，实现OS大部分的能力，类似overlay网络虚拟化方案。

![](https://i.imgur.com/FiReY7c.png)



### 方案要点

1. OS 做基本的hardware abstraction
    * CPU, thread library
    * IO_URING
    * I/O tap device，OS只提供基础的数据进出和简单控制，协议栈实现在用户空间实现
2.  OS把CPU core和内存分配给Super Runtime（SR），并通过OS affinity锁定，实现对资源的一次分配
    * SR是OS的一个process，和OS上的其他process/apps并存，SR上的green task（GT）可以通过IPC等机制和其他App实现通信。
    * SR内部的GT看到的是平坦内存空间，GT直接可以通过共享内存通信。
4.  SR是GT的OS，对GT实现资源的二次分配
    *  SR实现了对GT的资源管理、任务调度、系统服务核设备驱动，所以SR可以认为是GT的OS
    * SR管理GT，SR对GT提供了基本OS的功能，包括内存管理（green task的内存和stack都在SR的内存中），GT的任务调度机，GT可以按照QoS策略调度，SR提供了用户态的设备驱动、协议栈等lib，GT共享这些lib。
    *  GT基于Rust crate实现，GT之间的内存隔离是编译时间静态的划分的，通过Rust的内存ownership和borrow checker保证了GT不会对内存越界访问。
    *  一个SR的App包括多个GT，一个GT包括多个Crate，Crate是SR分发和运行管理的最小单位，运行时的Crate称为Cell，Crate以Obj方式分发，被SR动态加载，动态链接（https://www.usenix.org/conference/osdi20/presentation/boos）
    *  因此用户App核保护的GT也可以被SR动态加载，动态调度
    *  GT被实际调度到底层OS的thread上运行，SR对GT的调度时用户态的调度。
    * SR还可以集成例如quark这样的I/O虚拟化能力， 从而可以精细化管理用户App对系统I/O使用时的命名空间和资源使用隔离，可以加入capability based security 策略。

5. SR支持Domain Specific Runtime
    * Rust社区面向不同的应用场景已经出现了繁荣的异步Domain Specific Runtime（DSR）生态（网络、计算、前端I/O），这些生态都有自己的Runtime，实现了面向domain应用的优化调度策略，这些生态也都提供了基础的网络、文件系统等用户态的异步lib，这些Runtime已经在domain具备了生态影响了。
    * 但是面向移动应用，这些Runtime都有缺陷，因为移动应用需要支持大量第三方应用同时运行，这些第三方应用内部既有I/O bound的任务，也有CPU Bound的任务；而DSR设计的场景是cloud和服务器，即为single App的功能提供最大的throughput或者计算量，因此可以独占大量的CPU资源，不存在支持大量第三方应用的需求。如果有，也是通过硬件虚拟化完成，总之DSR面向的是单租户场景。而移动应用是多租户和多并发并行场景。
    * SR提供了一条让DSR支持多租户的技术线路，因为SR带来的GT之间的内存和I/O隔离，不同App的GT可以在同一个DSR上运行，即DSR可以支持多租户，即按照DSR原有设计模式，每个OS App自带一个DSR，如果有多个App，则会出现多个DSR同时在不同的OS process运行，这样DSR可以独占CPU的假设不成立了，而因为SR的隔离特性，这些App现在可以同时在SR的process运行，App和App共享DSR，也共享内存和用户态的协议栈lib和系统设备驱动lib。不同的DSR仍然可以通过SR的二次资源分配获得固定的CPU Core资源。 SR在OS和DSR之间加入了用户态OS的基础功能。
6. 挑战
    * 移动应用通常有严格的QoS机制，例如：Swift提供的4级QoS，首先有前台应用和后台应用的区分，前台应用是目前用户正在使用的App，那么属于这个App的所有GT的优先级需要提升，App内部，做UI渲染的GT优先级最高，而且有60fps这样的QoS需要，所以这类UI GT需要能随时抢夺CPU，App内部的background GT的优先级最低，但是也高于所有后台App的GT。另外，移动SoC内部还要非常强的energy efficiency的QoS需求，有追求性能的大核，和追求能效的小核，App的GT要根据应用场景来决定到底使用性能核还是能效核。目前DSR的调度策略都不支持QoS的分级，基本都是cooperative方式的调度，并且thread和core的锁定关系是固定的。所以SR可能需要进一步接管DSR的调度功能，才能实现移动场景复杂的基于QoS和能效的调度需求。

7. SR应用开发
    * 如前所述，SR可以充分利用DSR的生态，开发一个Rust App的时候，可以利用tokio，rayon，yew等面向不同应用控制场景的DSR，SR对App时透明的，App运行的时候也是之间运行在DSR之上。
    * SR基于Crate开发和构建，尤其大量使用DSR提供的Crate，SR对App开发提供了一条完整的DevOps工具链，App开发者需要遵从SR的开发规范开发、构建和部署，和普通的Rust App开发的差异很小。
