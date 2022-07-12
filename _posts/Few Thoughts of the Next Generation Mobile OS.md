# 下一代移动操作系统的构想

## 主要需求
1. 兼容既有生态
2. 计算效率 - 内存、CPU制程、功耗
3. 编程语言
4. 安全
5. 开发者生态
    * 生态 - library
    * 开发者体验 - learning curve
## 硬件技术趋势

1. 多核硬件
2. 异构计算

## 软件技术趋势

1. 并行并发
2. domain specific runtime
3. exokernel
4. Rust zero-overhead abstraction

## 编程语言
1. traditional control flow
2. asynchronous control flow
3. message passing and data isolation
4. distributed data and compute

## OS

1. async I/O
2. user space drivers
3. capability based security
4. devops - continuous upgrade


## IDEA

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
6. Apps
    * App由多个task构成，一个task由1个或者多个cell构成，每个cell对应一个crate，crate编译为cell
    * cell可以单独升级，动态swap in swap out，下载package少，可以continuous upgrade
    * App based 调度策略，
    * blueprint based deployment
7. Challenges
    * tokio这样的runtime需要改造为multi-tenant，多个app同时调用tokio library，但是tokio的task scheduler需要为多个app调度任务，目前不支持multi-tenancy，因为是libary
    * 需要替换runtime的task scheduler，由theseus runtime统一管理task调度？可行性？因为TR接管了linker，可以让不同的app共享library？library必须是stateless的
    * TR需要设计新的scheduler，由不同的调度策略需要实现，重用tokio的task scheduler设计，前台任务和后台任务的调度，QoS优先级
    * App之间的API通信机制，基于library？