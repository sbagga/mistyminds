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


移动OS的程序控制类型
1. 传统的同步串行控制流程
    * 大部分程序员熟悉的编程模式
    * 难以利用并发并行硬件
2. 异步并发控制流程
    * 异步编程方式破坏了程序的执行顺序，难以编写和调试，用户体验差
    * Async/Await 成为业界共识，可以以串行方式写并发应用，主流语言均已支持（Swift，JS，Python，Kotlin，Rust），成为新的异步并发编程范式
    * 主要用于解决事件触发机制的并发，包括：I/O访问，数据库访问，UI前端开发，2D并行图形渲染，游戏脚本开发
3. 消息传递和数据隔离
    * 通过消息机制来实现数据隔离，避免数据并发访问，所谓的actor模式，是实现并发编程的另一种主流模式
    * 发端于erlang语言，Go的最大特色就是实现了基于Actor的goroutine，取得了很好的效果，Swift也全面支持Actor，Rust通过channel机制支持
    * 主要解决带有共享状态的场景，例如：前端UI开发
4. 分布式数据和计算
    * 主要是数据密集型的SIMD计算模式，解决数据切分，计算切分问题
    * 使用场景：音视频媒体处理、GPU图像处理、AI/ML，物理引擎等



## Mobile Rust构想

> 
> C++ implementations obey the zero-overhead principle: What you don't use, you don't pay for [Stroustrup, 1994]. And further: What you do use, you couldn't hand code any better.
> 
> -- Stroustrup
> 

Rust 语言带来并行并发系统开发的弯道超车的机会
1. zero-overhead abstraction，尽可能在编译时间优化，而不要付出运行时的代价
* 内存ownership的机制实现内存不可能越界访问，因此运行时不需要做内存越界检查（C/C++没有这个特性，WASM必须付出运行时的代价）
* 内置并行并发特性，共享内存的场景使用严格受控，避免无必要的内存锁设计造成的性能损失
* 基于future实现的Async/Await被编译器静态展开为有限状态机，避免运行时动态申请内存的开销
* 语言核心特性严格管理，通过traits扩展功能（inheritance vs composition，C++ OOP失败的设计），形成良好内核抽象稳定、library生态活跃创新的格局，Chris Lattner特别强调的small things that compose的格局，避免了微软 fiber，Apple GCD等中心化创新的困境（无法通过大规模试验获取经验，迭代创新），出现了tokio这样的活跃library生态
2. Rust进入操作系统
* Rust正式成为Linux Kernel的开发语言，关键的kernel library会重写，包括ISRG Prossimo(https://www.memorysafety.org/about/)项目赞助的使用Rust重写Linux Kernel和关键library（TLS，NTP等），OpenSSF提出的用安全语言（Rust，Go）重写C/C++b编写的基础软件的项目，是增大kernel话语权的机会。
* Rust提供的基于软件task的异步并发特性被用于domain specific的library开发。例如：tokio，lunatic，这些library runtime提供了用户态的基于软件task的调度和管理能力，比OS的线程调度机制内存占用少、任务调度代价低，例如：OS的支持的thread只有几千个，而runtime可以调度的软件task可以达到10万以上，在domain问题领域能更为高效的利用硬件资源，
* 基于Rust开发的新型OS，例如：theseus OS利用Rust zero-overhead abstraction带来的内存隔离，摆脱了传统OS通过kernel和用户态的内存隔离，用户态进程之间的虚拟内存隔离带来的开销，实现了single address space的内存使用机制
* 基于Rust开发的应用I/O Kernel，例如：quark，配合Linux Kernel的新异步I/O机制IO_URING实现了高性能的I/O虚拟化

![](https://i.imgur.com/vfOq1j6.png)


![](https://i.imgur.com/D46NvXT.png)


综上所述，Rust zero-overhead abstraction 的设计理念强力支持了OS开发需要的安全、可靠、高效、高性能等需求，实现了对C/C++的跨代式超越，无论在Linux还是新型OS领域都会获得大量使用，是改变既有OS格局的重要武器。Rust提供的内存ownership模式，改变了操作系统传统基于硬件或者runtime实现的资源隔离，同时提供安全可靠和有优秀用户体验的异步并发软件开发能力，可以实现从OS到应用层的全栈异步并行开发，已经证明可以更加有效利用多核处理器，为颠覆性的OS创新提供了沃土。


## 基于Rust的实现的Super Runtime

### 需求

1. 对既有mobile OS生态的支持，完全不继承既有生态和代码资产不现实
2. 又需要实现全栈式（编程语言、OS、应用框架、开发者体验）的颠覆式创新，才能配合新的面积换性能硬件架构

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

前端UI框架也是async使用的主要场景，虽然这个领域因为Rust学习曲线较为陡峭，生态发展相对较慢，但是也出现了比较有特色的UIKit和App开发框架，这些框架也有鲜明的Rust生态的模块化可组合的鲜明特点，而且因为Rust是系统编程语言，这些框架也可以实现跨平台App开发。第一种思路是利用web生态，通过webview来渲染基于vue，react这样的前端框架产生的页面，tauri提供了对平台webview和window管理的抽象，提供了前端webview和后台事件业务处理直接的消息桥梁，后端可以和yew或者deno这样的异步应用框架对接，一个web app可以较好的映射到tauri的应用框架。Tauri相比elctron这样的跨平台App框架的优势在于，Tauri利用了原生平台的webview能力，后端也可以plugin不同的async应用开发框架，所以无论在尺寸和扩展性上都比Electron这种自带chromium webview和nodejs的方案要好。


![](https://i.imgur.com/u8flvKB.png)

Servo是Mozilla使用Rust开发的浏览器内核，也是当初创造Rust语言的第一个需求，Servo内部使用了Rust的内存安全和并发安全机制，实现了异步并行的CSS/HTML解析和渲染管线，而且起模块化的设计非常适合作为嵌入式webview引擎。Tauri也计划和Servo做集成，这样Tauri可以选择和Servo一起分发，解决使用平台webview带来的渲染效果不一致的问题。

Makepad是一款Rust语言开发的模块化和高性能的UIKit，和React JSX或者SWiftUI这样的前端DSL一样，Makepad也有Makepad UI DSL，可以描述UI组件和页面布局，不同的地方是Makepad的DSL充分兼容了figma这样的前端设计工具的语法，这样很容易把设计师通过设计工具产生的UI设计转化为Makepad UI DSL，南向，Makepad的DSL可以动态被编译为GPU的shader language，通过GPU实时渲染出页面，这样可以直接通过修改和更新Makepad UI DSL实现对UI的实时修改。通过直接调用GPU的shader能力做渲染，可以同时支持2D和3D图形渲染，具有很好的前瞻性。Makepad这种直接通过GPU渲染的方式称为Direct Mode，它的优点是渲染任务完全卸载给GPU，不占用CPU资源，传统2D图形库充分利用了2D UI具有很多局部变化的特点，对渲染页面做了内存优化，以大幅度减少需要渲染的工作量，这种方式叫做retained mode，Makepad实现中也借鉴了retained mode，减少过多使用GPU带来的功耗问题，

![](https://i.imgur.com/vdc20Zo.png)

