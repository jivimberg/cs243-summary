## Reaching definitions

A definition **_d_ reaches a point _p_** if there exists path from the point immediately following _d_ to _p_ such that d is not killed (overwritten) along that path.

|                 | Reaching definitions                                             |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of definitions                                              |
| Direction       | forwards                                                         |
| Transfer f(x)   | f<sub>b</sub>(x) = Gen<sub>b</sub> ∪ (x - EKill<sub>b</sub>)     |
| Meet            | ∪                                                                |
| Boundary        | out[entry] = ∅                                                   |
| Initialization  | out[b] = ∅                                                       |

where Kill[s] = set of all other defs to x in the rest of program

## Live variable analysis

A variable **_v_ is live at point _p_** if the value of _v_ is used along some path in the flow graph starting at _p_.

|                 | Live variables                                                   |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of variables                                                |
| Direction       | backwards                                                        |
| Transfer f(x)   | f<sub>b</sub>(x) = Use<sub>b</sub> ∪ (x - Def<sub>b</sub>)       |
| Meet            | ∪                                                                |
| Boundary        | in[exit] = ∅                                                     |
| Initialization  | in[b] = ∅                                                        |


## Live Range

Live range = Live variable ∩ Reaching definition

## Available Expressions

|                 | Available expressions                                            |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of variables                                                |
| Direction       | forwards                                                         |
| Transfer f(x)   | f<sub>b</sub>(x) = Gen<sub>b</sub> ∪ (x - Kills<sub>b</sub>)     |
| Meet            | ∩                                                                |
| Boundary        | out[entry] = ∅                                                   |
| Initialization  | out[b] = U                                                       |

## Constant propagation

Replaces expressions that evaluate to the same constant every time they are executed.

**Semilattice**

![semilattice for constant propagation](/images/semilatticeCP.png)

**Transfer function**

![transfer function for constant propagation](/images/transferFunctionCP.png)

Transfer function for _x = y + z_

![transfer function for x = y + z](/images/transferFunctionForX=Y+Z.png)

Constant Propagation is **not distributive**. We prove that it's not distributivity with an example.

## Lock

Analysis that execute warnings in the following cases:
* **Warning I:** LOCK → LOCK. Issue a warning on a LOCK operation if it can potentially follow another LOCK operation.
* **Warning II:** UNLOCK → UNLOCK. Issue a warning on an UNLOCK operation if it can potentially follow another UNLOCK operation.
* **Warning III:** LOCK → EXIT Issue a warning on a LOCK operation if the program may terminate without performing an UNLOCK.
* **Warning IV:** ENTRY → UNLOCK. Issue a warning on an UNLOCK operation if it may be executed before any LOCK operations.

|                 | Locking                                                          |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Values of lock                                                   |
| Direction       | forwards                                                         |
| Transfer f(x)   | **if x == Lock()** ⇒ f<sub>b</sub>(x) = Locked; **if x == Unlock()** ⇒ f<sub>b</sub>(x) = Unlocked; **Else** propagate input      |
| Meet            | _see lattice_                                                    |
| Boundary        | out[entry] = Undefined                                           |
| Initialization  | out[b] = Undefined                                               |

**Lattice**

![lock lattice](/images/lockLattice.png)

**Note:** The following analysis covers the first 3 warnings. For warning IV we need to do a backward analysis or a different forward analysis. For example we could do an analysis that tags each LOCK statement and treats them like reaching definitions (issuing the warning if any LOCK operation reaches the exit block).

Both analysis are **monotone** and **distributive**.

##  Uninitialized variables

|                 | Uninitialized variables                                          |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of variables                                                |
| Direction       | forwards                                                         |
| Transfer f(x)   | **if x = u** ⇒ mark x as _defined_ **else** _check the table below_ |
| Meet            | _see lattice_                                                    |
| Boundary        | OUT[Entry] is an array with all variables initialized to **not used**|                                  
| Initialization  | init all variables to **not used**                               |

**Lattice**

![uninitialized variables lattice](/images/UninitializedVariablesLattice.png)

**Transfer Function** (continuation)

![uninitialized variables transfer function](/images/UninitializedVariablesTF.png)

## Null check

## Dead code elimination (aka Faint variables)

Any assignments or addition that do not contribute to the computation of the variables used in output and conditional statements should be eliminated

|                 | Dead code elimination                                            |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of variables (variables that will be used in the computation of a branch or output statement |
| Direction       | backward                                                         |
| Transfer f(x)   | _see below_                                                      |
| Meet            | _see lattice_                                                    |
| Boundary        | in[Exit] initialized to top (empty set)                          |                                  
| Initialization  | all interior nodes initialized to top (empty set)                |

**Transfer function**

Given an input set of variables keep everything the same but:
* if output(x): add x to the set
* if x1 = x2: Add x2 to the set if x1 is in the set. _Remove x1 from the set if present_.
* if x1 = x2 + x3: Add x2 and x3 to the set if x1 is in the set.  _Remove x1 from the set if present_.
* if (x) goto L: Add x to the set

**After running the analysis** we can remove the statement if the target variable **is not** in the variable set immediately after the statement.

This analysis is **monotone** and **distributive**.

## Taint analysis

|                 | Dead code elimination                                            |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of variables for each print                                 |
| Direction       | backward                                                         |
| Transfer f(x)   | _see below_                                                      |
| Meet            | _see lattice_                                                    |
| Boundary        | in[Exit] initialized to top (empty set)                          |                                  
| Initialization  | all interior nodes initialized to top (empty set)                |

**Transfer function** m' = f(m)
* a = ... if a ∈ m, m' = m \ {a}
* a = b if a ∈ m, m' = m \ {a} ∪ {b}
* a = b + c if a ∈ m, m' \ {a} ∪ {b} ∪ {c}
* a = read(), if a ∈ m, m' = m \ {a}
* print(a), if this is the print I'm calculating m'= m ∪ {a}

**After running the analysis** we go through the program one more time. For each INPUT if the var assigned by the input is in the variable set immediately after the INPUT for a PRINT we put the line number of the input on that PRINT. Then we issue warnings if there are variables on the set for each print.

This analysis **converges** is **monotone** and **distributive**.

## Same value analysis

|                 | Same value analysis                                              |
| --------------- | ---------------------------------------------------------------- |
| Domain          | Sets of partitions of variables that have the same value         |
| Direction       | forward                                                          |
| Transfer f(x)   | **if x = y** ⇒ remove _x_ from it's block and add it to _y_ ; if **x = x + 1** Remove x from it's current block and place it in a new empty one     |
| Meet            | the partition that places variables x and y in the same equivalence class if and only if both argument partitions place them in the same equivalence class.                                 |
| Boundary        | in[Exit] initialized to top                                      |                                  
| Initialization  | all interior nodes initialized to top                            |

Meet example:
````
{{a,b,c},{d,e},{f}} /\ {{a,b},{c,d},{e,f}} = {{a,b},{c},{d},{e},{f}}.
````
