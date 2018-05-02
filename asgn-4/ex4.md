---
title: Logic in Computer Science - Assignment 4
author: Frederik Hanghøj Iversen <hanghj@chalmers.se>
date: Sep. 25 2017
...

\newcommand*{\defeq}{\mathrel{\vcenter{\baselineskip0.5ex \lineskiplimit0pt
                     \hbox{\scriptsize.}\hbox{\scriptsize.}}}%
                     =}

Problem 1
=========

We start with the observation

\begin{align*}
p^{0}        & = 1                                        \\
p^{2n}     & = \left( p^{2} \right) ^ {n}               \\
p^{2n + 1} & = \left( p^{2} \right) ^ {n} \cdot p
\end{align*}

And define the following program implementing this idea:

    pow p 0 = 1
    pow p n =
      if even n
      then pow (n * n) (n     `div` 2)
      else pow (n * n) (n - 1 `div` 2)

This algorithm is correct by the above observation.

The amount of multiplications performed to compute `pow p n` is given by:

\begin{align*}
T(0) & = 0 \\
T(n) & \leq T(2^{i+1}) && \text{For some $i \in \mathbb{N}$}   \\
     & = 1 + T(2^i)    && \text{By definition of \texttt{pow}} \\
     & \Rightarrow     && \text{By the master theorem}         \\
T    & \in O(n \mapsto \log n)
\end{align*}

As desired.

The master theorem is applicable because of the last equality and because $T(n)$
is smaller than this function. So we also use the proposition $f \leq g \land g
\in O(\bar{g}) \Rightarrow f \in O(\bar{g})$.

Problem 2
=========

Problem 2.a
-----------

The following program implements equality testing of sets of comparable
elements:

    eq xs ys = eq' (sort xs) (sort ys)
    
    eq' [] [] = True
    eq' [] _  = False
    eq' _  [] = False
    eq' (x:xs) (y:ys)
      = case x `compare` y of
        EQ -> eq' xs ys
        _  -> False

The runtime of `eq` is $O(n \mapsto n \cdot \log n)$. This is becasue we first
sort the input arrays. This can be done in $O(n \mapsto n \cdot \log n)$ we then
perform a simple fold over the arrays which can be performed in $O(n \mapsto
n)$. So to finish the argument we apply the proposition $f \in O(\bar{f}) \land
g \in O(\bar{g}) \Rightarrow f + g \in O(max(\bar{f},\bar{g}))$.

Problem 2.b
-----------

There may be a much simpler algorithm but here is my attempt which performs as
well as any solution depending on sorting the input lists.

We first calculate the intersections of all the intervals. This will give us a
new list of disjoint intervals for which we *can* use the naïve approach of just summing the intervals lengths.

Here is a sketch of the algorithm:

    cover is =
      let xs = [ (x, p)
         | x all points in is
         , p indicates if x is a start-point or an endpoint 
           and which interval it comes from
         ]
      let ys = intersections xs
      return sum of lengths of ys
    
    intersections (x : xs) =
      let R, S = empty set
      add start-point of x to R
      add interval for   x to S
      for x in tail xs:
        if x is an endpoint:
          remove the interval of x from S
        else:
          add    the interval of x to S
        if S is empty:
          add endpoint of x to R
      return R

Again, we can sort the arrays in $O(n \mapsto n \cdot \log n)$ time. We can sum
the lengths of the intersections in linear time. The tricky part is the analysis
of `intersections`. The loop runs `n-1` times and at each iteration we perform a
lookup/addition/removal in a set whose size is bounded by $n$. This can be done
in $O(\log n)$ time. So the complexity of `intersections` is $O(n \mapsto n
\cdot \log n)$ which is then also the overall complexity of `cover`.

Problem 2.c
===========

First off we assume that the words in the set are all unique (you could argue
that this holds since it is a *set*). We need this condition because otherwise
the problem may not have a solution.

Denote the ordered set of strings by $S = \{ s_i \in \Sigma^* \mid i \in \{1,
\ldots n \} \}$. We can now calculate the set of prefixes in the following way.
Define $\mathit{suf}$ *iteratively* thus (the set $S$ is an implicit variable to
$suf$):

\begin{align*}
\mathit{suf} & : \Sigma^* \rightarrow \mathbb{N} \rightarrow \mathbb{N} \rightarrow \Sigma^* \times \mathbb{N} \\
\mathit{suf}(C, i, j) & =
\begin{cases}
  \mathit{suf}(C, i, j+1)               & \text{if $s_{i-1}|_j \leq s_i|_j$} \\
  \mathit{suf}(C \cup \{s_i\}, i+1, j)  & \text{if $s_i|_j \leq s_{i+1}|_j$} \\
  \{ (C, j) \} \cup \mathit{suf}(\{s_i\}, i+1, S_{i-1} \cap s_i) & \text{otherwise}
\end{cases} \\
\mathit{suf}(C, n, j) & = \{(C,j)\}
\end{align*}

$\mathit{suf}$ returns a list of partitions of the input paired with a number.
This number indicates the length of the prefixes that makes every string in each
partition unique. In the definition of $\mathit{suf}$ I've made use of some
auxillary functions (and overloading of symbols) as follows:

For $a \in \Sigma^*$ let $a|_n$ denote the prefix of $a$ with $n$ elements.

For $a , b \in \Sigma^*$ let $a \leq b$ assert the fact that $a$ is a prefix of
$b$.

For $a , b \in \Sigma^*$ let $a \cap b$ denote the minimum number such that
$\neg ( a \leq b)$.

An invartiant for this algorithm is that all elements of the first parameter to
$\mathit{suf}$, the ordered set $C$, contains elements that are all prefixes of
the its last element. This set must also be non-empty.

We can design an algorithm from this function in quite a straight-forward way
that has the complexity $O(n, m \mapsto n \cdot m)$. The algorithm will be a
direct translation of the definition given above albeit with a simple
modification. When we have to calculate $s_{i-1}|_j \leq s_i|_j$ in the first
conditional it suffixes to calculate $(s_{i-1})_j = (s_i)_j$ i.e. establish if
the strings deviate in their $j$'th component. This is a necessary and
sufficient condition for the other elements of $C$ to be a prefix of the new
element $s_i$ (due to the invariant of $C$).

The argument for the complexity then goes like this: At each step of the
variable $i$ we have to do work at most $O(n, m \mapsto m)$ since in the case
when $i$ does not increase can happen at most $m$ times (the strings in the
input are unique). And at each step where $i$ *does* increase we also do $O(n, m
\mapsto m)$ work for a total of $O(n, m \mapsto n \cdot m)$ work.

I know that my hand-in should also contain a proof of correctness and, I assume,
optimality. I won't give a full proof because I think that the exposition is
complicated enough as-is and I think it would simply become a too huge task to
give this proof in full detail. But the proof should necessarily show the
invariant for $C$ and then use this to show that if you take prefixes of
elements from $C$ of the lengths that it gets paired with you get minimal unique
suffixes. The proof would show this by also showing that $\leq$ ("prefix-of") is
a transitive relation. I.e.: adding an element to $C$ for which the last element
of $C$ is a prefix of will imply that all elements in the new set are prefixes
of the new last element.
