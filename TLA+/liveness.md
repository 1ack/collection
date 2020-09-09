

[]P means that P is true for all states. 


<> means eventually: <>P means that for every possible behavior, at least one state has P as true. 

As long as every behavior has at least one state satisfying the statement, an eventually is true.


~> means leads to: P ~> Q implies that if P ever becomes true, at some point afterwards Q must be true.


<>[] means stays as: <>[]P says that at some point P becomes true and then stays true.   

If your program terminates, the final state has to have P as true. Note that P can switch between true and false, as long as it eventually becomes permanently true.

[]<> P to mean P is true infinitely often

**Liveness is Slow**
It’s easy to check that reachable states satisfy invariants. But to check liveness, how you reach those states also matters. If a thousand routes lead through state S, that’s 1,000 routes that need to be checked for liveness and only one state that needs to be checked for safety. So liveness is intrinsically much, much slower than checking invariants.


**Temporal Properties are Very Slow**
 