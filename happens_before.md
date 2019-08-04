# Happens-before Order

一直对happens-before的理解有些模糊，看了网上的一些解读也是似懂非懂。怎么办？直接看看The Java® Language Specification中是如何解释的，准确、权威。

### 17.4.5. Happens-before Order

Two actions can be ordered by a happens-before relationship. If one action happens-before another, then the first is visible to and ordered before the second.

If we have two actions x and y, we write *hb(x, y)* to indicate that x happens-before y.

- If x and y are actions of the same thread and x comes before y in program order, then *hb(x, y)*.

- There is a happens-before edge from the end of a constructor of an object to the start of a finalizer (§12.6) for that object.

- If an action x synchronizes-with a following action y, then we also have *hb(x, y)*.

- If *hb(x, y)* and *hb(y, z)*, then *hb(x, z)*.
