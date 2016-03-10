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
