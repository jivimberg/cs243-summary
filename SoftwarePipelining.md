## Software pipelining

### Loop unrolling (for "DoAll" loops)

**Length of u iteration** = length of it + initiation interval (u - 1)
**Execution time per source iteration** = Length of u iteration / u → lim to ∞ = initiation interval

The wider the machine the more you unroll

### Software pipelined code

Unlike loop unrolling we always use the same schedule. That way you find an steady state.

**Execution time for pipelined code** = Steady state size * iterations + remaining operations.

The locally compacted code may not be optimal, but we only care that the steady state is tight.

With "doAccross" loops there's a limit of how much you can parallelise the code.

#### Resource bound on Initiation Interval

```
for all resource i
  number of units require by one iteration: n<sub>i</sub>
  number of units in system: R<sub>i</sub>
  
Lower bound = max<sub>i</sub> n<sub>i</sub>/R<sub>i</sub>
```

#### Scheduling constraint: Resource

**Initiation interval = size of steady state**

![resource reservation table](/images/resourceReservationTable.png)

#### Scheduling constraint: Precedence

![precedence](/images/precedence.png)

Label edges with <𝛿, d>
* 𝛿 = iteration difference
* d = delay

```
𝛿 * T + S(n<sub>2</sub>) - S(n<sub>1</sub>)  ≥ d
```

Cycles in the dependence graph mean that now you're bound on both ends. You can't stretch T arbitrarily 

**Algorithm for bound on Initiation Interval based on precedence**
```
for all cycles c,
  max<sub>c</sub> CycleLength(c) / Iteration difference(c)
```

### Algorithm for Acyclic graphs

```
Find lower bound of initiation interval: T<sub>0</sub>
  based on resource constraints

For T = T<sub>0</sub>, T<sub>0</sub> + 1, ... until all nodes are scheduled
  For each node n in topological order
    s<sub>0</sub> = earliest n can be scheduled
    for each s = s<sub>0</sub>, s<sub>0</sub> + 1
      if NodeScheduled(n, s) break;
    if n cannot be scheduled break;
    
NodeScheduled(n, s)
  Check resources of n at s in modulo reservation table
```

If your loop is such that:
* Every operation uses only 1 resource
* There are no cyclic dependencies on the loop

**You can always meet the lower bound.** You just keep stretching the cycle until you pack them all in.

### Algorithm for Cyclic graphs

**Strongly connected component:** A set of nodes such that every node can reach every other node. Every node constraints all others from above and below. As each node is scheduled we find the lower and upper bounds of all other nodes in the SCC

SCC are hard to schedule. When you're working with the critical cycle (cycle lenght / iteration difference = T) you have **no slack**.

Edges between SCC are acyclic. So when backtracking we can move the SCC as a whole moving it's starting point. We can only try T times because on T + 1 we would be repeting so there's no point on keep trying.

This algorithm works for acyclic graphs too:
```
Find lower bound of initiation interval: T<sub>0</sub>
  based on resource constraints and precedence constraits

For T = T<sub>0</sub>, T<sub>0</sub> + 1, ... until all nodes are scheduled
  E<sup>*</sup> =  longest path between each pair
  For each SCC c in topological order
    s<sub>0</sub> = Earliest c can be scheduled
    For each s = s<sub>0</sub>, s<sub>0</sub> + 1, ... s<sub>0</sub> + T - 1 
      If SCCScheduled(c, s) break;
    If c cannot be scheduled return false;
  Return true;
  
SCCScheduled (c, s)
  Schedule first node at s, return false if fails
  For each remaining node n in c
    s<sub>l</sub> = lower bound on n based on E<sup>*</sup>
    s<sub>u</sub> = upper bound on n based on E<sup>*</sup>
    For each s = ss<sub>l</sub>, s<sub>l</sub> + 1, ..., min(s<sub>l</sub> + T - 1, s<sub>u</sub>)
      if NodeScheduled(n, s) break;
    if n cannot be scheduled return false;
  Return true;
```
