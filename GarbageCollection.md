## Garbage Collection

### Introduction

**Memory manager** keeps track of free space and respond to malloc and free operations on the program. 

To be efficient the allocation algorithm has to take into cosideration:
* **Space efficiency:** don't leave to many small holes that can't be used 
* **Time efficient:** dominated by allocation of small chunks
* **Spatial Locality:** Best if all space is coalesced

**Algorithms**
* Best fit: best utilization of space
* Next fit: Improves locality because consecutively allocated data tend to have similar lifespan

### What is garbage?

We'd like to reclaim everything that is **dead**. But analysing that is hard, so we just reclaim everything that is 
**unreachable**.

**Reachable:**
* Things in the stack frames
* Global variables
* Registers
* All objects referenced in one of the aboves and their references too.

| Operation                                           | Effect                    |
| ----------                                          | ------------------------- |
| New/malloc                                          | Creates objects           |
| Store p in a pointer variable or field in an object | p ← o<sub>1</sub>, o<sub>1</sub> is now reachable throug p (and all objects referenced from o<sub>1</sub>). <br> If p was used to point to o<sub>2</sub> now that may be unreachable. |
| Load                                                | Add pointer to reachable object |
| Procedure calls                                     | **entry:** add pointers to all incoming parameters (new pointers in the stackframe). <br> **exit:** Remove the pointers in the stackframe. We add a new pointer for the return value |

**Once an object is unreachable, it stays unreachable for ever and ever!**

### Reference counting 

Free objects as they transition from reachable to unreachable **keeping count of pointers to each object**.

When refrence count of an object = 0:
* delete object
* subtract reference counts of objects it points to
* recurse if needed

Not reachable ⇏ zero reference: Think in the case of a cyclic data structure that is no longer reachable.

**Cost**: It's good that it reclaims the space as soon as the object is not reachable. But it adds an overhead for each
statement that changes the ref. count

### Baker's algorithm 

Assumes everything is not reachable unless proven otherwise. It's a stop-the-world type of GC

![Baker's algorithm](/images/baker.png)

### Frequency of GC

Cost of reachability analysis: **depends on reachable objects**.
Less frequent: **faster overall** but requires more memory.

Performance metric

|                             | Reference counting | Trace Based |
| --------------------------- | ------------------ | ----------- |
| Space reclaimed             |                    | :white_check_mark: |
| Overall execution time      |                    | :white_check_mark: |
| Space usage                 | :white_check_mark: |             |
| Pause time                  | :white_check_mark: |             |
| Program locality            |                    | **opportunity<sup>\*</sup>** |

<sup>\*</sup> we can repack data because we calculate everything that is reachable

### Copying collector

Memory devided in 2 semi-spaces. When the semi-space is nearly full we invoke GC which copies reachable objects to the other space re-packing them. Note that when copying we also have to change all the pointer addresses.

Same as baker algorithm but changes set representation. Now we use 3 pointers: scanned, unscanned and free.

![copying collector](/images/copyingCollector.png)

### Incremental GC

Interleaves GC with mutator action to reduce pause time

![incremental GC](/images/incrementalGC.png)

May misclassify some **unreachable as reachable** (this is safe)

```
Ideal = (R ∪ New) - Lost ⊆ Anser ⊆ (R ∪ New)
```

**To resume GC **
* Find root sets
* Place newly reached objects in "unnscanned list"
* Continue to trace reachability without redoing scanned objects

** But this doesn't find all the reachable objects!** 

![missed reachable objects](/images/missedReachableObjects.png)

**Solution** Intercept p in any of the 3 steps.
* Read barrier: Remember all loads of pointers from B → C
* Write barrier: Remember all stores of pointers from A → C
* Overwrite barrier: Remember all orverwrites from B → C

**The most efficient is the write barrier.** Because writes are less frequent than reads. And it includes less unreachable objects than over-write barriers

### Generational GC

Exploits the property that **most objects die young.**

```
Separate heaps in partitions, numbering them from 0 to n with. 
Lower numbered partitions hold younger objects. 
We allocate new objects on partiton 0.

GC
1. When partition 0 fills up we run GC and the surviving objects are moved into partition 1
2. Repeat step 1 until partition 1 fills up.
3. GC 0 and 1 (and move surviving objects from 0 to 1 and from 1 to 2).
4. Keep going...
```

The root set for a partial collection invoked on set i includes the remembered sets for partition i and below. That is a list of objects from partitions above i that points to objects in i.

**May misclassify unreachable as reachable:** When pointers in earlier generations are overwritten

It's effective because time is spent on objects that are mostly garbage (young objects). Eventually a full GC is performed.
