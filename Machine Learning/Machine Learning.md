# Machine Learning

What is learning?
>“A computer program is said to
learn from experience E
with respect to some class of tasks T
and performance measure P,
improves with experience E”  
Mitchell 1997

Knowledge in machine learning comes from:
- experience
- induction

Can only learn what is present in the data, if the dataset is not rich enough i cannot learn anything

Traditional programming:  
data + program -> output  
Machine learning approach:  
data + output -> program

Used when it is easy to solve the task for a human but it is very difficult to code a program to do the same task (e.g. tell if there is a cat or a dog in a picture)

Why machine learning?  
We need computer to make informed decisions on new unseen data where writing a program directly is difficult or unfeasible. Machine learning allows to automatically extract information from data and applying it to new data -> automating automation.

Fields:
- computer vision
- speech recognition
- finance
- biology
- video gaming
- space exploration
- ...

## Overview
There are 3 main subfields in machine learning:
- Supervised learning
  - learn the model, relationship between the input data and the desired output
- Unsupervised learning
  - learn the representation, there is no output value we want to find structure in the input data (e.g. find clusters, grouping according to some property)
- Reinforcement learning
  - learn the policy, to control, make the best decision to collect the best utililty.

We will deal mainly with supervised and reinforcement learning.

### Supervised learning
Goal: estimate an unknown model that maps known inputs to known outputs.  
Problems:
- classification, target is a discrete set of values
- regression, target value is continous
- probability estimation, target is a probability value (basically a constrined regression)  

Techniques:
- Artificial Neural Networks
- Support Vector Machines
- Decision Trees
- ...

Dataset: `D = {<x, t>}`, we want to find `f(x) = t` where:
- `f` the function we want to approximate
- `x` features, predictors, attributes given in input
- `t` target, responses, labels, represent desired output
  - discrete
  - continuous
  - probability

When to apply?
- task for which there is no human expert
- where it is too difficult to write a program to do it (remember image classification example)
- the desired function changes frequently
- each user needs a customized function(e.g. spam filtering)

#### Essence
We want to approximate the **unkwown** function `f` given the dataset `D`. The steps are:
- define a loss function `L` 
  - measures the distance between any function to the function you want to estimate, define how good or bad a candidate function is.
- define an hypothesis space `H`
  - subspace of all function in which we search the best function (e.g. only linear functions).
- optimize to find an approximate model `h`
  - search the best function in `H` that minimizes our loss function.

The size of the hypothesis space depends on the sample that we have -> the larger `H` the more samples we need.  
Also we do not have the true loss function but only the empirical loss function given by the samples that we have -> it's only an approximation of the true loss and can be (very) noisy. The more samples we have the more `L` will be close the true loss (is the true loss if we have infinite samples).  
It is not possible to learn complex models with few data -> will lead to overfitting. Choosing the right `H` is the most difficult task in supervised learning.  

NOTE: assumption of supervised learning is that the distribution of the dataset is the same of the real world data we will observe, if not we need to take this into account and basically it is like having fewer samples.

### Unsupervised learning
Goal: learning a better representation of a set of unknown inputs.  
Problems:
- compressing
- clustering

Techniques:
- K-means
- Self Organizing Maps
- Principal Component Analysis (the only one we will see)
- ...

### Reinforcement learning
Goal: learning the optimal policy to collect the maximum utility through a series of actions  
Problems:
- Markov Decision Problems
- Stochastic games  

Techniques:
- Q-learning
- SARSA
- ...

## Linear regression
Map an input space `x` to a continous target output `t`.  
Examples:
- predict stock market
- predict age of web user
- predict price of a house
- predict temperature in a building
- ...

Many problems can be approximated by linear models that can be solved analytically (can also model non-linear relations using kernels, see later). This means that i can always find the global optimum of the problem.

A linear model is a parametric method, linear in the **parameters** (not necessarily linear in the variables):  
![linear_model](assets/linear_model.png)  
- **`x`**` = (1, x1, x2, ..., xD-1)`
- `w0` is the offset

NOTE:
- parametric methods: size of `H` independently from the number of samples in the dataset.
- non parametric methods: size of `H` changes accordingly to the number of samples, see later (for examples kernel).

### Loss function
Needs to measure if a model is good or not.  
`L(t, y(`**`x`**`))`  
We do not have the true loss function, we can only define it over the samples that we have.  
Common choices:
- squared loss function, sum of all the inputs of the squared loss `(t - y(`**`x`**`))^2` for each sample  
The optimal solution is the conditional average:  `y(`**`x`**`) = E[t|`**`x`**`]`
- minkowski loss function, generalization of squared loss `(t - y(`**`x`**`))^q` where `q` is a parameter that we can adjust based on what we want to learn (from easier to harder to learn):
  - `q = 2` (squared loss), conditional average (mean)
  - `q = 1`, conditional median
  - `q → 0`, conditional mode

