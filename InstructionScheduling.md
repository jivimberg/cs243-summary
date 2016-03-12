# Instruction Scheduling

### Scheduling constraints

* Data dependences
* Control dependences
* Resource constraints

#### Data dependence

- **True dependence:** write → read (RAW hazard)
 
 ```
 a = ...
 ... = a
 ```
- **Output dependence:** write → write (WAW hazard)
 
 ```
 a = ...
 a = ...
 ```
- **Anti-dependence:** read → write (WAR hazard)

 ```
 ... = a
 a = ...
 ```
 
#### Resource constraints

Using **resource reservation table**

* _Pipelined_ functional units: occupy only one slot. You **can** schedule other operations even if the previous one has not finished yet.
* _Non pipelined_ functional units: occupy multiple time slots. You **can't** schedule other operations if the previous one has not finished yet.

Some machines also have restrictions on the **amount of instructions that can be issued per clock**. You can model this by adding a new column to the resource reservation table where you keep an int with the amount of resource available.

### Basic block scheduling

![basic block example](/images/basicBlockExample.png)

Since we do not know how the values of R1 and R7 relate, we have to consider the possibility that an address like 8 (R1) is the same as the address 0 (R7). That's why we have arrows from i<sub>1</sub>, i<sub>2</sub>, i<sub>3</sub> and i<sub>6</sub> to i<sub>7</sub>

#### List scheduling

```
READY = nodes with 0 predecessors

Loop until READY is empty{
 Let n be de node in READY with highest priority
 
 Schedule n in the earliest slot that satisfies precedence + resource constraints
 
 Update predecessor count of n's successor nodes
 Update READY
}
```

Schedules operations in **topological order** and never backtracks. 

Variations for the priority function:
* **Critical path** max clocks from n to any node
* Resource requirements
* Source order

### Global scheduling

* **Control equivalence** Two operations are control equivalent if o<sub>1</sub> is executed if and only if o<sub>2</sub> is executed
* **Control dependence** An o<sub>2</sub> is control dependent on o<sub>1</sub> if the execution of o<sub>2</sub> depends on the outcome of o<sub>1</sub>
* **Speculation** An operation is speculatively executed if it is executed before **all** the operations it depends on (control-wise) have been executed

#### Code motions

Moving instructions **up**:
* Move instruction to a cutset (from entry). This includes moving it to a Control Equivalent
* Speculation: even when not anticipated

Moving instructions **down**:
* Move instruction to a cut set (from exit)
* Can duplicate code too provide one path where the op is executed and one where it's not

| :warning: Remember to updated the constraints upon moving an expression |
|---|

### A basic scheduling algorithm

* Schedule innermost loops first
* Only upward code motion
* No creation of copies
* Only one level of speculation. 

#### Program representation

* A **region:** in a CFG is a set of basic blocks such that _control from outside the region must enter through a single entry block_
* A **function** is represented as a hierarchy of regions. The whole program is a region. Each natural loop is a region. Natural loops are hierarchically nested

#### Algorithm

Schedule regions **from inner to outer** treating each inner loop as a black box unit. We ingore back edges so inside each region we have an acyclic graph

```
for each region from inner to outer {
 For each basic block B in prioritized topological order {
  CandidateBlocks = ControleEquiv{B} ∪ Dominated - Successors{ControlEquiv{B}};
  CandidateInsts = ready operations in CandBlocks;
  
  for (t = 0, 1, ... until all operations from B are scheduled){
   for(n in CandidateInsts in priority order){
    if (n has no resource conflict at time t){
     S(n) = <B, t>
     Update resource commitments
     Update data dependencies
    }
   }
   
   Update CandidateInsts;
  }
 }
}
```

A prioritization could be the basic block most likely to be executed.
`CandidateBlocks = ControleEquiv{B} ∪ Dominated - Successors{ControlEquiv{B}};` here we see that we only consider one level of speculation
`S(n) = <B, t>` Means that we schedule on **n** the instruction from basic block B offset t

**Priority function:** Non-speculative before speculative
