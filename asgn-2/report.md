---
author: Frederik Hanghøj Iversen
title: Algorithms - Assignment 2
date: September 12th 2017
...

\newcommand*{\defeq}{\mathrel{\vcenter{\baselineskip0.5ex \lineskiplimit0pt
                     \hbox{\scriptsize.}\hbox{\scriptsize.}}}%
                     =}

# Problem 1 - Shooting Balloons

## Question a - Algorithm and analysis

### Reformulating the problem

I first want to make the observation that we can transform the problem into an
equivalent one that does not involve so much trigonometry. For each balloon we
can calculate the angle of the two lines that are tangental with the sides of
the balloon. These two angles gives us an interval. We must shoot the laser
within this interval to hit that balloon. (Note that if the balloon contains
origo we will just say that the interval is $\left] 0, \tau \right[$).

The problem states that there is *a* place where we can shoot the laser without
hitting a balloon. I will assume $0$ is such a direction. This is an equivalent
problem since we can just rotate the problem such that this direction is
intervalline with the $x$-axis.

To summize; the input, $\mathbb{X}$, can be represented as an array of pairs:

$$
\mathbb{X} \defeq \Set*{\left( a, b \right)}{a\ , b \in \left]0, \tau\right[ \ a \leq b}
$$

### Pseudo-code implementation

Now, the algorithm goes like this:

    laser(X):
      sort X by the start-points
      let R = the empty set
          i = 0
      while i < len(X)
        let ( a , b ) = X[i]
        let m = b
        do
          i++
          if i == len(X) break
          let ( a , b ) = X[i]
              m = min(m, b)
        add m to R
        while a <= m

The idea of the algorithm is that we fix a balloon. Since we must shoot this
balloon down we must shoot a laser in the direction between either side of the
balloon. Panning from one side, if we encounter a balloon we know that we can
shoot down this ballon at the same time. If we want to do this however we must
restrict outselves to shooting between the beginning of this ballon and
whichever balloon ends first, the original balloon or the one that we are now
including.

### Complexity

The complexity of the algorithm is $O(n \mapsto n \cdot \lg n )$. We can sort
the input in $O(n \mapsto n \cdot \lg n)$. The complexity of the outer
while-loop is $O(n \mapsto n)$ since `i++` can be executed at most `len(X)`
times (the condition `i == len(X)` is checked between each invocation and we the
loop is escaped when this condition holds).

It was difiicult for me to give a more formal proof of this. I tried proving it
by saying that the inner loop is executed at most $n-i$ times but this is *not*
a sufficient proof. The proof would need to talk about the current value of $i$
at a given point. I don't feel I have the proper tools to show this, however.

### Optimality of algorithm

I will prove this by induction on the set of ballons $\mathbb{X}$. For
$\mathbb{X} = \{ \}$ it is obvious that we don't need to shoot
the laser. And so the correct result is the empty set, this is exactly what our
algorithm returns (for `len(X)=0` the while condition is false and `R` is the
empty set).

Assuming we have an optimal $R$ solution for $\mathbb{X}$ we can obtain an
optimal solution for $\{(a,b)\} \cup \mathbb{X}$ like so:

\begin{align*}
\mathit{span}(R, \mathbb{X}) \defeq \bigcap_{x \in \mathbb{X}, r \in R , r \in x} x
\end{align*}

For each group of balloons shot down with a given laser-shot $r$ this formula
calculates the intersection of all those balloons. So for any given laser shot
it would shoot down the exact same balloons if it is anywhere in this
intersection and hence changing each individual $r$ would not change the overall
solution.

\begin{align*}
\mathit{OPT}( \{\} ) & = \{ \} \\
\mathit{OPT}( \{(a, b)\} \cup \mathbb{X}) & =
\begin{cases} 
      \mathit{OPT}( \mathit{rem}( \mathbb{X} , r ) )
               & \exists r \in \mathit{span}(R, \mathbb{X}) \cap ( a, b ) \\
      R \cup \{a\} & \text{otherwise}
\end{cases}
\end{align*}

This formula expresses the fact that when calculating the optimal solution at
the next step we can assume that we have a solution for the problem of one size
smaller. If the current balloon that we are considering $(a,b)$ intersects with
the $\mathit{span}$ of the previous solution then we can substitute the laser
shot for any point in this intersection. If it does not, we must shoot a laser
at any point in the direction of this new balloon (here we've arbitrarily chosen
the start-point).

