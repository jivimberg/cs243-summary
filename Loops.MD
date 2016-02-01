# Loops

## Dominators
Node _d_ dominates node _n_ if every path from the start node to _n_ goes through _d_

Node _d_ **immediately dominates** node _n_ if:
  * _d_ dominates _n_
  * _d_ ≠ _n_
  * There are no nodes between _d_ and _n_
  
Immediate dominance relationships form a tree.

### Finding dominators dataflow analysis

## Natural loops

**header (_d_)**:  is the single entry of the loop. The header dominates all nodes in the loop

**back edge (_n_ → _d_)** is an edge whose destination dominates its source (_d_ dominates _n_)

**natural loop of a back edge (_n_ → _d_)** is d + {nodes that can reach _n_ without going through _d_}
