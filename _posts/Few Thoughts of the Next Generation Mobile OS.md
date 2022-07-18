# 下一代移动操作系统的构想




## 高性能并行并发系统软件是支持下一代硬件创新的基础


移动SoC未来技术发展有两种方式，第一种是延长线创新，通过先进的2/3nm制程技术持续获得功耗和性能的改善。另一种通过domain specific 软件+芯片co-design架构，chiplet组合和先进3D封装带来的面积换性能等系统性的创新，在制程技术不变的情况下获得性能和功耗的改善。这种新的系统性硬件设计方法和架构的变革对软件系统提出了全新的挑战，尤其是如何有效的支持并行异构计算。

LLVM编译器和SWIFT语言的发明人Chris Lattner也系统阐述了这种异构并行硬件对编程语言提出的挑战。（https://gist.github.com/yxztj/7744e97eaf8031d673338027d89eea76）

多核CPU（几十到上百核）已经是趋势，传统的OS和编程语言对硬件的抽象是基于共享内存的SMP多核计算模式的，在这种模式下，CPU被抽象为一台单处理模式的逻辑硬件，从而使软件开发者仍然可以使用串行的控制流方式来编程，为此OS核编程语言提供了锁、原子化指令等管理共享内存的机制，但是这些软件工具限制了硬件处理性能，也没有简化开发者的负担，并行并发应用开发是非常困难的任务。

多核并发CPU管理cache一致性时，也需要实现硬件锁来提供原子化的内存访问，从而把并行访问变成了串行，硬件系统的内存访问机制如果从强一致性（strong consistancy）变为弱一致性（relaxed consistancy），内存访问速度可以提升20-100倍。GPU就是一个例子，它的编程模式鼓励开发者使用本地的快速内存而避免使用全局的共享内存。软件系统需要配合硬件创新的变革。


未来的软件系统应该摆脱这种内存共享范式的限制，而让开发者可以做数据抽象，并发抽象和并很好的理解他们系统，而这样的软件系统可以在运行在多核甚至多机系统上，而不需要每次重新编写程序。

回顾云计算是如何从scale模式走向scale out模式的历程，和Chris的这些观点遥相呼应，这这个过程中，引入了面向应用的多种数据一致性实现方式，例如：提供事实一致性的NOSQL数据库，从而极大提高了电商应用的交易处理速度，引入了go语言，提供了良好的并发应用抽象，成为高性能云计算基础设施开发的主力语言。

新一代的移动OS需要在编程语言、操作系统和编程框架做系统性的并行并发支持，才能完全释放新一代移动SoC的能力。

## 移动OS并行和并发的场景(需求)和趋势

### 异步并发并行的概念
* Concurrent - 并发

    * 任务可以交替执行（alternatively），计算目标是单处理器，计算任务是重复执行的低占空比（duty cycle，任务处理时间和任务周期时间的比值）计算
    * 典型的并发应用是I/O，网络，数据库。文件系统等所谓的I/O bound任务，即任务的吞吐量取决于数据进出的流量，性能取决于计算系统的数据接口和内存带宽，平均每单位数据上的计算耗时占任务处理时间的比重低

* Parallel - 并行
    * 并行的计算目标是多个处理单元，可以是一个多核处理器，也可以是计算集群系统
    * 典型的并行计算任务是数据处理，数据可以切分到不同的处理单元，不同单元数据处理的指令/程序类似。在处理器内部，DSP，GPU等支持SIMD（Single Instruction Multiple Data）的处理器是典型的可并行的处理器，在大数据处理里，map/reduce算法是典型的可大规模并行的数据处理算法。在AI/ML学习任务里，训练模型时候可以采取数据切分，模型切分，参数切分等并行化策略，也是典型的并行计算任务。
