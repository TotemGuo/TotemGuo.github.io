# 前面
记得初学Java，对于Java中的线程和锁的相关内容理解的不够透彻，对涉及到并发的相关代码缺乏判断力，并不真切的知道代码背后的含义。市面上的一些书籍（各大技术网站推荐的某并发编程的艺术）并不是说不正确，但总感觉介绍的很晦涩。后来看到Java语言规范的原版，才发现很多书籍和网上文章就是在翻译规范，而且是选择性翻译，造成初学者对此一知半解，似懂非懂。**一知半解的知识，还不如不懂！**这里贴一下规范原版，并写一些自己的注解，欢迎交流。
# Chapter 17. Threads and Locks
While most of the discussion in the preceding chapters is concerned only with the behavior of code as executed a single statement or expression at a time, that is, by a single thread, the Java Virtual Machine can support many threads of execution at once. These threads independently execute code that operates on values and objects residing in a shared main memory. Threads may be supported by having many hardware processors, by time-slicing a single hardware processor, or by time-slicing many hardware processors.

Threads are represented by the Thread class. The only way for a user to create a thread is to create an object of this class; each thread is associated with such an object. A thread will start when the start() method is invoked on the corresponding Thread object.

The behavior of threads, particularly when not correctly synchronized, can be confusing and counterintuitive. This chapter describes the semantics of multithreaded programs; it includes rules for which values may be seen by a read of shared memory that is updated by multiple threads. As the specification is similar to the memory models for different hardware architectures, these semantics are known as the Java programming language memory model. When no confusion can arise, we will simply refer to these rules as "the memory model".

These semantics do not prescribe how a multithreaded program should be executed. Rather, they describe the behaviors that multithreaded programs are allowed to exhibit. Any execution strategy that generates only allowed behaviors is an acceptable execution strategy.

译     
前面章节的大多数讨论只关注同一时刻只有一个语句或表达式执行时（也就是单线程场景下）代码的行为，而实际上，Java虚拟机支持很多线程同时执行。这些线程独立执行代码，而这些代码是对位于共享主内存中的值和对象进行操作的。对（多）线程的支持有多种方式，比如使用多硬件处理器，对单硬件处理器进行时间分片以及对多硬件处理器进行时间分片。  

线程是通过Thread类来表示的。用户创建线程的唯一方式是创建该类的一个对象；每一个线程与这样一个对象对应（关联）。当对应Thread对象上的start()方法被触发时，线程就开启了。    

如果没有经过正确的同步，线程的行为会令人感到困惑，甚至违反直觉。本章节描述多线程程序的语义；它包含了（变量）值可能被共享内存的读取操作可见的一些规则，这些共享内存是被多线程更新的。由于本篇说明跟针对不同硬件架构的内存模型很相似，这些语义又被成为Java编程语言内存模型。在没有歧义的情况下，我们称这些语义为内存模型。

这些语义并没有规定一个多线程程序应该怎样被执行。相反的，它们描述了多线程程序被允许呈现出的一些行为。只生成被允许的行为的任何执行策略都是一个可接受的执行策略。

## 17.1 Synchronization
The Java programming language provides multiple mechanisms for communicating between threads. The most basic of these methods is synchronization, which is implemented using monitors. Each object in Java is associated with a monitor, which a thread can lock or unlock. Only one thread at a time may hold a lock on a monitor. Any other threads attempting to lock that monitor are blocked until they can obtain a lock on that monitor. A thread t may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation.

The synchronized statement (§14.19) computes a reference to an object; it then attempts to perform a lock action on that object's monitor and does not proceed further until the lock action has successfully completed. After the lock action has been performed, the body of the synchronized statement is executed. If execution of the body is ever completed, either normally or abruptly, an unlock action is automatically performed on that same monitor.

A synchronized method (§8.4.3.6) automatically performs a lock action when it is invoked; its body is not executed until the lock action has successfully completed. If the method is an instance method, it locks the monitor associated with the instance for which it was invoked (that is, the object that will be known as this during execution of the body of the method). If the method is static, it locks the monitor associated with the Class object that represents the class in which the method is defined. If execution of the method's body is ever completed, either normally or abruptly, an unlock action is automatically performed on that same monitor.

The Java programming language neither prevents nor requires detection of deadlock conditions. Programs where threads hold (directly or indirectly) locks on multiple objects should use conventional techniques for deadlock avoidance, creating higher-level locking primitives that do not deadlock, if necessary.

Other mechanisms, such as reads and writes of volatile variables and the use of classes in the java.util.concurrent package, provide alternative ways of synchronization.

