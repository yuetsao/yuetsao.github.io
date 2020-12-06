# MULTI_THREAD
## 基本概念
* 进程
<br/>异常是允许操作系统内核提供进程（process）概念的基本构造块，进程是计算机科学中最深刻、最成功的概念之一。
<br/>在现代系统上运行一个程序时，我们会得到一个假象，就好像我们的程序是系统中当前运行的唯一的程序一样。我们的程序
好像是独占地使用处理器和内存。处理器就好像是无间断地一条接一条地指向我们程序中的指令。最后，我们程序中的代码和数据
好像是系统内存中的唯一对象。这些假象都是通过继承的概念提供给我们的。
<br/>进程的经典定义就是一个***执行中程序的实例***。系统中的每个程序都运行在某个进程的***上下文(context***)中。上下文
是由程序正确运行所需要的状态组成的。这个状态包括存放在内存中的程序的代码和数据，它的栈、通用目的寄存器的内容、程序
计数器、环境变量以及打开文件描述符的集合。
<br/>每次用户通过向shell输入一个可执行目标文件的名字，运行程序时，shell就会创建一个新的进程，然后在这个新进程的上下文中运行这个可
执行目标文件。应用程序也能够创建新进程，并且在这个新进程的上下文运行它们自己的代码或其他应用程序。
<br/>关键抽象
<br/> * 一个独立的逻辑控制流，它提供一个假象，好像我们的程序独占地使用处理器。
<br/> * 一个私有的地址空间，它提供一个假象，好像我们的程序独占地使用内存系统。
* 线程
<br/>作为进程里面最小的执行单元叫做线程，用简单的话来说一个程序例不同的执行路径。
每个线程都运行在进程的上下文中，并共享同样的代码和全局数据。由于网络服务器中对并行处理的要求，线程称为越来越重要的编程
模型，因为多线程之间比多进程之间更容易共享数据，也因为线程一般来说都比进程要更高效。当有多处理器可用的时候，多线程也是一种使
得程序可以运行得更快的方法。
* 纤程
<br/>线程的英文Fibers
<br/>A Fiber is a lightweight thread that uses cooperative multitasking instead of preemptive multitasking. A running fiber must explicitly "yield" to allow another fiber to run, which makes their implementation much easier than kernel or user threads.
<br/>A Coroutine is a component that generalizes a subroutine to allow multiple entry points for suspending and resuming execution at certain locations. Unlike subroutines, coroutines can exit by calling other coroutines, which may later return to the point where they were invoked in the original coroutine.
<br/>A Green Thread is a thread that is scheduled by a virtual machine (VM) instead of natively by the underlying operating system. Green threads emulate multithreaded environments without relying on any native OS capabilities, and they are managed in user space instead of kernel space, enabling them to work in environments that do not have native thread support.
<br/>

## 参考

<div id="refer-anchor-1"></div>

- [1] [深入理解计算机系统]()

<div id="refer-anchor-2"></div>

- [2] [马士兵多线程与高并发]()

<div id="refer-anchor-2"></div>

- [2] [stackoverflow](https://softwareengineering.stackexchange.com/questions/254140/is-there-a-difference-between-fibers-coroutines-and-green-threads-and-if-that-i)
