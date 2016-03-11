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
