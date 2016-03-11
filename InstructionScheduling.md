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

* _Pipelined_ functional units: occupy only one slot
* _Non pipelined_ functional units: occupy multiple time slots

### List scheduling

Schedules operations in **topological order** never backtracks. A variation is to prioritize ???

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
* Only one level of speculation