## 17.2  Wait Sets and Notification
Every object, in addition to having an associated monitor, has an associated wait set. A wait set is a set of threads.

When an object is first created, its wait set is empty. Elementary actions that add threads to and remove threads from wait sets are atomic. Wait sets are manipulated solely through the methods Object.wait, Object.notify, and Object.notifyAll.

Wait set manipulations can also be affected by the interruption status of a thread, and by the Thread class's methods dealing with interruption. Additionally, the Thread class's methods for sleeping and joining other threads have properties derived from those of wait and notification actions.

## 17.3 Sleep and Yield
Thread.sleep causes the currently executing thread to sleep (temporarily cease execution) for the specified duration, subject to the precision and accuracy of system timers and schedulers. The thread does not lose ownership of any monitors, and resumption of execution will depend on scheduling and the availability of processors on which to execute the thread.

It is important to note that neither Thread.sleep nor Thread.yield have any synchronization semantics. In particular, the compiler does not have to flush writes cached in registers out to shared memory before a call to Thread.sleep or Thread.yield, nor does the compiler have to reload values cached in registers after a call to Thread.sleep or Thread.yield.

> For example, in the following (broken) code fragment, assume that this.done is a non-volatile boolean field:
>     while (!this.done)
>       Thread.sleep(1000);
> The compiler is free to read the field this.done just once, and reuse the cached value in each execution of the loop. This would mean that the loop would never terminate, even if another thread changed the value of this.done.

## 17.4 Memory Model
A memory model describes, given a program and an execution trace of that program, whether the execution trace
is a legal execution of the program. The Java programming language memory model works by examining each read
in an execution trace and checking that the write observed by that read is valid according to certain rules.  
内存模型会检查每一个执行轨迹中的每一个读动作，并根据确定的规则检查这个读动作观察到的写动作是合理的。  
The memory model describes possible behaviors of a program. An implementation is free to produce any code it
likes, as long as all resulting executions of a program produce a result that can be predicted by the memory
model.  
This provides a great deal of freedom for the implementor to perform a myriad of code transformations, including
the reordering of actions and removal of unnecessary synchronization.  
The memory model determines what values can be read at every point in the program. The actions of each thread
in isolation must behave as governed by the semantics of that thread, with the exception that the values seen
by each read are determined by the memory model. When we refer to this, we say that the program obeys intra-thread
semantics. Intra-thread semantics are the semantics for single-threaded programs, and allow the complete prediction
of the behavior of a thread based on the values seen by read actions within the thread. To determine if the
actions of thread t in an execution are legal, we simply evaluate the implementation of thread t as it would
be performed in a single-threaded context, as defined in the rest of this specification.  
内存模型决定了在程序的每一个点上什么样的数值可以被读取。对于每一个处于相互隔离的线程，该线程的动作集合必须表现的就像被线程语义
管理一样, 除非内存模型能决定每一个读操作看到的值。当我们讨论这个的时候，我们是指程序遵循"线程内语义"。线程内语义是为单线程程序
准备的语义，并且它允许基于线程内的读动作看到的值进行线程行为的完全预测。（关于线程内语义的理解参考：https://stackoverflow.com/questions/25711048/understanding-intra-thread-semantics）
为了决定一个execution中线程t的动作集合是否合法，我们会简单的评估线程t的实现，就像它在单线程上下文中执行，正如这篇规范剩余部分
定义的那样。  

### 17.4.1 Shared Variables  

......  
  
Two accesses to (reads of or writes to) the same variable are said to be conflicting if at least one of the accesses is a write.

### 17.4.3 Programs and Program Order
Among all the inter-thread actions performed by each thread t, the program order is a total order that reflects
the order in which these actions would be performed according to the intra-thread semantics of t.  
A set of actions is sequentially consistent if all actions occur in a total order (the execution order) that is
consistent with program order, and further more, each read r of a variable v sees the value written by the write
to v such that:  
- w comes before r in the execution order, and
- there is no other write w' such that w comes before w' and w' comes before r in the execution order.  

### 17.4.5 Happens-before Order
When a program contains two conflicting accesses (§17.4.1) that are not ordered by a happens-before relationship, it is said to contain a data race.  
  
......  
  
A program is correctly synchronized if and only if all sequentially consistent executions are free of data races.  
If a program is correctly synchronized, then all executions of the program will appear to be sequentially consistent.  
  
......  
  
A set of actions A is happens-before consistent if for all reads r in A, where W(r) is the write action seen by
r, it is not the case that either hb(r, W(r)) or there exist a write w in A such that w.v=r.v and hb(W(r), w)
and hb(w, r).  
