## Software pipelining

### Loop unrolling (for "DoAll" loops)

**Length of u iteration** = length of it + initiation interval (u - 1)
**Execution time per source iteration = **Length of u iteration** / u → lim to ∞ = initiation interval

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