Since the model is linear in the parameters we can transform the input variables as we please, also using non linear function. These are called basis functions:  
![linear_model_with_basis_function](assets/linear_model_with_basis_function.png)  
where **`Φ`**`(`**`x`**`) = (1, Φ1(`**`x`**`), ..., ΦM-1(`**`x`**`)` is called *feature vector* and each component is a *feature*.  
Examples:
- polynomial
- gaussian
- sigmoidal

This is very useful because often the input variables are not linearly related to the target but there can be more complex relationships and all of these cann be modeled using linear models.

Different approaches to supervised learning:
- statistical
  - generative, approximate the joint probability `p(x, t)`
  - discriminative, model the distribution `p(t|x)`
- direct, find a regression function `y(x)` from training data

We start with the direct approach to minimize the least squares.
### Minimizing the least squares
Consider the following loss function on `N` samples.  
![sum_of_squared_errors](assets/sum_of_squared_errors.png)  
This is called Residual Sum of Squares (RSS) or Sum of Squared Errors (SSE) and it is our empirical approximation of the true loss function given the samples that we have.
We can optimize it in closed form:
- rewrite using matrix notation  
![RSS_matrix_form](assets/RSS_matrix_form.png)
- compute first and second derivative  
![RSS_derivative](assets/RSS_derivative.png)
- assuming the second derivative is a non singular matrix we can compute the parameters (gradient = 0)  
![RSS_optimal](assets/RSS_optimal.png)

The computational complexity of this method `O(NM^2 + M^3)` where `N` is the number of samples and `M` is the number of features, so it becomes unpractical for large number of features. Where is the matrix singular (and therefore i cannot compute inverse)?
- linearly dependent features
- more features than samples

NOTE: it is always positive semidefinite.

If we have too many feature we can avoid using the closed form solution and use instead the **gradient approximation**  
![RSS_gradient_approximation](assets/RSS_gradient_descent.png)  
This is guaranteed to converge if the learning rates `α` comply with the following conditions (e.g. `1/k` satisfies both conditions):  
![convergence_conditions](assets/convergence_conditions.png)  
This iterative approach has a quadratic complexity and therefore more convenient if the number of iteration is lower than the number of features.

### Maximum Likelihood (ML)
Discriminative approach where we assume that the target value is given by:  
`t = f(`**`x`**`) + ε`  
We want to approximate `f(`**`x`**`)` with `y(`**`x`**`, `**`w`**`)`

If the noise `ε` is gaussian with `0` mean and variance `σ^2` we can write the likelihood function given `N` samples  
![likelihood_function](assets/likelihood_function.png)  
Our objective now is to find the parameters that maximize the likelihood. We can compute it in two steps:
- compute log-likelihood  
![log_likelihood](assets/log_likelihood.png)
- put gradient = 0 and solve for `w`
![ML_optimal](assets/ML_optimal.png)

We can see that the result is the same as minimizing the least squares, this means that those two approaches are two equivalent ways of looking at the same problem. Note that the optimal model is independent from the variance of the noise.

We can estimate the variance of the parameters that we have found by computing the variance-covariance matrix (need to estimate `σ^2` using RSS)  
![ML_parameter_variance](assets/ML_parameter_variance.png)  
This allows to perform some feature selection. If the parameter is small and has high variance then likely it is not relevant to our estimate and can be removed. Statistical tests are used to determine what parameters can be discarded.

NOTE: can be used also in case of multiple outputs, the inverse matrix only needs to be computed once and cam be reused for each component.

#### Gauss-Markov theorem
The least square estimate of **`w`** has the smallest variance among all linear unbiased estimates.

NOTE:
- there maybe other biased solutions with smaller MSE
- unbiased estimators are not always the best approach, as we will see low bias means high variance so there are trade offs to be made.

### Overfitting and underfitting
- low order polynomial (small hypothesis space) lead to underfitting, performs bad both on data and on generalization
- high order polynomial (big hypothesis space) yield excellent performance on the training data but generalize very poorly on new data because it is a poor representation of the true function.

Increasing the size of the hypothesis space always lead to reducing the training error but the true error raises.

How can we avoid overfitting?  
It is a problem of model selection
- use more samples in the same `H`
- regularization techniques

### Regulatization techniques
- RIDGE REGRESSION  

Constraints the value of the weights by adding a penalization factor in the loss function for large values of the weights:  
`L(`**`w`**`) = Ld(`**`w`**`) + λ*Lw(`**`w`**`)`  
where:
- `Ld(`**`w`**`)` is the error on the data (e.g. RSS)
- `Lw(`**`w`**`)` is the model complexity (e.g. squared sum of the norm of the weights)
- `λ` is a parameter that we can tweak to decide how "heavy" we want to regularize (usually very small: 10^-8 to 10^-6)

