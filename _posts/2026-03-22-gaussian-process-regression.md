---
layout: post
title: "Gaussian Process Regression Part I: Practical Implementation of Uncertainty Quantification"
date: 2026-03-22
lang: en
---


This post collects some practical notes on implementing uncertainty quantification (UQ) in Gaussian Process Regression (GPR).

I did some work back in my post doc days with approximate kernels that I never published, so I am writing a few blog posts to share the work.
I'm starting with a practical intro to GPR+UQ and then I will move on to approximate kernels.

In this first part, I will just cover the basic GPR equations and how to implement them.
The next posts will cover:
- GPR+UQ with derivatives (e.g. for molecular forces)
- Practical examples of GPR+UQ in chemistry
- UQ for approximate kernels

---
### Introductory notes and notation
In textbooks, the predictive mean and variance are usually written as

$$
\hat{y}_* = \mathbf{k}_*^\top \left(\mathbf{K} + \lambda \mathbf{I}\right)^{-1} \mathbf{y}
$$

and

$$
\hat{\sigma}_*^2 = \mathpzc{k}(\mathbf{x}_*, \mathbf{x}_*) - \mathbf{k}_*^\top \left(\mathbf{K} + \lambda \mathbf{I}\right)^{-1} \mathbf{k}_*.
$$

where the first equation is very analogous (basically identical) to kernel-ridge regression (KRR).

Here I use a hat, $\hat{\cdot}$, to denote predicted quantities, and an asterisk, $*$, to denote a test point. $\mathbf{x}$ is the feature vector or representation for a training/test point.

In this notation:

- $\mathbf{K}$ is the $n \times n$ kernel matrix over the training data, with entries
  $$
  K_{ij} = \mathpzc{k}(\mathbf{x}_i, \mathbf{x}_j),
  $$
- <span>$\mathbf{k}_*$</span> is the $n \times 1$ covariance vector between the test point <span>$\mathbf{x}_*$</span> and all training points,
  $$
  \mathbf{k}_* =
  \begin{bmatrix}
  \mathpzc{k}(\mathbf{x}_1, \mathbf{x}_*) \\
  \vdots \\
  \mathpzc{k}(\mathbf{x}_n, \mathbf{x}_*)
  \end{bmatrix}.
  $$

These equations are correct, but they are slightly misleading from an implementation point of view. For example, you would never explicitly compute the inverse of $\left(\mathbf{K} + \lambda \mathbf{I}\right)^{-1}$. Plus a few other differences that we'll get to later/


### Training the model
Our task here is to solve for the regression coefficients $\alpha$:

$$
 \alpha = \mathbf{y}(\mathbf{K} + \lambda \mathbf{I})^{-1}.
$$

In practice, you wwill never form the inverse explicitly. 
Instead, the most efficient approach is to do a Cholesky factorization the regularized kernel matrix and solve linear systems:

$$
\mathbf{K} + \lambda \mathbf{I} = LL^\top,
$$

with

$$
L = \operatorname{Cholesky}(\mathbf{K} + \lambda \mathbf{I}).
$$

where $L$ is lower triangular (L is for lower).

This is both more numerically stable and more efficient than forming the inverse explicitly. It also avoids storing the inverse matrix in memory. Instead of storing an $n \times n$ dense inverse, you only keep the lower-triangular Cholesky factor, which requires $\tfrac{n(n+1)}{2}$ entries if you work with packed storage matrices.

In LAPACK, this is typically done with `dpotrf`. In Python, this corresponds to `numpy.linalg.cholesky()` or `scipy.linalg.cho_factor()`.




Using the Cholesky factorization, this is equivalent to solving two triangular systems:

$$
L \mathbf{z} = \mathbf{y}
$$

followed by

$$
L^\top \alpha = \mathbf{z}.
$$

In LAPACK, this is usually handled by `dpotrs` which actually does both solves in one go. In SciPy, the corresponding interface is `scipy.linalg.cho_solve()`.

Heres a very minimal example using `qmllib` in Python:

```python
import numpy as np
from scipy.linalg import cho_factor, cho_solve
from qmllib.kernels import gaussian_kernel 

K = gaussian_kernel(X_train, X_train, 2.0)

# Add regularization / observation noise to the diagonal
K[np.diag_indices_from(K)] += lambda_

# Cholesky factorization - save this one for later
c, lower = cho_factor(K, lower=True)

# Solve (K + lambda I) alpha = y
alpha = cho_solve((c, lower), y)
```

#### A useful shortcut for the training residual

Heres a small trick I find very useful in practice: 
After training, you often want to compute the training residuals, which are the differences between the training labels and the predictions on the training set.

Looking at the equations you might think you would need the same kernel matrix as you just used to to solve the regression coefficients, but actually you can compute the training residual _without_ that big kernel.

Lets define the residual error (difference between true labels and predictions) as:

$$
\hat{\mathbf{r}} = \mathbf{y} - \hat{\mathbf{y}}.
$$

Since the predictions on the training set are

$$
\hat{\mathbf{y}} = \mathbf{K}\alpha,
$$

we have

$$
\hat{\mathbf{r}} = \mathbf{y} - \mathbf{K}\alpha.
$$

Now use the fact that the fitted coefficients satisfy exactly that

$$
(\mathbf{K} + \lambda \mathbf{I})\alpha = \mathbf{y}.
$$

Substituting this into the residual expression gives

$$
\hat{\mathbf{r}} = (\mathbf{K} + \lambda \mathbf{I})\alpha - \mathbf{K}\alpha
$$