* Asynchronous - 异步
    * 和异步处理对应的是同步处理，同步程序的控制流是顺序执行的，上一条指令没有执行完之前不会执行下一条指令，异步程序的控制流不是顺序执行的，程序编写的先后顺序和程序执行的先后顺序不一致。
    * 异步处理的困难主要是因为程序开发者如何描述异步处理的逻辑，因为传统的编程语言是配合单处理器，串行控制流硬件而设计的，例如：Fortune等，C语言等，程序员员编写的串行控制流就是硬件执行的顺序，是一致的。而现代处理器的处理速度非常快，而I/O设备的数据传输速度是一个从低速几KB/s到高速几百GB/s的巨大区间，处理器速度和I/O速度不匹配时，同步阻塞式I/O会浪费大量的处理能力。异步编程是为了解耦处理器处理速度和I/O数据传输速度，提高处理器的利用率。
    * 异构并行处理器也是芯片技术发展的趋势，异构计算体系包含了不同处理速度的多核处理器的面向应用的加速器，控制程序对这些异构处理器的访问也是异步的，类似对I/O的访问，按照异构处理器处理速度及时提供数据流。

### 移动OS的程序控制类型
1. 传统的同步串行控制流程
    * 大部分程序员熟悉的顺序执行的编程模式
    * 移动应用对处理器功耗和利用效率有极高要求，移动应用有大量异步并行和并发的场景，串行编程和执行方式难以发挥底层的异步I/O和并行化异构处理器。
2. 异步并发控制流程
    * 移动应用的异步计算场景包括：I/O访问，数据库访问，UI前端开发，2D并行图形渲染，游戏脚本开发等
    * 异步编程方式破坏了程序的顺序执行顺序，必须引入新的语意来描述程序并发，数据同步，错误处理等问题，提供给用户调试异步应用的工具，解决异步并发应用难以编写和调试，用户体验差的问题。
    * 不仅仅是用户程序，移动OS也需要重构，提供全异步的I/O模式的设备驱动程序，甚至内核完全使用异步并发并行的微内核方式编写，这样才能构成一个全异步并发的完整软件栈。
    * Async/Await 成为异步并发程序的标准语意，可以以串行方式写并发应用，主流语言均已支持（Swift，JS，Python，Kotlin，Rust）。Async/Await的完整实现需要基于语言自带或者生态框架提供的任务调度机制实现。
3. 消息传递和数据隔离
    * 主要解决带有共享的可改变内存状态的场景，例如：前端UI开发，高异步并行化的I/O bound和CPU bound得应用，例如：2D/3D渲染业务
    * 通过消息机制来实现数据隔离，提供对共享状态的安全并发访问，所谓的actor模式，是实现并发编程的另一种主流模式
    * 发端于erlang语言，Go的最大特色就是实现了基于Actor的goroutine，取得了很好的效果，Swift也全面支持Actor，Rust通过channel机制支持
4. 分布式数据和计算
    * 使用场景：音视频媒体处理、GPU图像处理、AI/ML，物理引擎等
    * 主要是数据密集型的SIMD计算模式，解决数据切分，计算切分问题
    * 通常这类问题可以通过声明式编程方式解决，即用户提供基于DAG等抽象数据结构的计算图，由Compiler和Runtime共同来完成数据的分割和任务的调度，例如：renderscript，taskgraph，halide等并行计算框架。

### 异步并发软件框架的几个概念
1. OS thread vs user space thread （green task）


    * OS把多核处理器抽象为thread软件对象，并提供基于thread的异步并行并发的调度机制，例如：POSIX提供的pthread抽象
    * OS提供支持异构并行并发的数据同步API，例如：POSIX的mutex，semaphore等原子操作
    * OS提供select和epoll等事件驱动的I/O API，配合thread实现异步并发并行能力。
    * 不同的编程语言都提供使用OS原生的异步并发并行能力的语言API，例如：POSIX API本身就是C语言实现的，
    * 编程语言的功能是是提供给开发者一个安全易用的异步并行并发语言表达和API，例如：函数调度为基于线程的异步执行，多线程函数之间的数据同步，线程的生命周期管理。
    * C/C++之后发展出来的编程语言例如：java，C#，javascript，python都带有一个复杂的runtime，用于支持安全动态的内存使用，也能完整的控制用户程序运行行为，所以发展出了所谓的用户态线程和用户态的任务调度机制，为了区别 OS thread，这类用户态的thread一般称为green task（coroutine）。green task提供给用户和编程语言特性融合的多任务抽象，编程语言runtime则实现了green task到CPU thread的映射，这种映射可以是1:1映射，即green task不会被调度到多于一个CPU thread，也可以是N:M模式，即green task可以被调度到多于一个CPU thread上运行。Runtime的映射策略可以是静态的也可以是动态的。