The optimized weights are computed in a similar way as RSS and ML:  
![ridge_optimal](assets/ridge_optimal.png)  
The effect of this technique is to "smooth" the value of the parameters uniformely towards 0 (avoid overfitting) while still maintaining a closed form solution. I can use this approach also to test models with many features to see how they work.

NOTE: another intesting point is that the matrix is always invertible.

- LASSO REGULARIZATION

Similar to ridge but is is not linear. The loss function is  
![lasso_loss_function](assets/lasso_loss_function.png)  
where `||`**`w`**`||1` is the sum of the norms of the weights.

The disadvantage is that we no more have a closed form solution.
The advantage w.r.t. to ridge is that for some `λ` some values of the weights can be put to 0. Lasso performs feature selction and is able to yield sparse models.

### Bayesian linear regression
In contrast with the approaches seen before, adds prior information to the problem. Also it does not compute the optimal weights but yields a probability distribution for them starting from a prior distribution that encodes our prior knwoledge about the problem. After observing the data we combine what we have observed with our prior and compute our posterior distribution for the parameters.

This allow to also have the uncertainty about our decision directly encoded in our posterior.

The posterior distribution is obtained using the bayes rule:  
![bayesian_posterior](assets/bayesian_posterior.png)  
where:
- `p(`**`w`**`|D)` is the posterior probability of the parameters **`w`** given the dataset `D`
- `p(D|`**`w`**`)` is the likelihood of observing `D` given **`w`**
- `p(`**`w`**`)` is the prior distribution of the parameters
- `p(D)` is a normalization factor (marginal likelihood)

We want to obtain the most probable value for the weights given the data that we have -> Maximum A Posteriori (MAP).

NOTE: if the prior distribution does not bring any knowledge (i.e. uniform distribution) then it is equivalent to the maximum likelihood approach.

What is the advantage?  
Thanks to the prior we can avoid overfitting and there is no need to regularize, the prior act as a regularizer.

#### Gaussian prior
Using a gaussian prior is very useful in a scenario when we have sequential data (prior -> new data -> posterior -> use posterior as new prior -> new data -> new posterior -> ...) because this ensures that the posterior will also be a gaussian (so i can iterate the process indefinitely). If the prior and posterior are distributions in the same family we say that the prior is a *conjugate prior* and this allows to have a closed form solution:  
![closed_form_bayesian](assets/closed_form_bayesian.png)  
As we can see:
- **`wN`**, the mean of the new posterior, depends on the mean of the prior and the OLS solution, in particular if we choose a prior with 0 mean and variance this approaches reduces to ridge regression.
- with infinite variance (uniform distribution) it is reduced to ML.

The more data we have the less the prior becomes relevant in determining the posterior distribution (for lots of data it is usually preferred to use ML since it is cheaper to compute BUT yields a point prediction instead).

#### Posterior predictive distribution
In the gaussian case, the posterior predictive distibution is also a gaussian (the distribution of our prediction, obtained for instance by performing a weighted average of the outcome of all the models based on their posterior):  
![posterior_predictive_distibution](assets/posterior_predictive_distribution.png)  
And the uncertainty is given by  
![bayesian_prediction_Uncertainty](assets/bayesian_prediction_Uncertainty.png)  
where:
- `σ^2` is the noise in the target value
- the second term is the uncertainty associated to our parameters value

The good news is that in the limit of infinite samples the second term goes to 0, so the variance of our prediction depends only on the noise in the target value.

#### Challenges
- specify a suitable prior distribution and suitable model
- computational cost for the posterior distribution if we do not have gaussian distribution (no closed form, approximations like gaussian approximation or monte carlo integration)

## Linear classification
Goal: assign an input `x` into one of the `k` discrete classes `Ck`. Typically each input is assigned to only one class.

To achieve this the input space is divided into regions whose boundaries are called *decision boundaries* or *decision surfaces*. For classification we would like to use a non-linear function:  
![non_linear_classification_model](assets/non_linear_classification_model.png)  
We call this **generalized linear model** because the expression of the decision surfaces remains linear (`y(`**`x`**`, `**`w`**`) = constant` that solving for x yields the hyperplane equation of the decision surface).

#### Notation
- In 2 class problems we assume `t ∈ {0, 1}` where `t = 0` represents the negative class and `t = 1` represent the positive class (we can interprete `t` as the probability of the positive class)
- if there are `K` classes we can use:
  - one-hot encoding
  - probability vector (extension of one-hot)

### Approaches
Similarly to regression we can have different approaches:
- probabilistic, models the conditional distribution `p(Ck|x)` and uses it to make optimal decision. We can distinguish in:
  - discriminative, models it directly
  - generative, models the conditional densities `p(`**`x`**`|Ck)`and then computes the conditional distribution using bayes rule (not treated in this course)
