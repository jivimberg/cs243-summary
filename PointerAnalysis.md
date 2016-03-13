## Pointer Analysis

Inter-procedural, context-sensitive, flow-sensitive.

### Datalog

Stratification: a negation can only appear in the right hand side of another stratum. Stratified datalog will terminate 
* Compute one group at a time
* Can negate only the result of previous strata

//TODO REVIEW

### Flow insensitive Points-to analysis

**Alias analysis:** Can 2 pointers point to the same location
**Points-to analysis:** What objects does each pointer points to? 2 pointers **cannot be aliased** if they **must** point to different objects. Therefore I will consider them to be aliased if they may point to the same thing

We collapse objects by giving them the same name. For each line of code we say that all objects created in that line of code we give it the same name. This is **very** conservative.

In a program you can have:
* variables (v)
* Heap-allocated objects (h)
  * fields on an allocated object(f)
  
![Pointer Analysis Rules](/images/pointerAnalysisRules.png)

**For flow insensitive:** On assignment there is no kill. If it used to point to a different thing now it points to both things

```
  1. Convert source program to datalog: assign numbers to elements in domain
  2. Extract initial relations
  3. Generate predicate dependency graph
  4. Determine iteration order
  5. Apply rules until convergence
```

![Datalog Iteration Order](/images/datalogIterationOrder.png)

**Iteration order:**
1. Not recursive first
2. Then conected components as individuals
3. Then as a whole

This is just an optimization. You can execute them in any order and it'll work.

#### Virtual method invocation

**Class hierarchy analysis cha(t,n,m)**:
* Given an invocation v.n(...), if v points to object of type t, then m is the method invoked
* m is in t's first superclass that defines n
* _Optimization:_ if you know that the program only allocates on type of object then you don't have to do cha

Can be computed statically from the definitions!

#### We can use the Pointer analysis information to improve the Call Graphs

If you know the type of the object you can know exactly with method is being called and runtime and shave off impossible relationships.

![Call Graph Improved](/images/callGraphImproved.png)

### Context-Sensitive Pointer analysis

Without context sensitivity more or less is like doing type analysis: _"this are the kinds of types that can be passed in"_

**# of contexts is exponential!** (even without recursion)

![Context explosion](/images/contextExplosion.png)

**How we handle recursion:** 
1. Figure out which are the recursive cycles 
2. Expand the graph but treat each recursive unit as an instance. Meaning if you have N different ways of calling the cycle then you'll have 3 different instances.
3. Note that you'll also have to copy the things that the recursive unit references

![Handling recursion](/images/handlingRecursion.png)
