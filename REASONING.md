<!--no-pdf-->
# Why Each Prediction Reads the Way It Does

Write one entry per case, before you run anything. Name the rule behind each
predicted field. One to three sentences is enough. The grader checks the
structure of this file only. It does not judge how good the reasoning is, and
it sets no word minimum.

For every case you predicted wrongly, add a short correction after you run
it: what you predicted, what the language printed, and which rule you
misread. Corrections belong in the correction commits, not the prediction
commit.

Commit this file together with `predictions.tsv`, before any case runs:

```bash
git add predictions.tsv REASONING.md
```

The example below uses a query that is not one of the 16 cases, so it gives
no answer away. `parent(pat, X)` reads `parent(pat, jim)` because the `parent`
facts are tried top to bottom and only that one has `pat` as its first
argument:

```text
## parent(pat, X)

The goal scans the parent facts in source order. Only parent(pat, jim) has
pat first, so X binds to jim. No second fact matches, so the first ; finds
nothing and the search ends after one answer.
```

## P01

Prolog scans parent/2 facts from top to bottom, which matches the `parent(tom, liz)` first `(X=liz)` and backtracking to `parent(tom, bob)` `(X=bob)`. No other parent facts have `tom` as the first argument (or let say the parent), so further backtracking fails and the total count is 2.

## P02

Because the query contains no variables `parent(tom, bob)`, the unification checks the parent/2 facts from top to bottom and succeeds at the second fact which is `parent(tom, bob)`, which then returns true. There are no more remaining parent facts that unify with `parent(tom, bob)`, so backtracking produces `none` and the total count is 1.

## P03

Prolog scans all parent/2 facts from top to bottom looking for tom in the second argument position. Because no fact has tom as child, the unification fails on clauses, which returns false with none for the second answer, and final count of 0.

## P04

The goals execute left to right, `X = f(Y, a)` which unifies X with the compound term `f(Y, a)` while Y is still unbound, and `Y = b` then binds Y to b (which updates `X` to `f(b,a)`). Because =/2 has only one definition and no choice points remain, backtracking produces none and the final count is 1.

## P05

Unification matches the two f/2 compound terms argument-by-argument, which binds `X` to `a` and also `Y` to `b` in the query variable order `(X=a,Y=b)`. The deterministic unification does not leaves untried clauses to backtrack into, which gives second answer as none and the count is 1.

## P06

Unification checks if the atom `a` and the atom `b` is identical. Because two distinct atoms does not unify, the query immediately `fails` with false as a result and none for the second answer, which then gives count=0.

## P07

Goal order evaluates `parent(tom, P)` first, binding `P=liz`, where it fails on `parent(liz, X)` because liz is not a parent of any child, and forces backtracking to `P=bob`. From `P=bob`, `parent(bob, X)` matches `ann` first which then `(X=ann)` and backtracks to `pat` second `(X=pat)`, which exhaust all parent facts for a total count of 2.

## P08

Because of the ordering of ancestor, where ancestor/2 places its base clause parent(A, D) first, it finds tom's direct children in source order `(X=liz, then X=bob)` before entering the recursive clause. Backtracking into the recursive clause then traverses bob's descendants (which are `ann, pat, jim, and sue`), producing 6 count in total.

## P09

Because ancestor_r/2 places its recursive clause first (compared to `ancestor` where its base clause comes first), it descends through tom's first child `liz` (which fails) and the second child `bob` to find bob's descendants (which are `ann, pat, jim, and sue`) before falling back to the base clause. After exhausting the recursive branch, backtracking into the base clause `parent(tom, X)` resulting `liz` and `bob` last, keeping the total count=6.

## P10

With ancestor/2 trying its base clause first (based on the ordering), `parent(X, jim)` immediately matches `parent(pat, jim)` where `(X=pat)`. The backtracking enters the recursive clause `parent(X, P)` which `ancestor(P, jim)`, walking up the tree to find bob second `(X=bob)` and tom third `(X=tom)` for a total count=3.

## P11

Clause order tries the first rule of reach/2 first, whose leftmost goal is the recursive call `reach(a, Z)`. Because of how clauses are ordered, there are no base edge fact is consulted before recursing, the results is that Prolog enters `infinite left recursion` without ever producing an answer, making first, second, and count equal to `unbounded`.

## P12

Compared to P11, P12 reach_r/2 tries its `edge(X, Y)` base clause first before recursing, following `a -> b -> c -> d` to return true as its first answer. However, backtracking traverses the cycle (results in a loop) `a -> b -> c -> a` indefinitely and keeps rediscovering the `d`, making second true and count to unbounded.

## P13

len/2 recurses down the `[H|T]` structure of `[a, b, c]` until reaching the base clause len([], 0), then goes back while evaluating `N` is `M + 1` at each step to bind `N=3`. Because the `[]` and `[_|T]` are mutually exclusive patterns, no choice points remain, the result is that the second is none and count=1.

## P14

The join/3 matches its base clause which is `join([], L, L)` first with `X=[]` and `Y=[1,2]`, then backtracks into the recursive clause `join([H|T], L, [H|R])` which is to split the list at each element. It continues until the recursive clause fails on [], producing `X=[1],Y=[2]` second and `X=[1,2],Y=[]` third, giving a total count=3.

## P15

The `left-to-right` goal order evaluates `X > 2` before `elem(X, [1, 2, 3])` has bound X to a number. Because arithmetic comparison requires instantiated numeric operands, Prolog immediately throws an error (`instantiation error`), setting first, second, and count to error.

## P16

Compared to P15, `left-to-right` goal order runs `elem(X, [1, 2, 3])` first, which binds `X=1` and `X=2` (resulting both to fail `X > 2` and triggers backtracking) before binding `X=3`, which passes `3 > 2 (X=3)`. Backtracking past 3 reaches the empty list [] where elem/2 fails, so second is none and the count=1.

