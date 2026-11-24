# The Search and the Stack

== trace P09 ==
Call: ancestor*r(tom,X)
Call: parent(tom,*)
Exit: parent(tom,liz)
Call: ancestor*r(liz,X)
Call: parent(liz,*)
Fail: parent(liz,_)
Call: parent(liz,X)
Fail: parent(liz,X)
Fail: ancestor_r(liz,X)
Redo: parent(tom,liz)
Exit: parent(tom,bob)
Call: ancestor_r(bob,X)
Call: parent(bob,_)
Exit: parent(bob,ann)
Call: ancestor*r(ann,X)
Call: parent(ann,*)
Exit: parent(ann,sue)
Call: ancestor*r(sue,X)
Call: parent(sue,*)
Fail: parent(sue,_)
Call: parent(sue,X)
Fail: parent(sue,X)
Fail: ancestor_r(sue,X)
Redo: parent(ann,sue)
Fail: parent(ann,_)
Call: parent(ann,X)
Exit: parent(ann,sue)
Exit: ancestor*r(ann,sue)
Exit: ancestor_r(bob,sue)
Exit: ancestor_r(tom,sue)
answer 1: X=sue
Redo: ancestor_r(tom,sue)
Redo: ancestor_r(bob,sue)
Redo: ancestor_r(ann,sue)
Redo: parent(ann,sue)
Fail: parent(ann,X)
Fail: ancestor_r(ann,X)
Redo: parent(bob,ann)
Exit: parent(bob,pat)
Call: ancestor_r(pat,X)
Call: parent(pat,*)
Exit: parent(pat,jim)
Call: ancestor*r(jim,X)
Call: parent(jim,*)
Fail: parent(jim,_)
Call: parent(jim,X)
Fail: parent(jim,X)
Fail: ancestor_r(jim,X)
Redo: parent(pat,jim)
Fail: parent(pat,_)
Call: parent(pat,X)
Exit: parent(pat,jim)
Exit: ancestor*r(pat,jim)
Exit: ancestor_r(bob,jim)
Exit: ancestor_r(tom,jim)
answer 2: X=jim
Redo: ancestor_r(tom,jim)
Redo: ancestor_r(bob,jim)
Redo: ancestor_r(pat,jim)
Redo: parent(pat,jim)
Fail: parent(pat,X)
Fail: ancestor_r(pat,X)
Redo: parent(bob,pat)
Fail: parent(bob,*)
Call: parent(bob,X)
Exit: parent(bob,ann)
Exit: ancestor*r(bob,ann)
Exit: ancestor_r(tom,ann)
answer 3: X=ann
Redo: ancestor_r(tom,ann)
Redo: ancestor_r(bob,ann)
Redo: parent(bob,ann)
Exit: parent(bob,pat)
Exit: ancestor_r(bob,pat)
Exit: ancestor_r(tom,pat)
answer 4: X=pat
Redo: ancestor_r(tom,pat)
Redo: ancestor_r(bob,pat)
Redo: parent(bob,pat)
Fail: parent(bob,X)
Fail: ancestor_r(bob,X)
Redo: parent(tom,bob)
Fail: parent(tom,*)
Call: parent(tom,X)
Exit: parent(tom,liz)
Exit: ancestor_r(tom,liz)
answer 5: X=liz
Redo: ancestor_r(tom,liz)
Redo: parent(tom,liz)
Exit: parent(tom,bob)
Exit: ancestor_r(tom,bob)
answer 6: X=bob
Redo: ancestor_r(tom,bob)
Redo: parent(tom,bob)
Fail: parent(tom,X)
Fail: ancestor_r(tom,X)
== 6 answers ==

- Which of its lines push a frame onto Activity 15's runtime stack? Which
  lines pop one?

For the trace in P09, every `Call` line pushes a new activation frame that holds local variables and a return address. In this case, `Call: ancestor_r(tom,X)` shown on line 2 pushes the initial frame, `Call: parent(tom,_)` on line 3 pushes a child frame one level deeper, and `Call: ancestor_r(liz,X)` located on line 5 pushes another frame once line 4 returns. On the other hand, `Fail` lines are the ones that pop frames when a goal exhausts matching clauses and unwinds to tis caller. We see this on line 7 which is `Fail: parent(liz,_)`, on line 9 `(Fail: parent(liz,X))` and line 10 `(Fail: ancestor_r(liz,X))`. The aforementioned lines collapse their indentation levels. For some `Exit` lines like `Exit: parent(tom,liz)` (line 4) and `Exit: ancestor_r(ann,sue)` (line 29), they also return control and bindigns up one level. When compared to Activity 15, Prolog keeps a choice point when untried clauses remain while Activity 15's runtime stack pops and destroys the frame when it returns.

- What does a `Redo` line resume that a returned frame cannot?

A Redo line resumes execution from a saved choice point. When `Exit: parent(tom,liz)` on line 4 suceeds, Prolog records which `parent/2` fact matched, the variable bindings, and the remaining untried `parent/2` clauses in the knowledge base. When `ancestor_r(liz,X)` fails on line 10, `Redo: parent(tom,liz)` which is located on line 11 jumps back to that choice point, undoes the binding to `liz`, and advances to match line 12 which is `Exit: parent(tom,bob)`. Conversely, a returned frame only knows its caller's return address, therefore it cannot rewind bindings or scan alternative clauses.

- Why does swapping the two clauses of `reach/2` change whether the search
  ever answers, while swapping the two clauses of `ancestor/2` only changes
  the order of the answers?

Swapping the two clauses of `reach/2` changes termination due to its rule on recursive clause. The recursive clause `reach(X,Y) :- reach(X,Z), edge(Z, Y)`, places the recursive call first then consumes no `edge/2` fact before recursing. Placing this recursive rule first causes reach to call itself infinitely due to the way Prolog tries clauses from top-to-bottom and goals from left-to-right. As for P09, it traces `ancestor_r/2`, where the recurse clause `ancestor_r(A,D) := parent(A,P), ancestor_r(P,D)` comes first yet it will terminate. Each recursive call first invokes `parent(...)` to consume a real `parent/2` fact before recursion. The family tree is finite and acyclic, so parent eventually runs out of facts and fails, so swapping `ancestor/2`'s clauses only changes answer order rather than causing an infinite loop.
