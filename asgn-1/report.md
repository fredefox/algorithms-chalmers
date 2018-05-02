---
author: Frederik Hanghøj Iversen
title: Algorithms - Assignment 1
date: September 5th 2017
...

\newcommand*{\QED}{\hfill\ensuremath{\square}}

# Problem 1

The inner loop is performed $\frac{n-p^2}{p}$ times. This is due to the fact
that $m \in \lbrace p^2 , \ldots , p^2 + i * p , \ldots n \rbrace$ and the size
of this set is exactly that. So the sum we arrive at is

\begin{align*}
T(n) & = \sum_{p = 2}^{\lfloor\sqrt{n}\rfloor} \frac{n-p^2}{p}
       = \sum_{p = 2}^{\lfloor\sqrt{n}\rfloor} \left( \frac{n}{p} - p^2 \right)
       \leq \sum_{p = 2}^{\lfloor\sqrt{n}\rfloor} \frac{n}{p}
       = n * \sum_{p = 2}^{\lfloor\sqrt{n}\rfloor} \frac{1}{p} \\
     & \leq n * c_0 * lg(\lfloor \sqrt{n} \rfloor)  && \text{For some $c_0$} \\
     & \leq n * c_0 * lg(\sqrt{n}) \\
     & \Rightarrow \\
T    & \in O ( n \mapsto n * lg(n))
\end{align*}

The second inequality is due to the fact that the harmonic sequence is bounded
by $O(x \mapsto lg(x))$. \QED

# Problem 2

The following program `br` implements blit-rotate

    br []      = x
    br x@(_:_) = x
    br x
        let (nw, ne, sw, se) = split_in_four(x)
        rotate(br(nw), br(ne), br(sw), br(se))

The function `rotate` takes four blocks and *blit* them into place. In the
problem statement it is hinted that this can be done using 5 *blits* so we will
assume this in the following.

For the reminder of this problem I want to make use of the following theorem:

\newtheorem{thm1}{Theorem}
\begin{thm1}
\label{thm1}
\begin{align*}
T(1)       & = k \\
T(c^{i+1}) & = a + b * T (c ^ i) \\
           & \Rightarrow \\
T          & \in O ( c^i \mapsto b ^ { i } ) = O ( n \mapsto b ^ { lg_c n})
\end{align*}
\end{thm1}

The proof will be on induction on the argument, $c^i$, to $T$. For $i=0$ we
obviously have that $k \in O (n \mapsto b^{lg_c n})$. For the inductive step let $T_i \in O( i \mapsto b ^ {i * n})$:

\begin{align*}
T ( c ^ {i + 1}) & = a + b * T(c^i) \\
    & \leq a + b * b ^ { i } * c_0    && \text{Inductive hypothesis. For some $c_0$} \\
    & = a + b^{i+1} * c_0 \\
    & \Rightarrow \\
T   & \in O ( c^i \mapsto b^i ) = O(n \mapsto b ^ {lg_c n})
\end{align*}

## Question a

Assuming that the width of the array is a power of 2 the amount of *blits*
performed is given by the reccurence:

\begin{align*}
B(2^0) & = 0           \\
B(2^{i+1}) & = 5 + 4 * B(2^i)
\end{align*}
Where the argument to $B$ is the size of the problem.

In the base-case ($i=0$) no blits are performed. In the other case 5 blits are
performed and `br` are called 4 times on a problem of half the size whose number
of blits then is given by $B(2^{i-1})$.

The closed form of $B$ is:

\begin{align*}
B(2^0) & = 0 \\
B(2^i) & = 5 \ \frac{4^{i-1}-1}{3}
\end{align*}

Proof: For $i=0$ it holds by definition. For $i=1$ the closed form of $B$ gives:

\begin{align*}
B(2^1)
  & = 5 \ \frac{4^{1-1}-1}{3}
    = 0
\end{align*}

Now assume that it holds for $i > 0$ now we must show it for $i+1$:

\begin{align*}
B(2^{i+1})
  & = 5 + 4 * B(2^i)
    = 5 + 4 * 5 \ \frac{4^{i-1}-1}{3}
    = 5 * \left( 1 + 4 \ \frac{4^{i-1}-1}{3} \right)
    = 5 \ \frac{4^i-1}{3}
\end{align*}

And this concludes our proof. \QED

## Question b

Assuming that we can calculate `split_in_four(x)` in constant time. The
execution time of `br` is given by:

\begin{align*}
T(2^0)     & = k_0                                                   \\
T(2^{i+1}) & = k_1 + 5 * (2^i)^2 + 4 * T (2^i)                       \\
           & \Rightarrow                                             \\
T          & \in O ( n \mapsto 4 ^ {lg_2 n }) = O ( n \mapsto n^2 )
           && \text{According to \ref{thm1}}
\end{align*}

The last equality is due to $2^{2^{lg_2 n}} = 2^{{lg_2 n}^2} = n^2$. \QED

## Question c

I would simply change the algorithm to move every pixel directly into place:

    rotate(A of size n by m):
      for P = (x , y) in A
        move P to (n - y , x)

This algorithm also runs in $O(n \mapsto n^2)$ but is simpler and works for any
size array.

And since my solution has to be a *modification* of the original algorithm,
this is how you modify it: 1) Delete it 2) write this.
