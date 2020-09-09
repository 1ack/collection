
# fairness

If S is a safety property and L a liveness property, then the pair S,L is said to be machine closed iff the following condition is satisfied
```
Every finite sequence of states that satisfies S can be extended to a behavior that satisfies S ∧ L .
```

L is a fairness property for S iff S,L is machine closed. We can then call L a fairness property if the safety
property S is understood from the context.


**The WF and SF Operators**

An action formula A is a predicate on steps.

a step is an A step iff A equals true on that step

The formula ENABLED A is defined to equal true on a state u   
iff there exists a state w such that u → w is an A step.  

A reachable state of S is one that occurs in some behavior that satisfies S  

在formula S的约束下，不一定存在 u → w 的reachable state  

ENABLED A 意味着存在两个使A为true的两个state，但这两个state不一定能构成一个符合S的step


For an action A , the action <\<A\>>_v is defined to equal A /\ (v’/= v) .

We define a behavior b to satisfy WF_v(A) iff any of the following equivalent conditions are satisfied
* Any suffix of b whose states all satisfy ENABLED <\<A\>>_v contains an <\<A\>>_v step.


We define SF by defining a behavior b to satisfy SF_v(A) iff the following equivalent conditions are satisfied.
* Any suffix of b containing infinitely many states that satisfy ENABLED <\<A\>>_v contains an <\<A\>>_v step.

Any behavior satisfying SF_v(A) also satisfies WF_v(A) .

If <\<A\>>_v is not equivalent to FALSE , then WF_v(A) and
SF_v(A) are liveness properties. 


an action B is defined to be a subaction of Next   
iff the following condition is satisfied: 
```
If u is a reachable state of S and u → v is a step satisfying B ,then u → v satisfies Next .
```

Note that B is a subaction of B \/ C for any action C .

S equals Init /\ [][Next]_vars for some state formula Init , action Next , and state expression vars .

Formulas WF_v(A) and SF_v(A) are fairness properties for S if <\<A\>>_v is a subaction of Next


Weak fairness over an action means if that action is continuously enabled, it must eventually be taken.

Weak fairness WF_e(A) means if action A is enabled continuously (i.e. without interruptions), it must eventually be taken. 

Strong fairness SF_e(A) means if action A is enabled continually (repeatedly, with or without interruptions), it must eventually be taken.