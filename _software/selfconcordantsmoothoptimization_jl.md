---
layout: page
title: SelfConcordantSmoothOptimization.jl
description: Julia implementation of self-concordant smoothing algorithms for convex composite optimization 
img: 
redirect: https://github.com/adeyemiadeoye/SelfConcordantSmoothOptimization.jl
github: https://github.com/adeyemiadeoye/SelfConcordantSmoothOptimization.jl
importance: 1
category: optimization
# related_publications: adeoye2023self
toc:
  sidebar: left
---

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/prox-SCORE.svg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<a style="text-decoration:underline" href="https://github.com/adeyemiadeoye/SelfConcordantSmoothOptimization.jl" target="_blank">SelfConcordantSmoothOptimization.jl<i class="fab fa-github gh-icon"></i></a> is a Julia package that implements the self-concordant regularization (SCORE) technique for nonsmooth convex optimization introduced in <a style="text-decoration:none" href="https://arxiv.org/abs/2309.01781" target="_blank">this paper</a>. In particular, `SelfConcordantSmoothOptimization.jl` considers problems of the form

$$
\min f(x) + g(x)
$$

where $$f\colon \mathbb{R}^n \to \mathbb{R}$$ is smooth and convex, and $$g\colon \mathbb{R}^n \to \mathbb{R}$$, which may be nonsmooth, is proper, closed and convex. The smooth part $$f$$ defines the problem's objective function, such as quantifying a data-misfit, while the nonsmooth part $$g$$ imposes certain properties, such as sparsity, on the decision variable $$x$$. Please see [Implementation details and recommendations](#implementation-details-and-recommendations) for functions that are currently supported for each implemented algorithm.

## Installation
Install the package via Julia's `REPL` with

```julia
using Pkg

Pkg.add("SelfConcordantSmoothOptimization")
```

## Usage example
#### A simple sparse logistic regression problem

```julia
# Load the package
using SelfConcordantSmoothOptimization

using Random, Distributions, SparseArrays

# Generate a random data
Random.seed!(1234)
n, m = 100, 50;
A = Matrix(sprandn(n, m, 0.01));
y_prob = 1 ./ (1 .+ exp.(-A * zeros(m)));
y = rand.(Bernoulli.(y_prob));
unique_y = unique(y); 
y = map(x -> x==unique_y[1] ? -1 : 1, y);
x0 = randn(m);

# Define objective function and choose problem parameters
f(A, y, x) = 1/m*sum(log.(1 .+ exp.(-y .* (A*x))));

# choose problem parameters
reg_name = "l1";
λ = 0.4;
μ = 1.0;
hμ = PHuberSmootherL1L2(μ);

# set problem
model = Problem(A, y, x0, f, λ);

# Choose method and run the solver
method = ProxNSCORE();
solution = iterate!(method, model, reg_name, hμ; max_iter=100, x_tol=1e-6, f_tol=1e-6);
# get the solution x
solution.x
```

To use the `ProxGGNSCORE` algorithm, a model output function $$\mathcal{M}(A,x)$$ is required
(Note that it is essential to define the function f in two ways to use `ProxGGNSCORE`; Julia will decide which one to use at any instance -- thanks to the multiple dispatch feature):

```julia
# f as defined above
f(A, y, x) = 1/m*sum(log.(1 .+ exp.(-y .* (A*x))));
# f as a function of y and yhat
f(y, yhat) = -1/m*sum(y .* log.(yhat) .+ (1 .- y) .* log.(1 .- yhat))
# where yhat = Mfunc(A, x) is defined by the model output function
Mfunc(A, x) = 1 ./ (1 .+ exp.(-A*x))
# set problem
model = Problem(A, y, x0, f, λ; out_fn=Mfunc);

# Choose method and run the solver
method = ProxGGNSCORE();
solution = iterate!(method, model, reg_name, hμ; max_iter=100, x_tol=1e-6, f_tol=1e-6);
# get the solution x
solution.x
```

