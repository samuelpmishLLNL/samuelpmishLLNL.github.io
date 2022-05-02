---
layout: page
title: "Derivatives of Symmetric Matrix Functions"
image: ../../images/matrix_function_derivatives.png
description: blah blah
---


## Derivatives of Symmetric Matrix Functions

Matrix functions generalize the scalar functions we know and love (e.g. sine) to work on matrices by considering a Taylor series of the scalar function, and replacing the scalar argument by a (square) matrix instead. For example, one definition of (scalar) exponential function is:

$$
\exp(x) = 1 \; +\; x \;+ \;\frac{x^2}{2} \;+ \;\frac{x^3}{6} \; + \; \; ...
$$

the matrix version of the exponential function looks pretty much the same,

$$
\exp(\bold{A}) = \bold{1} \; +\; \bold{A} \;+ \;\frac{1}{2} \; \bold{A}^2 \;+ \;\frac{1}{6} \; \bold{A}^3 \; + \; \; ...
$$

These kind of calculations show up in quite a few different applications:

- matrix logarithm / exponential: differential equations, computer graphics
- matrix sign / square root: polar and singular value decompositions
- matrix inverse: everywhere

Although the series definition of matrix functions is useful in proofs, it is not very useful for carrying out calculations, since it can take a lot of terms in the series to get an accurate approximation. 

However, when $\bold{A}$ is symmetric, there is a very nice way to evaluate matrix functions that involves taking advantage of the spectral factorization (using a 3x3 matrix as an example):
$$
\bold{A} = \bold{Q}^\top \bold{\Lambda} \; \bold{Q} = \begin{bmatrix}
q_{11} & q_{21} & q_{31} \\ q_{12} & q_{22} & q_{32} \\ q_{13} & q_{23} & q_{33}
\end{bmatrix}\;\begin{bmatrix}
\lambda_1 & 0 & 0 \\ 0 & \lambda_2 & 0 \\ 0 & 0 & \lambda_3
\end{bmatrix}\;\begin{bmatrix}
q_{11} & q_{12} & q_{13} \\ q_{21} & q_{22} & q_{23} \\ q_{31} & q_{32} & q_{33}
\end{bmatrix}
$$
where $\lambda_i, \bold{q}_i$ are the eigenvalues and (orthogonal) eigenvectors of $\bold{A}$. To understand how it helps to express $\bold{A}$ like this, let's see what happens if we substitute the spectral decomposition in to the series expansion of $\exp$ from earlier:
$$
\begin{align}
\exp(\bold{A}) &:= \bold{1} \; +\; \bold{A} \;+ \;\frac{1}{2} \; \bold{A}^2 \;+ \;\frac{1}{6} \; \bold{A}^3 \; + \; \; ... \\ \\
&= \bold{1} \; +\; \bold{Q}^\top \bold{\Lambda} \; \bold{Q} \; + \;\frac{1}{2} \; \bold{Q}^\top \bold{\Lambda} \; \cancel{\bold{Q} \; \bold{Q}^\top} \bold{\Lambda} \; \bold{Q} \;+ \;\frac{1}{6} \; \bold{Q}^\top \bold{\Lambda} \; \cancel{\bold{Q} \; \bold{Q}^\top} \bold{\Lambda} \; \cancel{\bold{Q} \; \bold{Q}^\top} \bold{\Lambda} \; \bold{Q} \; + \; \; ... \\ \\
&= \bold{1} \; +\; \bold{Q}^\top \bold{\Lambda} \; \bold{Q} \; + \;\frac{1}{2} \; \bold{Q}^\top \bold{\Lambda} \;  \bold{\Lambda} \; \bold{Q} \;+ \;\frac{1}{6} \; \bold{Q}^\top \bold{\Lambda} \; \bold{\Lambda} \; \bold{\Lambda} \; \bold{Q} \; + \; \; ... \\ \\
&= \bold{Q}^\top \big(\bold{1} \; +\;  \bold{\Lambda} \; + \;\frac{1}{2} \;  \bold{\Lambda} \;  \bold{\Lambda}  \;+ \;\frac{1}{6} \; \bold{\Lambda} \; \bold{\Lambda} \; \bold{\Lambda}  \; + \; \; ... \big) \; \bold{Q} \\ \\
\Aboxed{\exp(\bold{A}) &= \bold{Q}^\top \exp(\bold{\Lambda}) \; \bold{Q}}
\end{align}
$$
So, all of the $\bold{Q} \; \bold{Q}^\top$ products vanish (due to orthogonality), and matrix function ends up only acting on $\bold{\Lambda}$-- which is much easier to evaluate, since it is trivial to evaluate any matrix function on diagonal matrices:
$$
f(\bold{\Lambda}) = \begin{bmatrix}
f(\lambda_1) & 0 & 0 \\ 0 & f(\lambda_2) & 0 \\ 0 & 0 & f(\lambda_3)
\end{bmatrix}
$$
Great, so now we have a good way to evaluate matrix functions: first, we compute the spectral decomposition and then just apply that function to the eigenvalues, and reassemble. However, one of the tricky parts of using matrix functions is that if you need to solve nonlinear equations that use them, then you'll likely be using an algorithm (e.g. Newton's method) that requires derivatives of these matrix functions. 

To figure out how to do this, let's imagine that $\bold{A}$ isn't constant any more, but a function of time:
$$
\bold{A}(t) := \bold{A}_0 + \bold{\dot{A}} \; t
$$
Now, $f(\bold{A}(t))$ is also indirectly a function of time, so its time derivative can be 
$$
\frac{d f(\bold{A}(t))}{dt} = \frac{\partial f(\bold{A})}{\partial \bold{A}_{ij}} \; \bold{\dot{A}}_{ij}
$$
the term we're interested in is $\frac{\partial f(\bold{A})}{\partial \bold{A}_{ij}}$, the "gradient" of the matrix function with respect to each entry of $\bold{A}$. If we knew that, then we could just carry out the inner product with $\bold{\dot{A}}$ (i.e. the  derivative in the "direction" of $\bold{\dot{A}}$). One way to do this is to expand 

