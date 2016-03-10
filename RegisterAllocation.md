# Register Allocation

## Algorithm

```
Step 1. Build interference graph
  a. find nodes
  b. find edges
Step 2. Coloring 
```

## Building the interference graph

We draw an edge to the interference graph if there are 2 overlaping **live ranges** at some point in the program (remember _Live range = Live variable ∩ Reaching definition_).

**Optimization:** Check for interference only at the start of each life range. This works even if the ranges overlap in the middle because if the variable was previously undefined we can assign any value to it.

We can assign different registers for different live ranges of the same var. But **if 2 live ranges overlap** for the **same** variable must be merged (treated as the same live range).  
In other words if var is live an there are multiple livereaching definitions then we merge.

## Coloring 

1. A node with degree < n ⇒ it can be colored
2. A node with degree = n ⇒ we don't know
3. A node with degree > n ⇒ we don't know

```
Iterate until stuck or done
  a. Pick any node with degree < n
  b. Remove the node and its edges from the graph
If done (no node left)
  a. Reverse the removed nodes and color.
```

**If you get stuck** choose a node by heuristics (least frequently executed, has many neighbors) place v on stack and mark it to know that you may have to spill in that node. 

The minimum number of registers needed to color an interference graph can be obtained by the **biggest clique**. A **clique** is a subset of vertices of an undirected graph, such that its induced subgraph is complete.
