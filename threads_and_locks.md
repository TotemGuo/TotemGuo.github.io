# Threads and Locks
记得初学Java，对于Java中的线程和锁的相关内容理解的不够透彻，对涉及到并发的相关代码缺乏判断力，并不真切的知道代码背后的含义。市面上的一些书籍（各大技术网站推荐的某并发编程的艺术）并不是说不正确，但总感觉介绍的很晦涩。后来看到Java语言规范的原版，才发现很多书籍和网上文章就是在翻译规范，而且是选择性翻译，造成初学者对此一知半解，似懂非懂。**一知半解的知识，还不如不懂！**这里贴一下规范原版，并写一些自己的注解，欢迎交流。

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

### 17.4.3 Programs and Program Order
Among all the inter-thread actions performed by each thread t, the program order is a total order that reflects
the order in which these actions would be performed according to the intra-thread semantics of t.
A set of actions is sequentially consistent if all actions occur in a total order (the execution order) that is
consistent with program order, and further more, each read r of a variable v sees the value written by the write
to v such that:
.
.
### 17.4.5 Happens-before Order
......
A program is correctly synchronized if and only if all sequentially consistent executions are free of data races.
If a program is correctly synchronized, then all executions of the program will appear to be sequentially consistent.
......
A set of actions A is happens-before consistent if for all reads r in A, where W(r) is the write action seen by
r, it is not the case that either hb(r, W(r)) or there exist a write w in A such that w.v=r.v and hb(W(r), w)
and hb(w, r).
