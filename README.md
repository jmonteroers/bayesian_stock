# bayesian_stock

A Python package to help you manage your stock using Bayesian statistics.
Input should be provided as two numpy arrays, available_product and product_left, of the same length. The main
contribution of this package is to take into account the widespread censorship in actual sales, i.e. that consumers can't
buy more than what is provided to them, to provide more precise estimation.

This package plans to integrate with scipy distribution functions, to allow you to specify any initial distribution for the
parameters and for the distribution of the stock.

## Interface
```
get_parameter_distribution(
    available_product,
    product_left,
    consumption_distribution,
    initial_distributions,
    resolution
)
```
Returns a posterior distribution over the parameter space, taking into account truncation

```
find_optimal_stock(
    price,
    cost,
    cost_throw_away,
    consumption_distribution,
    parameter_distribution,
)
```
Find optimal purchase that maximises expected benefits, taking into account costs of throwing away products.

## The Maths behind it

First, we need to distinguish between desired consumption and actual consumption, which I will call $\hat{C}$ and $C$
respectively. The difference between the two is that, while the latter is the amount that consumers wish to purchase,
the latter is the actual amount purchased at a store.

There is a close relationship between desired and actual consumption. Given the distribution of $\hat{C}$ and the
available stock for a period t, the distribution of $C$ will equal $f_{\beta}(\hat{C}=c)$ if $c < t$, and
$1 - F_{\beta}(\hat{C} = c)$ if c = t.

We can take this fact into account when computing the posterior distribution of the
parameters using Bayes formula:

$$
g(\beta|c_1, ..., c_n, t_1, ..., t_n) = N g(\beta) \prod{f_{\beta, T}(C = c)}
$$
where N is just a normalising parameter to ensure that all probabilities sum up to one. $g(\beta)$ is the prior distribution of the parameters.

### Making decisions with a distribution for the parameters

Knowing $g(\beta)$ is helpful to make decisions, since it allows us to obtain the non-conditional distribution of the desired consumption:
$$
f(\hat{C}=c) = \int_{\beta} g(\beta) f_{\beta}(\hat{C}=c)
$$

What is the decision function ($\Pi$) we want to optimise?
$$
\Pi = E_C[\pi] = E_C[Price*C - Cost*S - Throw(S-C)I(S > C)]
$$
where the decision parameter is S, set by the firm. The algorithm to determine the maximum over this distribution needs to be defined.
