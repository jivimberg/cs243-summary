## Dynamic Compilation

Is a combination between compiled code and interpreted code. We use the runtime information to compile to binary. This way you can optimize the program based on the input.

**Binary translator:** binary to binary. Mostly dinamic. Used for software migration (running binary code from a machine into another)


#### Closed world Vs. Open world

* **Closed world assumption (most static compilers)**: All code is available a priori for analysis and compilation
* **Open-world assumption (most dynamic compilers)**: Code is not available; arbitrary code can be loaded at runtime. This precludes many optimization opportunities. To solve this you can optmistically assume the best case but provide a way out if necessary.

### Speculative Inlining

**Virtual call sites are deadly!** Kill optimization opportunities, because you don't know which code you're calling. A good solution is to speculatively inline the most likely call target based on the class hierarchy or profile information. Many virtual call sites have only one target so this works pretty well. 

### Compilation policy 

`ΔT<sub>total</sub> = T<sub>compile</sub> - (n<sub>executions</sub> * T<sub>improvement</sub>)`

If ΔT<sub>total</sub> is negative the policy is effective

#### Latency vs. Throughput 

![Latency vs. Throughput](/images/latencyVsThroughput.png)

We need to use a little bit of each. You start interpreting the code. If it's executed many times you compile it. If it's being executed many many times then you optimize it.

#### Granularity of compilation

We try to optimize hot regions (not methods!). That is only the most frequently executed segments within a method. This is called **partial method optimization**. Also you can optimize that region even if other paths get penalized

### Dynamic code transformations

1. Compiling partial methods, compiling only hot blocks
2. Partial dead code elimination
3. Escape analysis

#### Partial methods compilation

1. Use code coverage information from the first compiled version to identify hot regions.
2. Perform **live variable analysis** to identify de live variables at rare block entry point.
3. Redirect the control flow edges that targeted rare blocks and remove the rare blocks.
4. Perform compilation normally and treat the rare block as something you can't analyse. You just keep the variables that need to be live at the beggining of the rare block (meaning you can't eliminate them as part of other optimization)
5. Record a map for each interpreter transfer point. Generate a map that specifies the location, in registers or memory, of each of the live variables.

![Partial Method Compilation](/images/partialMethodCompilation.png)

#### Partial dead code elimination

Move computation that is only live on a rare path into the rare block, thus saving computation time in the common case. For example is a variable is only used inside a rare if block then move variable initialization to such block.

#### Escape analysis

Finds objects that do not escape a method or a thread.
  * "Captured" by method: can be allocated on stack or in registers (instead of allocating it in the heap, you save time on GC)
  * "Captured" by thread can avoid synchronization operations.
  * If you mix this with partial method compilation it can get complicated. You know have to copy stack-allocated objects to the heap and upated pointers
  

