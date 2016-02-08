#Dataflow Analysis

## Reaching definitions

A definition **_d_ reaches a point _p_**
if there exists path from the point immediately following d to p such that d is not killed (overwritten) along that path.

Live range = Live variable ∩ Reaching definition