By default, this package computes derivatives using [`ForwardDiff.jl`](https://github.com/JuliaDiff/ForwardDiff.jl). But users can supply functions that compute the derivates involved in the algorithms. This has at least two benefits:
1. Faster computations
2. Avoid the need to define $$f$$ twice when using `ProxGGNSCORE`

In the example above:

```julia
f(A, y, x) = 1/m*sum(log.(1 .+ exp.(-y .* (A*x))));

S(x) = exp.(-y .* (A*x));
# gradient of f wrt x:
function grad_fx(x)
    Sx = S(x)
    return -A' * (y .* (Sx ./ (1 .+ Sx))) / m
end;
# Hessian of f wrt x:
function hess_fx(x)
    Sx = 1 ./ (1 .+ S(x))
    W = Diagonal(Sx .* (1 .- Sx))
    hess = A' * W * A
    return hess / m
end;

# The following are used by ProxGGNSCORE
# Jacobian of yhat wrt x:
jac_yx(yhat) = vec(yhat .* (1 .- yhat)) .* A;
# gradient of \ell wrt yhat:
grad_fy(yhat) = (-y ./ yhat .+ (1 .- y) ./ (1 .- yhat))/m;
# Hessian of \ell wrt yhat:
hess_fy(yhat) = Diagonal((y ./ yhat.^2 + (1 .- y) ./ (1 .- yhat).^2)/m);
```

```julia
# Now (for ProxNSCORE):
model_n = Problem(A, y, x0, f, λ; grad_fx=grad_fx, hess_fx=hess_fx);
method_n = ProxNSCORE();
sol_n = iterate!(method_n, model_n, reg_name, hμ; max_iter=100, x_tol=1e-6, f_tol=1e-6);
sol_n.x
```

```julia
# And for ProxGGNSCORE (does not require hess_fx):
model_ggn = Problem(A, y, x0, f, λ; out_fn=Mfunc, grad_fx=grad_fx, jac_yx=jac_yx, grad_fy=grad_fy, hess_fy=hess_fy);
method_ggn = ProxGGNSCORE();
sol_ggn = iterate!(method_ggn, model_ggn, reg_name, hμ; max_iter=100, x_tol=1e-6, f_tol=1e-6);
sol_ggn.x
```
(For sparse group lasso example, see example from paper in <a style="text-decoration:underline" href="https://github.com/adeyemiadeoye/SelfConcordantSmoothOptimization.jl/tree/main/examples" target="_blank">`/examples`<i class="fab fa-github gh-icon"></i></a>).

## Implementation details and recommendations
Below is a summary of functions $$f$$ supported by the algorithms implemented in the package:

| Algorithm      	| Supported $$f$$                                                                                                                                                                                                                                 |
|----------------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| `ProxNSCORE`   	| Any twice continuously differentiable function.                                                                                                                                                                                        		|
| `ProxGGNSCORE` 	| Any twice continuously differentiable function that can be expressed in the form $$f(x) =  \sum\limits_{i=1}^{m}\ell(y_i,\hat{y}_i)$$ where $$\ell$$ is a loss function that measures a data-misfit.<br>Requires a model $$\mathcal{M}(A,x)$$ that computes the predictions $$\hat{y}_i$$. 		|
| `ProxQNSCORE`  	| Any twice continuously differentiable function (currently not recommended).<br>Currently not recommended.                                                                                                                                                                                       		|

### Optional arguments

| Arg      	| Description & usage                                                                                                                                                                                                                                 |
|----------------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| `ss_type`   	| Value `1` and with a given `L` in `Problem` sets $$\alpha=\min\{1/L, 1\}$$.<br> Value `1` without setting `L` in `Problem` sets $$\alpha$$ to the value in `iterate!` (or a default value $0.5$ if not set).<br> Value `2` uses the "inverse" of Barzilai-Borwein method to set $$\alpha$$ (Not recommended).<br> Value `3` uses a line-search method to choose $\mathrm{\alpha}$.<br> Default value: `1`.<br> e.g. `method = ProxGGNSCORE(ss_type=1)`<br>                                                                                                                                                                                       		|
| `use_prox` 	| Value `true` uses the proximal scheme as described in the paper.<br> Value `false` skips the proximal step and takes only the associated Newton-type/gradient-based step.<br> Default value: `true`.<br> e.g. `method = ProxGGNSCORE(use_prox=true)`<br>                                                                                                                                                                                       		|


As the package name and description imply, the implemented algorithms use a generalized self-concordant smooth approximation $$g_s$$ of $$g$$ in their procedures. The algorithms do this for specific regularization functions that are specified by `reg_name` that takes a string value in the `iterate!` function. We summarize below currently implemented regularization functions, as well as the corresponding smoothing functions $$g_s$$.

| `reg_name` value 	| Implemented $$g_s$$ function(s)                                                                                                                                              	| Remark(s)                                                                                           		|
|------------------	|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|-----------------------------------------------------------------------------------------------------	|
| `"l1"`           	| `PHuberSmootherL1L2(μ)`<br>`OsBaSmootherL1L2(μ)`	| $$\mu>0$$                                                                                             	|
| `"l2"`           	| `PHuberSmootherL1L2(μ)`<br>`OsBaSmootherL1L2(μ)`                                                                       	| $$\mu>0$$                                                                                             	|
| `"gl"`       	| `PHuberSmootherGL(μ, model)`                                                                          	| For sparse group lasso regularizer <br> $$\mu>0$$ 	|
| `"indbox"`       	| `PHuberSmootherIndBox(lb,ub,μ)`                                                                          	| `lb`: lower bound in the box constraints <br> `ub`: upper bound in the box constraints <br> $$\mu>0$$ 	|

- `PHuberSmootherL1L2`, `PHuberSmootherGL`, `ExponentialSmootherIndBox` are recommended.

For more details and insights on the approach implemented in this package, please see the associated paper in [Citing](#citing) below.

## Citing
If you use `SelfConcordantSmoothOptimization.jl` in your work, particularly the algorithms listed above, we kindly request that you cite the following paper:
{% details Click here for BibTeX %}
```
@article{adeoye2023self,
  title={Self-concordant Smoothing for Convex Composite Optimization},
  author={Adeoye, Adeyemi D and Bemporad, Alberto},
  journal={arXiv preprint arXiv:2309.01781},
  year={2023}
}
```
{% enddetails %}
<div class="publications">
{% bibliography -f journal -q @*[key^=adeoye2023self]* %}
{% bibliography -f conference -q @*[key^=adeoye2023self]* %}
{% bibliography -f preprint -q @*[key^=adeoye2023self]* %}
{% bibliography -f technical -q @*[key^=adeoye2023self]* %}
</div>

## Contributing
Please use the [Github issue tracker](https://github.com/adeyemiadeoye/SelfConcordantSmoothOptimization.jl/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc) for reporting any issues. All types of issues are welcome including bug reports, feature requests, implementation for a specific research problem, etc.
