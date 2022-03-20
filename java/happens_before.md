# Happens-before Order

一直对happens-before的理解有些模糊，看了网上的一些解读也是似懂非懂。怎么办？直接看看The Java® Language Specification中是如何解释的，准确、权威。

### 17.4.5. Happens-before Order

Two actions can be ordered by a happens-before relationship. If one action happens-before another, then the first is visible to and ordered before the second.

If we have two actions x and y, we write *hb(x, y)* to indicate that x happens-before y.

- If x and y are actions of the same thread and x comes before y in program order, then *hb(x, y)*.

- There is a happens-before edge from the end of a constructor of an object to the start of a finalizer (§12.6) for that object.

- If an action x synchronizes-with a following action y, then we also have *hb(x, y)*.

- If *hb(x, y)* and *hb(y, z)*, then *hb(x, z)*.

The wait methods of class Object (§17.2.1) have lock and unlock actions associated with them; their happens-before relationships are defined by these associated actions.

It should be noted that the presence of a happens-before relationship between two actions does not necessarily imply that they have to take place in that order in an implementation. If the reordering produces results consistent with a legal execution, it is not illegal.  
For example, the write of a default value to every field of an object constructed by a thread need not happen before the beginning of that thread, as long as no read ever observes that fact.  

More specifically, if two actions share a happens-before relationship, they do not necessarily have to appear to have happened in that order to any code with which they do not share a happens-before relationship. Writes in one thread that are in a data race with reads in another thread may, for example, appear to occur out of order to those reads.  

The happens-before relation defines when data races take place.  

A set of synchronization edges, S, is sufficient if it is the minimal set such that the transitive closure of S with the program order determines all of the happens-before edges in the execution. This set is unique.  

It follows from the above definitions that:
- An unlock on a monitor *happens-before* every subsequent lock on that monitor.
- A write to a *volatile* field (§8.3.1.4) *happens-before* every subsequent read of that field.
- A call to *start()* on a thread *happens-before* any actions in the started thread.
- All actions in a thread *happen-before* any other thread successfully returns from a *join()* on that thread.
- The default initialization of any object *happens-before* any other actions (other than default-writes) of a program.
