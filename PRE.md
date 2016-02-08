## Partial Redudancy elimination (PRE)

### Preparing the flow graph
Add a basic block for every edge that leads to a basic block with multiple predecessors

**Intuition optimization:** You don't need to add the extra block if the previous block is empty and doesn't have multiple sucessors. 

### Pass 1: Anticipated Expressions
An expression is **anticipated at p** if its value computed at point p will be used along **all** subsequent paths

|                 | Anticipated Expressions                                          |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of expressions                                              |
| Direction       | backward                                                         |
| Transfer f(x)   | f<sub>b</sub>(x) = EUse<sub>b</sub> ∪ (x - EKill<sub>b</sub>)    |
| Meet            | ∩                                                                |
| Boundary        | in[exit] = ∅                                                     |
| Initialization  | in[b] = {all expressions}                                        |

The expression is killed if one of it's components it's recomputed.

### Pass 2: Place as early as possible
e will be **available at p** if e has been anticipated but not subsequently killed on **all** paths reaching p

|                 | Available Expressions                                            |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of expressions                                              |
| Direction       | forward                                                          |
| Transfer f(x)   | f<sub>b</sub>(x) = (Anticipated[b].in ∪ x) - EKill<sub>b</sub>)  |
| Meet            | ∩                                                                |
| Boundary        | out[entry] = ∅                                                   |
| Initialization  | out[b] = {all expressions}                                       |

> earliest(b) = anticipated[b].in - available[b].in

### Pass 3: Lazy code motion
An expression e is **postponable at p** if **all** paths leading to p have seen the earliest placement of e but not a subsequent use.

In other words an expression is **postponable** to the exit of block B if it is not used in the block, and either it is postponable to the entry of B or it is in earliest[B].

|                 | Postponable Expressions                                          |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of expressions                                              |
| Direction       | forward                                                          |
| Transfer f(x)   | f<sub>b</sub>(x) = (earliest[b] ∪ x) - EUse<sub>b</sub>)         |
| Meet            | ∩                                                                |
| Boundary        | out[entry] = ∅                                                   |
| Initialization  | out[b] = {all expressions}                                       |

> latest[b] = (earliest[b] ∪ postponable.in[b]) ∩ (EUse<sub>b</sub> ∪ ¬(∩<sub>s ∈ succ[b]</sub>(earliest[s] ∪ postponable.in[s])))

* Ok to place expression: earliest or postponable
* Need to place at b if either:
  * used in b, or
  * not OK to place in one of its successors
  
### Pass 4: Cleaning up

Eliminate temporary variable assignments unused beyond current block

|                 | Unused Expressions                                               |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of expressions                                              |
| Direction       | backward                                                         |
| Transfer f(x)   | f<sub>b</sub>(x) = (Euse[b] ∪ x) - latest[b])                    |
| Meet            | ∪                                                                |
| Boundary        | in[exit] = ∅                                                     |
| Initialization  | in[b] = ∅                                                        |

### Code transformation

```
For each basic block b,
 if (x+y) ∈ (latest[b] ∩ ¬ used.out[b]) {}
 else 
  if x+y ∈ latest[b]
   at beginning of b:
    create a new variable t
    t = x+y
  replace every ofiginal x+y by t
```

# Partial Redundant Assignment Elimination (PRAE)
An assignment instance a<sub>i</sub> ('x = e') is considered redundant if all paths from Start to a<sub>i</sub> has seen an instance of 'a', a<sub>j</sub> and there are no statements between a<sub>i</sub> and a<sub>j</sub> which re-define the variable _x_ or the operand of the expression _e_
