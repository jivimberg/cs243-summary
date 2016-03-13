##Parallelization

This is multiprocess parallelization. Not the instruction level parallelism from before.

### DoAll loops

```
for (i = 11 to 20) 
  a[i] = a[i-10} + 3
```
This is a do all too! because you're reading from 1 to 10 and writing from 11 to 20, no dependence.

Does there exists a data dependence edge beteween 2 different iterations? 
* A data dependence edge is _loop-carried_ if it crosses iterations boundaries
* DoAll Loops = loops without loop-carried dependences

Recall data dependences from [Instruction Scheduling](InstructionScheduling.md)

### Affine Array Accesses

When array indexes are affine expressions of surrounding loop indexes. Meaning they are linear expressions of indexes + constants. Example A[2*i+1].

This is the domain we'll focus on. If we introduce `read(n)` for example, we have no way of knowing what _n_ is at runtime.

In fact we only look at indexes with affine relations and ignore the other ones (for example if I have something like: A[i*i]). We ignore them and we assume they may refer to the same location.

When analizing a multi dimensional index all the indexes have to match for it to be a dependence. Therefore if you need to ignore one of the indexes you can still make the safe assumption that it matches and work with the others to see if the others don't.

### Data dependences analysis

Be sure to also check for output dependence! In such cases you add the constrait that i ≠ i'

![Dependence Analysis Example](/images/depAnalysisExample.png)

![Dependence Analysis](/images/depAnalysis.png)

:pencil2: //TODO understand the GCD test for data dependence

### Fourier-Motzkin Elimination

Eliminate one variable at a time to see if there's a solution at the end. 

```
To eliminate x<sub>1</sub>
  1. Write all inequalities as Lower bounds and Upper bounds of x<sub>1</sub>
  2. Cross product all Upper and Lower bounds of x<sub>1</sub>
```

**If one of the variables can only be a real number**. For example you discover that x=0,5. Then you create 2 problems by adding x ≤ 0 and x ≥ 1 and do the test all over again.

### Relaxing dependencies

![Relaxing dependences](/images/relaxingDependences.png)

To solve the scalar privatization each processor has to have it's own _t_. So no processor would be writing in another one's variable. The dataflow analysis that we need to figure out if it's privatized is an **Reaching def analysis**. The variable t is not live at the end of the iteration.

Reduction operations are parallelizable to. Think Map-Reduce

### Amdahl's Law

> "Suppose you have 10% of the program that is not parallelisable you can only speed up the program by 90%"

Hence we need to parallelise at the coarsest granularity (outer loops) minimizing communication. Or else if you found that the bulk of the computation is in a small core, you fully optimize that core.

:pencil2: //TODO understand intraprocedural parallelization