The algorithm given above does the same thing. For each group of balloons it
keeps track of the minimum end-point seen so far (`m`) and if the new ballon we
are considering has a start-point after this then we must shoot a laser at this
minimum and keep going from this new balloon.


## Question b - Dispensing with the special condition

If we want to dispense with the special condition that there is a line that does
not shoot down any ballons the problem arrises that there is now no longer a
natural total ordering on the start- and endpoints of the ballons (balloons in
the beginning of the circle may intersect with the balloons at the end of the
circle). However, we can arrive at a solution that gives the optimal answer in
some cases but is at most 1 worse than the optimal solution.

This solution simply checks if there is a balloon at $0$ (in terms of the
reformulated problem) and if there is a balloon here we shoot the laser in that
direction, remove all balloons this laser hits and continue as before).

This solution works because we now know there is not balloon at $0$ and hence we
know how to calculate an optimal solution - we already used one extra shot, but
this is within the allowed extra amounts of shots we could use.

# Problem 2

This is a special case of the knapsack problem where all the values are
restriced to $\{0,1\}$.

The optimization function is:

\begin{align*}
\mathit{OPT}(j, 0) & \defeq 0 \\
\mathit{OPT}(0, w) & \defeq 0 \\
\mathit{OPT}(j + 1, w) & \defeq \mathit{max}
  \{ \mathit{OPT}(j , w )
  , x_{j+1} + \mathit{OPT}(j-1, w - w_{j+1})
  \}
\end{align*}

And the values of this function can be calculated by iteratively calculating a
table corresponding to it's values:

    knapsack(W, V, Cap):
      let R = a matrix of dimension len(W) by Cap
      for i = 0 to len(W):
        for j = 0 to Cap:
          if i == 0:
            R[i][j] = 0
          else if j == 0:
            R[i][j] = 0
          else:
            if Cap - W[i] < 0:
              prev = 0
            else:
              prev = V[i] + R[i-1][Cap - W[i]]
            R[i][j] = max(R[i-1][j], prev)
      return R

This algorithm simply calculates the table for $OPT$. This is done in an order
that ensures that all previous values are calculated. In our case all the
weights `V` are $1$. (I've just kept the input here because it's such a
canonical algorithm, and making it more general does not impact the complexity
of the algorithm).

Now in our formulation of the problem we need to give the exact containers that
are included in the shipment. This can be done by doing "backtracking" on the
output array:

    container(W, Cap):
      let R = knapsack(W, [1..1] of lenght len(W), Cap)
      return backtrack(R, W, len(W), Cap)
      
    backtrack(R, W, i, j):
      let curr = R[i][j]
      if i <= 0 or j <= 0:
        return empty set
      if curr == R[i-1][j]:
        return backtrack(R, W, i-1, j)
      else if curr == R[i-1][j-W[i]]:
        return ( add i to backtrack(R, W, i-1, j-W[i])

The complexity of this algorithm is $O(n , W \mapsto n \cdot W)$ where $n$ is
the amount of containers and $W$ is the maximum capacity of the ship. If we can
assume that $W$ is bounded then we have a linear algorithm.

Then there is an argument that has been presented during the lectures a few
times relating the "size" of the input to the running time. The observation
being that the size of the input $W$ is on the order of $\lg W$ and thus it's
really an exponential complexity. I think this idea of the size is a bit fluffy
since the complexity expressed in terms of $W$ is exactly as given. Introducing
this concept of the "size" of an input makes it needlessly more complex in my
opinion. The amount of containers $n$ of course is not directly an input to the
algorithm but a property of the array passed to it so to be even more precise
one should maybe express the complexity as $O(\mathbb{X} , W \mapsto
\mathit{len}(\mathbb{X}) \cdot W)$ for an appropriate definition of
$\mathit{len}$ (where $\mathbb{X}$ is the actual input to the algorithm
representing the container weights.

## Analysis

The optimality of the algorithm follows inductively on $\mathit{OPT}$. For
either $w=0$ or $j=0$ the optimal solution is to take no containers for a total
value of $0$. For $j+1$ we consider not including the current container. In
which case the answer is $\mathit{OPT}(j , w )$ which per the induction
hypothesis is optimal the optimal solution for $j$. The other case is including
it in which case we the total value is given by $x_{j+1} + \mathit{OPT}(j, w -
w_{j+1})$. Again $\mathit{OPT}(j-1, w - w_{j+1})$ is the optimal solution for
$j,w-w_{j+1}$. The optimal solution in the new case is of course the maximum of these two.