and, since the $\mathbf{K}\alpha$ terms cancel out, therefore

$$
\hat{\mathbf{r}} = \lambda \alpha.
$$

Once you have computed $\alpha$, you can discard $\mathbf{K}$ and get the training residual for free.

### Predicting at test time
Ok, now we are ready to do inference on new test points.

For a test point <span>$\mathbf{x}_*$</span>, first compute the covariance vector

$$
\mathbf{k}_* =
\begin{bmatrix}
\mathpzc{k}(\mathbf{x}_1, \mathbf{x}_*) \\
\vdots \\
\mathpzc{k}(\mathbf{x}_n, \mathbf{x}_*)
\end{bmatrix}.
$$

The predictive mean is

$$
\hat{y}_* = \mathbf{k}_*^\top \alpha.
$$

Once $\alpha$ has been computed during training, this is just a dot product between the test covariance vector and the stored coefficients.

### Predictive variance

The textbook formula is

$$
\hat{\sigma}_*^2 = \mathpzc{k}(\mathbf{x}_*, \mathbf{x}_*) - \mathbf{k}_*^\top (\mathbf{K} + \lambda \mathbf{I})^{-1} \mathbf{k}_*.
$$

Again, we do not form the inverse explicitly. Instead, we use the same Cholesky factor $L$ and solve

$$
L \mathbf{v} = \mathbf{k}_*.
$$

Then

$$
\hat{\sigma}_*^2 = \mathpzc{k}(\mathbf{x}_*, \mathbf{x}_*) - \mathbf{v}^\top \mathbf{v}.
$$

This works because

$$
\mathbf{k}_*^\top (\mathbf{K} + \lambda \mathbf{I})^{-1} \mathbf{k}_*
= \mathbf{k}_*^\top (LL^\top)^{-1} \mathbf{k}_*
= \|L^{-1}\mathbf{k}_*\|^2
= \mathbf{v}^\top \mathbf{v}.
$$

-Ok, but we're not done yet! Just implemnenting the equation above will likely not give you what you are looking for

### Unit scale of the predictive variance?!

You might notice this: if the kernel is unitless, then the predictive uncertainty is also unitless, which is ~~a bit~~ totally weird.
In contrast, the predictive mean is in the same units as the training labels, and the units of the training labels seem to get absorbed into alpha.

In fact, you can see that no matter what labels you put in your $\mathbf{y}$ vector, the predictive variance formula seems to give the same variance. But we'd expect that you'd get a 1000 times larger variance if you trained on labels in millimeter instead of meter, for example.

This is because commonly, the kernel in GPR is defined to contain the signal variance $\sigma_f^2$ as a multiplicative factor, e.g. for the scaled RBF kernel: 

$$
\mathpzc{k}^{\text{scaled}}(\mathbf{x}, \mathbf{x}') = \sigma_f^2 \exp\left(-\frac{\|\mathbf{x} - \mathbf{x}'\|^2}{2l^2}\right).
$$

So GPR expects, where as in e.g. KRR, the kernel is often defined without the signal variance factor.


If you just want to work with a "normal", _unitless_ kernel, for example the Gaussian: $\mathpzc{k}^{\text{unitless}}(\mathbf{x}, \mathbf{x}') = \exp\left(-\frac{\|\mathbf{x} - \mathbf{x}'\|^2}{2l^2}\right)$, then the equations change slightly. The predictive mean is still <span>$\hat{y}_* = \mathbf{k}_*^\top \alpha$</span>, since the "missing" scaling is absorbed into $\alpha$ during training.

However, the predictive variance is missing the signal variance factor, so we need to add it back in explicitly, and the predictive variance becomes:

$$
\hat{\sigma}_*^2 = \sigma_f^2 \left(  \mathpzc{k}^{\text{unitless}}(\mathbf{x}_*, \mathbf{x}_*) - \mathbf{v}^\top \mathbf{v} \right).
$$

To further simplify things for the unitless Gaussian kernel, the self-kernel 
<span>$$\mathpzc{k}^{\text{unitless}}\left( \mathbf{x}_*, \mathbf{x}_*\right)$$</span>
is always 1, so the predictive variance simplifies to a very simple form:

$$
\hat{\sigma}_*^2 = \sigma_f^2 \left( 1 - \mathbf{v}^\top \mathbf{v} \right).
$$

Ok, so what is the value of $\sigma_f^2$? In principle, it's a hyperparameter that needs to be fitted, for example to a validation set to calibrate the predictive uncertainty. 
In practice you can use the maximum likelihood estimate of $\sigma_f^2$ from the training data, which is given by the average product of the training labels and the regression coefficients:

$$
\sigma_f^2 = \frac{1}{n} \mathbf{y}^\top \alpha.
$$

Putting it all together, here is a complete example of the variance estimation at test time:

```python
import numpy as np
from scipy.linalg import cho_factor, cho_solve, solve_triangular
from qmllib.kernels import gaussian_kernel 

# --- Prediction at a test point ---
# k_star is the n_train x 1_test covariance vector between x_* and training points
K_star = gaussian_kernel(X_train, X_test, 2.0)

# Predictive mean
y_pred = K_star @ alpha

# MLE estimate of signal variance
sigma_f_sq = y @ alpha / len(y)

# Predictive variance
v = solve_triangular(c, K_star, lower=True)
var_pred = sigma_f_sq * (np.ones(n_test) - np.sum(v**2, axis=0))
```

In my next blogpost, I will show how to do variance calculation with kernel-derivatives for molecular energy and forces with a few practical examples.


_**-Anders**_