* preemptive vs cooperative, stackful vs stackless task
    * 语言Runtime实现green task的调度有两种策略，一种是协作式的（cooperative），即green task的运行不会被Runtime打断，会一直运行到结束为止，即run to complete模式，这种情况下Runtime不需要为green task做function stack的完全备份和恢复，因为任务总是会正常退出的，这样runtime得设计会比较简单，缺点是green task必须自觉把CPU让出给Runtime（yield），如果不让出，任务可以一直运行，导致其他任务得不到运行时间。而且在需要保证任务实时QoS的情况下，Runtime如果不能随时切换green task，则不能保证任务的QoS。
    * 另一种是抢夺式的任务调度（preemptive），即Runtime可以随时中断当前运行的green task，置换为另一个green task运行，即stackful runtime。因此，Runtime需要对green task的function stack做备份和恢复，Runtime可以直接利用OS thread能力（作为cooperative task scheduling的补充），也可以实现自己的用户态的stack，设计的重点是如何实现dynamic stack growth，即如何最经济的使用stack空间，运行比OS thread更多的green task，例如：Linux 的thread stack缺省是8MB，在用户设备上最多实现1000-2000个thread，而goroutine的stack则是8KB起，这样goroutine的task密度是Linux thread的1000倍，比较小的stack size也意味着更快的任务切换时间，更好的计算利用率。
* language idiom and runtime
    * 编程语言提供的join/fork，async/await，actor等语意的目的都是为开发者提供更好的人机交互（ergonomics），而且也逐渐形成了业界通行的标准，任何一个新的语言都需要考虑兼容这些语言设计上的事实标准
    * 语言实现这些语意需要编译器和runtime的配合，甚至需要进一步提供异步的设备驱动和协议栈实现，才能真正实现异步并发并行。Runtime因为在用户态实现，所以可以不依赖OS kernel的限制和约束，可以快速创新，是语言核心竞争力的体现。例如：go语言的goroutine基本借鉴了erlang相应的语言设计，但是其高性能核心是其runtime实现的N:M的preemptive scheduling，和HTTP等核心协议库。

## Rust带来异步并行并发Mobile OS弯道超车的机会

> 
> C++ implementations obey the zero-overhead principle: What you don't use, you don't pay for [Stroustrup, 1994]. And further: What you do use, you couldn't hand code any better.
> 
> -- Stroustrup
> 


1. ### zero-overhead abstraction，尽可能在编译时间优化，而不要付出运行时的代价
* 内存ownership的机制实现内存不可能越界访问，因此运行时不需要做内存越界检查（C/C++没有这个特性，WASM必须付出运行时的代价）
* 内置并行并发特性，共享内存的场景使用严格受控，避免无必要的内存锁设计造成的性能损失
* 基于future实现的Async/Await被编译器静态展开为有限状态机，避免运行时动态申请内存的开销
* 语言核心特性严格管理，通过traits扩展功能（inheritance vs composition，C++ OOP失败的设计），形成良好内核抽象稳定、library生态活跃创新的格局，Chris Lattner特别强调的small things that compose的格局，避免了微软 fiber，Apple GCD等中心化创新的困境（无法通过大规模试验获取经验，迭代创新），出现了tokio这样的活跃library生态


2. ### 基于Rust的异步并发的Runtime生态创新极其活跃

Christ Lattner列出的三种并发并行控制流，在Rust生态都有对应的项目
* 异步并发控制流：Rust在2019年发布的1.39版正式内置了Async/Await语义。