- discriminant function (direct approach), build a function that directly maps input to classes, does not output a probability value of an input belonging to other classes

### Discriminant function
We start with the 2 classes case using a linear model(spoiler alert: not a good idea to use a linear model):
- we pick a linear model (e.g. the same as logistic regression)
- explicitate the boundary
  - assign to `C1` if `y(`**`x`**`) >= 0`
  - assign to `C2` otherwise
- if we compute the parameters of our model we obtain that it is orthogonal to the decision surface.

What if i have`K` classes?
- ONE-VERSUS-THE-REST
  - `K-1` classifiers, each classifies if it belongs to one class or all the others. Can lead to ambiguous points
- ONE-VERSUS-ONE
   - `K(K-1) / 2` classifiers, one for each pair of classes, leads to similar problems of ambiguous

How do we solve this ambiguitly?  
Assign the class based on the largest value(remember that the output represents the probability). the advantage in doing this is that the decision boundaries are singly connected and convex.

#### Least squares for classification
The idea is to use linear regression for each target class using K models.

PROBLEMS:
- outliers: very sensitive to outliers because it will try to "bend" the linear model towards those outliers to reduce the squared error, moving the decision surface and causing misclassification.
- non-gaussian distribution in the target noise, since linear regression

#### Fixed basis function
As seen for regression, we do not have to find linear boundaries in the input space but we can define a feature space in which the boundaries are linear, obtained by transforming the input using vectors of basis functions `Φ(`**`x`**`)`

#### Perceptron
Oldest approach to classification, it is a two class model  
![perceptron](assets/perceptron.png)  
where +1 correspond to class `C1` and -1 to `C2`.  
The idea is to find the separating plane by minimizing the sum of the distance of the misclassified points from the decision boundary.  
The loss function to be minimized is the following  
![perceptron_loss_function](assets/perceptron_loss_function.png)  
where:
- `M` is the set of misclassified points
- 0 error is given to correctly classified points

This can be optimized using gradient descent  
![perceptron_gradient_descent](assets/perceptron_gradient_descent.png)  
The value of the learning rate `α` is not relevant since the separating plane does not change with the value of `w` (is a vector orthogonal to the plane), so we can put `α = 1` (the opposite from regression where `α` had to respect certain conditions). 

The alghorithm iterates over all the points until all the points are correctly classified. It is guaranteed to converge if the points are linearly separable (but if they aren't it never stops, it is semidecidable -> cannot distinguish between non separable and slowly convergence by running the algorithm). If there are more than one solutions the one that is found depends on the order in which we inspect the points.  

### Probabilistic discriminative model: logistic regression
We want to learn the probability for a certain element to belong to a certain class. We start with the 2 classes scenario:  
![sigmoid_logistic_regression](assets/sigmoid_logistic_regression.png)  
the probability of belonging to the second class is given by  
`p(C2|Φ) = 1 - p(C1|Φ)`  
We use the non-linear [sigmoid function](https://en.wikipedia.org/wiki/Sigmoid_function), also known as logistic function, hence the name. As we said before, the model is non linear in **`w`** but if we put a threshold on the probabilty to have a linear decision surface.

#### Maximum likelihood for logistic regression
To train the parameters of the logistic regression we use maximum likelihood, by finding the weights that maximize the likelihood of getting the right label:  
![maximum_likelihood_logistic_regression](assets/maximum_likelihood_logistic_regression.png)  
Using the same approache we saw before we take the log likelihood(cross-entropy error function) and find the maximum by imposing gradient = 0.  
![cross_entropy_error_function](assets/cross_entropy_error_function.png)  
![cross_entropy_gradient](assets/cross_entropy_gradient.png)  
It has similar structure to the SSE but since the sigmoid function is non linear there is no closed form solution. However the function is still convex and can be optimized using the gradient descent technique.

MULTICLASS CASE  
We represent the posterior probabilities using a *softmax* transormation where the probability of belonging to each class is given by:  
![softmax_multiclass_logistic_regression](assets/softmax_multiclass_logistic_regression.png)  
Then we can apply again a maximum likelihood approach to find the weights obtaining a similar result, the only difference is that i have weights for each class so i have multiple gradients to compute.

### Comparison perceptron vs logistic regression
The percetron can be seen as a particular case of logistic regression where the sigmoid becomes a step function. Both algorithms use the same update rule. The advantage of using logistic regression is that we can obtain a result even if the samples are not linearly separable (altough there will be some misclassification).

## No free lunch theorems


## Bias-Variance trade off


## PAC Learning and VC dimensions


## Kernel methods


## Support Vector Machines (SVM)


## Markov Decision Processes (MDP)


## Dynamic programming


## Reinforcement learning in MDPs


## Multi Armed Bandit (MAB)
