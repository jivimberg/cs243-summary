## BDDs in Pointer Analysis

### Call graph example
You have 4 objects you need only 2 bits to represent 4 objects.

Then you set up a truth table with 2 bits for the from node, 2 bits for the 2 node and a new column saying if the relationship is true or not.

![BDD Truth table](/images/bddTruthTable.png)

We can turn this table into a decision diagram like this:

![Decision Diagram](/images/decisionDiagram.png)

Then you can start collapsing redundant nodes, for example:
* You only need one node for 0 and one node for 1
* Then if 2 nodes are identical (meaning same input gets you to same output) then you can collapse them too.
* Then you can remove unnecesario nodes. That is if the node goes to 0 or 1 regardless it's input you can just remove it from the graph.

The resulting BDD graph represents the entire function.

To represent empty set you only need 1 node.
To represent universal set you need 1 node.

It's nice because you can represent big relationscompactly. **It size depends on redundancy.**

The number of nodes in the representation depends on the order of evaluation. The size of the graph is sensitive to how you number your objects!

**Algorithm for numbering contexts:**
Start from non-expanded graph. 
1. Assign 0 if there's only one instance
2. If multiple instance assign 0...n
3. Make sure that when comming from the same context you're giving it consecutive numbers to maintain the offset in the index.

**Context sensitive pointer analysis algorithm**
1. Do context-insensitive analysis to get call graph
2. Number contexts
3. Do context insensitive analysis on the cloned graph

You can issue a query to get information from a particular instance