基于async的软件task或者coroutine一般采取cooperative的调度模式，所以不需要为随机的context switch而准备call stack的存储和恢复，下表比较了基于CPU线程和async task的创建和调度开销。同时CPU线程需要MB级别的存储空间，而async task得内存是KB级别或者更少，所以同样的内存async task的数目可以是CPU tread的上千倍，async task的任务粒度更小，调度的代价和时延也更小，可以更好的利用CPU资源。

这些library runtime提供了用户态的基于软件task的调度和管理能力，比OS的线程调度机制内存占用少、任务调度代价低，例如：OS的支持的thread只有几千个，而runtime可以调度的软件task可以达到10万以上，在domain问题领域能更为高效的利用硬件资源。
https://kerkour.com/cooperative-vs-preemptive-scheduling

| ation          | async            | thread           |
|----------------|------------------|------------------|
| Creation       | 0.3 microseconds | 17 microseconds  |
| Context switch | 0.2 microseconds | 1.7 microseconds |



如下表所示，async/await已经被主流编程语言内置支持，成为异步并发应用编写的标准。


相比其他语言实现的Aysnc/Await，Rust的Async/Await由编译器直接转化为state machine，提前划分了内存，所以并不需要特别的runtime支持，即所谓的zero-overhead。

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


Rust Async/Await存在的问题

GO语言的goroutine实现，不仅包括了语言层面的API，同时也实现了完整的runtime来完成preemptive的任务调度，同时也提供了http这类主流lib的实现。同样，swift的asyc/await实现包括也包括了基于软件task cooperative调度模式的runtime。

与之对比，Rust的async/await设计只包括了基于traits的接口设计，并不包括runtime实现，完整的async/await必须包括外围生态提供的runtime。这样做的好处是，可以针对应用的特点实现不同的runtime调度机制，例如：基于I/O bound和CPU bound任务可以使用的不同的runtime，runtime可以实现基于cooperative task的调度机制，也可以实现基于preemptive的调度。缺点是，可能会出现多个竞争性的生态，用户选择太多导致无所适从。目前已经出现的async/await runtime就有6-7种
tokio，async-std，smol，monoio，glommio，lunatic






Rust提供的基于软件task的异步并发特性被用于domain specific的library开发。例如：tokio，lunatic，


* 消息传递和数据隔离：级所谓的actor模式。

Rust内置支持msg channel概念，例如MPSC channel。生态中，Lunatic runtime试图对标goroutine实现完整的actor模式和preemtive任务调度系统。Rust最初的版本包括preemptive调度的green task特性，非常类似Microsoft的用户态任务调度体系Fiber，后续版本删除了这个特性，因为希望由生态runtime Library来实现，而不需要在语言层实现。

* 分布式数据和计算

Rust的Ryaon并行计算库。

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
    * thread
    * I/O pipe
    * IO_URING
2.  OS分配/pin给Runtime thread和memory资源，一次分配
    * 动态分配，按需伸缩
    * Runtime和其他OS App并存，Runtime是个Super App
    * Virtual memory
4.  Runtime 接管大部分OS功能，二次分配
    * memory management, single address space, zero-overhead memory isolation, pre-allocated in the compiler time, dynamically linked
    * task management, cooperative green task, N:M mapping, work stealing
    * task vs thread, much smaller memory size, no context switch overhead
    * device drivers, user space drivers, isolations
    * capability based security, application kernel to screen the I/O system call
5. Runtime支持domain specific runtime
    * I/O bound - tokio
    * CPU bound - rayon
    * 前端UI - yew
6. 应用开发
    * App由多个task构成，一个task由1个或者多个cell构成，每个cell对应一个crate，crate编译为cell
    * cell可以单独升级，动态swap in swap out，下载package少，可以continuous upgrade
    * App based 调度策略，
    * blueprint based deployment
