# basic knowledge

```
formula 描述变量之间的关系，结果是true/false

invariant 每个可到达的状态（state)都为true的formula

array expressiong:
 sqr == [i \in 1..42 |-> i*i] 
 意为：sqr[i] = i* i
 i的范围称为domain 
 sqr称为function

\E r :declares r local to formula

\A r \in RM: for r in all RM ,the subformula is true

\= 或 # ：不等于

~：取反

\*:注释到行尾
（* 。。。 *）：块注释

[prof |->"Fred",num|->42] is a function f with doamin {"prof","num"}
such that f["prof"]= "Fred"
f["num"] = 42
f.prof是f["prof"]的简写

rmState' = [rmState EXCEPT ![r] = "prepared"]:仅r对应的值变化，其他维持不变


[f EXCEPT !["prof"]="Red"]
可以简写为
[f EXCEPT !.prof="Red"]

\subseteq : 子集

\cup 或者 \union：并集

\cap 或者 \intersect：交集

<<  >> :元组

Enabling conditions:Conditions on the first state of a step 原始状态

CHOOSE v \in S:P equals: 如果S中存在v，使得P为true，then such v
else :未定义    只返回一个值

Nat：自然数集合

/\ Majority \subseteq SUBSET Acceptor: 其中，"SUBSET Acceptor" 意为 Acceptor的所有子集的集合 pwoerset of Acceptro，写作 P(Acceptor)


S\T : 在S中，但不在T中的元素集合


[aa:bb,cc:dd] : record

let 。。。 in 。。。: 定义局部变量

{v \in S:P}  : S里满足P的所有v的集合, the subset of S consisting of all v satisfying P

{m.bal:m \in mset} : the set of all m.bal with m in mset

对称的变量不能用在choose语句里

P => Q  : if  P ,then  Q ,else true (we know nothing)

p => Q equals !Q => !P

A module-closed (module-complete)expression is a TLA+  expression that contains only:

built-in TLA+ operators and constructs

numbers and strings

declared constants and variables

identifiers declared locally within it



A module-closed formula is a Boolean-valued module-closed expression.

Constant Expressions:
has no declared variables
has no non-constant operators(except prime and unchanged)


ASSUME ...
must be constants formula



State Expressions: 
can contain anything a constant expression can as well as declared variables

A state expression has a value on a state.
a state assigns values to variables.
if state s assigns v<- Nat and w<- -42,then
v U {w} has the value Nat U {-42}
on state s


Action Expressions:
can contain anything a state expression can as well as '(prime)
  and UNCHANGED

a action expression has a value on a step(pair of states)

a state expression is an action expression  whose value on the setp s->t depends only on state s
(所有的action都符合这个要求？TPNext?)
a action formula is called an action

For any state expression e the value of the action expression e' on s->t is the value of e on state t

A temporal formula has a Boolean value on a sequence 
s1=>s2->s3->... of states
A behavior:a sequence of states


[] read always
[]TPNext is true on s1->s2->s3->... iff 
TPNext is true on si->si+1 for all i


Init /\ [] [Next]_<<v1,...,vn>>
initial formula Init
next-state formula Next 
declared variables v1,...,vn

TPNext is a state formula,also is an action

THEOREM TF asserts that TF is true on every posible behavior

```
