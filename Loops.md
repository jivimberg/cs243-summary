# Loops

## Dominators
Node _d_ **dominates** node _n_ if every path from the start node to _n_ goes through _d_

Node _d_ **immediately dominates** node _n_ if:
  * _d_ dominates _n_
  * _d_ ≠ _n_
  * There are no nodes between _d_ and _n_
  
Immediate dominance relationships form a tree.

The **dominance frontier** of a node _d_ is the set of all nodes _n_ such that _d_ dominates an immediate predecessor of _n_, but _d_ does not strictly dominate _n_. It is the set of nodes where d's dominance stops.

### Finding dominators dataflow analysis

|                 | Finding dominators                                               |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of basic blocks                                             |
| Direction       | forward                                                          |
| Transfer f(x)   | f<sub>b</sub>(x) = x ∪ {b}                                       |
| Meet            | ∩                                                                |
| Boundary        | {entry}                                                          |
| Initialization  | in[b] = {all blocks}                                            |

**Speed:** With _reverse post order_ it can be resolved in 1 pass for most flow graphs (reducible flow graphs)

## Natural loops

* **header (_d_)**:  is the single entry of the loop. The header dominates all nodes in the loop
* **back edge (_n_ → _d_)** is an edge whose destination dominates its source (_d_ dominates _n_)
* **natural loop of a back edge (_n_ → _d_)** is d + {nodes that can reach _n_ without going through _d_}. 

A flow graph is **reducible** if every retreating edge that flow graph is a back edge.

### Finding natural loops.

1. Compute the dominators tree
2. Identify the *back edges* in the graph (where a nodes goes back to it's dominator)
2. Now remove _d_ from graph and find all predecessors of _n_. That is nodes that can reach _n_ without going through _d_

### Inner loops
If 2 loops don't have the same header:
 * They are either disjoint or
 * One is entirely contained (nested) within the other
  
If 2 loops share the same header is hard to tell which is the inner loop, so we just combine them as one. 