7. 挑战
    * tokio这样的runtime需要改造为multi-tenant，多个app同时调用tokio library，但是tokio的task scheduler需要为多个app调度任务，目前不支持multi-tenancy，因为是libary
    * 需要替换runtime的task scheduler，由theseus runtime统一管理task调度？可行性？因为TR接管了linker，可以让不同的app共享library？library必须是stateless的
    * TR需要设计新的scheduler，由不同的调度策略需要实现，重用tokio的task scheduler设计，前台任务和后台任务的调度，QoS优先级
    * App之间的API通信机制，基于library？

## 基于Rust的实现的App开发框架

### 基于Rust的WGPU新的GUI library

提供WebGPU标准API， WebGPU是W3C标准，致力基于最新的底层Graphic API，例如：Vulcan，Metal，D3D12 提供支持

https://blog.devgenius.io/will-webgpu-be-the-webgl-killer-60a49509b806

https://surma.dev/things/webgpu/


* Reduce CPU overheads
* Good support of multi-threading
* Bringing the Power of General-Purpose Computing (GPGPU) to the Web with Compute Shaders
* Brand new shader language — WebGPU Shading Language (WGSL)
* Technology that will support “Real-time ray tracing” in the future

WGPU实现的




![](https://i.imgur.com/3ARyyxj.png)

### App 开发框架

https://blog.logrocket.com/current-state-rust-web-frameworks/

前端UI框架也是async使用的主要场景，虽然这个领域因为Rust学习曲线较为陡峭，生态发展相对较慢，但是也出现了比较有特色的UIKit和App开发框架，这些框架也有鲜明的Rust生态的模块化可组合的鲜明特点，而且因为Rust是系统编程语言，这些框架也可以实现跨平台App开发。第一种思路是利用web生态，通过webview来渲染基于vue，react这样的前端框架产生的页面，tauri提供了对平台webview和window管理的抽象，提供了前端webview和后台事件业务处理直接的消息桥梁，后端可以和yew或者deno这样的异步应用框架对接，一个web app可以较好的映射到tauri的应用框架。Tauri相比elctron这样的跨平台App框架的优势在于，Tauri利用了原生平台的webview能力，后端也可以plugin不同的async应用开发框架，所以无论在尺寸和扩展性上都比Electron这种自带chromium webview和nodejs的方案要好。


![](https://i.imgur.com/u8flvKB.png)

Servo是Mozilla使用Rust开发的浏览器内核，也是当初创造Rust语言的第一个需求，Servo内部使用了Rust的内存安全和并发安全机制，实现了异步并行的CSS/HTML解析和渲染管线，而且起模块化的设计非常适合作为嵌入式webview引擎。Tauri也计划和Servo做集成，这样Tauri可以选择和Servo一起分发，解决使用平台webview带来的渲染效果不一致的问题。

Makepad是一款Rust语言开发的模块化和高性能的UIKit，和React JSX或者SWiftUI这样的前端DSL一样，Makepad也有Makepad UI DSL，可以描述UI组件和页面布局，不同的地方是Makepad的DSL充分兼容了figma这样的前端设计工具的语法，这样很容易把设计师通过设计工具产生的UI设计转化为Makepad UI DSL，南向，Makepad的DSL可以动态被编译为GPU的shader language，通过GPU实时渲染出页面，这样可以直接通过修改和更新Makepad UI DSL实现对UI的实时修改。通过直接调用GPU的shader能力做渲染，可以同时支持2D和3D图形渲染，具有很好的前瞻性。Makepad这种直接通过GPU渲染的方式称为Direct Mode，它的优点是渲染任务完全卸载给GPU，不占用CPU资源，传统2D图形库充分利用了2D UI具有很多局部变化的特点，对渲染页面做了内存优化，以大幅度减少需要渲染的工作量，这种方式叫做retained mode，Makepad实现中也借鉴了retained mode，减少过多使用GPU带来的功耗问题，

![](https://i.imgur.com/vdc20Zo.png)

## Mobile Rust 需要投入的方向

1. ### 基于ABI实现dynamic linking
2. ### structued async/await
 
