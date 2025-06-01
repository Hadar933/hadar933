---
title: Discrete Choice Models and Logits
date: 2025-06-01 15:MM:SS +0300
author: hadar
categories: [Econometrics]
tags: [logit]   
math: true
---

# Discrete Choice Models and Logits
To better understand the underlying mechanisms of demand and supply, this document explores core concepts in econometrics as presented in the UMass Amherst course \textit{ResEcon 703}. Topics covered include logit models, random coefficients (mixed) logit models, and BLP estimation, among others. The goal is to investigate how these theoretical frameworks can support the development of more accurate and robust demand estimation models.
## Discrete Choice Problem

Steps to set up and analyze a discrete choice problem:

1. **Specify a choice set** - all possible alternatives available to the decision maker.
   
   For example, for a business trip this weekend, one can fly: first class, business class, economy class, etc.

2. **Formulate a model** of how the agent chooses among the choice set - this model can be parameterized, and when so, the next step would be to estimate those parameters.

Discrete choices are usually modeled under the assumption of *Utility Maximizing* behavior by the decision maker. One framework that provides this is the *Random Utility Model (RUM)*, which is based on two terms: observed utility (based on observed characteristics) and unobserved utility.

## Random Utility Model (RUM)

The core concept is that a decision maker chooses the alternative with the greatest utility.

That is, a decision maker $n$ faces a choice among $J$ alternatives, where each provides utility $U_{nj}$. $n$ will choose $i$ if and only if:

$$\forall j\neq i: \quad U_{ni} > U_{nj}$$

We decompose the utility into observed and unobserved factors:

$$U_{nj} = V_{nj}(x_{nj}, s_n) + \varepsilon_{nj} \quad \text{(1)}$$

where $V$ is called the representative utility, which is a function of the attribute vector of an alternative $x$ and a vector of decision maker attributes $s$. $\varepsilon$ is everything that affects utility that is not part of $V$.

Notice that $\varepsilon$, being the difference between utility and observed (representative) utility, depends on the specification of $V$. $\varepsilon$ is treated as a random variable (from the perspective of the researcher), where $f(\varepsilon_n)$ is the joint distribution of the random vector $\varepsilon_n=\{\varepsilon_{n1},...,\varepsilon_{nJ}\}$. Essentially it's a random term emphasizing unknown utility for each alternative $j\in[J]$ faced by decision maker $n\in[M]$.

### Linear Representative Utility

In many cases, the representative utility is modeled as a linear function of the attributes, with *structural parameters* $\beta'$ that should be approximated:

$$V_{nj} = \beta'x_{nj} \quad \text{(2)}$$

So we can rewrite equation (1) using our linear representative utility from equation (2):

$$U_{nj} = \beta'x_{nj} + \varepsilon_{nj} \quad \text{(3)}$$

## Choice Probabilities

As $\varepsilon$ in equation (3) is unknown, we'd have to make some probability assumptions:

The probability that decision maker $n$ chooses alternative $i$ is:

$$\begin{align}
P_{ni} &= \Pr(U_{ni} > U_{nj} \; \forall j \neq i) \\
       &= \Pr(V_{ni} + \varepsilon_{ni} > V_{nj} + \varepsilon_{nj} \; \forall j \neq i) \\
       &= \Pr(\varepsilon_{nj} - \varepsilon_{ni} < V_{ni} - V_{nj} \; \forall j \neq i)
\end{align}$$

Now, from definition, the probability for an event is the PDF integral over the event's region, which is equivalent to integrating over the entire region but only accounting for the relevant area by zeroing out everything outside the boundaries (with $\mathbf{1}(\cdot)$):

$$\Pr(x\in A) = \int_{A} f(x)dx = \int_{\mathbb{R}} \mathbf{1}(x\in A)f(x)dx$$

So we get:

$$P_{ni}=\int_{\varepsilon} \mathbf{1}(\varepsilon_{nj} - \varepsilon_{ni} < V_{ni} - V_{nj} \; \forall j \neq i) f(\varepsilon_n) \, d\varepsilon_n \quad \text{(4)}$$

Notice how only differences are important in equation (4). As a result, the scale of all $V$'s does not matter (a rising tide lifts all boats).

Our next step is to make a few assumptions on $f(\varepsilon_n)$, and by doing so, find the optimal assignment for the structural parameters $\beta$. When fitting the model to our data, we'd make sure that when the decision maker chooses $i$, the associated probability $P_{ni}\rightarrow 1$ and $\forall j \neq i$, $P_{nj}\rightarrow 0$.

## Logit Model

A logit model is simply the instantiation of $\varepsilon$ with a type I extreme value distribution, namely:

$$\varepsilon_{nj}\sim\text{i.i.d type I extreme value (Gumbel) with Var}(\varepsilon_{nj}) = \frac{\pi^2}{6} \quad \text{(5)}$$

This choice was made because the difference in $\varepsilon$'s is in fact a logistic distribution (hence the name), and plugging this difference yields a closed form for our choice probabilities from equation (4).

*Note: The original document references Figure 1 showing PDF and CDF of type I extreme value distribution (Top) and logistic distribution (Bottom), but the image is not available for conversion.*

Rewriting the choice probability we have:

$$\Pr(\varepsilon_{nj}  < \varepsilon_{ni} + V_{ni} - V_{nj} \; \forall j \neq i)$$

Assume all $\varepsilon_{ni}$, $V_{ni}$ and $V_{nj}$ are known. For a specific choice of $j$ and using the definition $\Pr(X<x) = F_X(x)$, we can replace the right-hand side with the CDF of Gumbel:

$$\Pr(\varepsilon_{nj}  < \varepsilon_{ni} + V_{ni} - V_{nj} | \varepsilon_{ni}) = \exp{(-\exp{-(\varepsilon_{ni} + V_{ni} - V_{nj} )})}$$

And since $\varepsilon_{nj}$ are i.i.d., $\forall j \neq i$ translates to the multiplication:

$$P_{ni}|\varepsilon_{ni} = \prod_{j\neq i}\exp{(-\exp{-(\varepsilon_{ni} + V_{ni} - V_{nj} )})}$$

Since this is still a conditional probability, the transition to probability over all $i$ is via integration over the density, giving us:

$$P_{ni} = \int_{\varepsilon_{ni}=-\infty}^{\varepsilon_{ni}=\infty} \prod_{j\neq i}e^{-e^{-(\varepsilon_{ni} + V_{ni} - V_{nj} )}} e^{-\varepsilon_{ni}}e^{-e^{-\varepsilon_{ni}}}d\varepsilon_{ni} = \frac{e^{V_{ni}}}{\sum_je^{V_{nj}}}=\text{Softmax}(V_{ni}) \quad \text{(6)}$$

(Proof in Train's book 2009, section 3.10)

Crucially, recall how softmax approaches 1 as the argument reaches infinity and approaches 0 when the argument reaches negative infinity.

## Marginal Effects & Elasticities

Marginal effects represent the amplitude of each feature in our data, i.e., its importance in determining the probability of an alternative. In general, we ask: how does the probability $P_{ni}$ change when we change some attribute $k$ on an alternative $j$?

Let $z_{nj} = [x_{nj}]_k$ represent the $k$-th attribute of alternative $j$:

$$\frac{\partial P_{ni}}{\partial z_{nj}}= - \frac{\partial V_{nj}}{\partial z_{nj}}P_{ni}P_{nj} \quad \text{(7)}$$

In the linear utility case, let :
$$V_{nj} = \beta_1 x_{nj1}+...+\beta_k \underbrace{x_{njk}}_{:=z_{nj}}+...$$
we have

$$\frac{\partial P_{ni}}{\partial z_{nj}}= - \beta_k P_{ni}P_{nj} \quad \text{(8)}$$

As an example, this helps us understand how changes in bus price affect the probability of choosing a car.

**Elasticity**, on the other hand, measures the *percentage change* in probability in response to a *percentage change* in the variable. It is defined as:

$$E_{ni, z_{nj}} = \frac{\partial P_{ni}}{\partial z_{nj}} \cdot \frac{z_{nj}}{P_{ni}} =  -\frac{\partial V_{nj}}{\partial z_{nj}}z_{nj}P_{nj}$$

This expression arises from the general definition of elasticity:

$$\text{Elasticity} = \frac{\%\ \text{change in output}}{\%\ \text{change in input}} = \frac{dP_{ni}/P_{ni}}{dz_{nj}/z_{nj}} = \frac{dP_{ni}}{dz_{nj}} \cdot \frac{z_{nj}}{P_{ni}}$$

Thus, elasticity is a rescaled version of the marginal effect, adjusted to provide a unit-free, interpretable measure of sensitivity: it tells us the percentage change in the probability of choosing an alternative $i$ when attribute $z_{nj}$ changes by 1%.

Using equation (8), we get:

$$E_{ni, z_{nj}} = \beta_k P_{ni}P_{nj} \cdot \frac{z_{nj}}{P_{ni}}=-\beta_k z_{nj}P_{nj}$$

Formally, when $i\neq j$ we refer to this as *cross-elasticity*, and one thing to note here is that $i$ (the alternative) is not present in the right-hand side. This is the result of what is known as the Independence of Irrelevant Alternatives (IIA).

## Independence of Irrelevant Alternatives (IIA)

Formally, IIA states that the ratio of any two logit choice probabilities $i,j$ does not depend on attributes of any other alternative. For our multinomial logit model, this is easy to see as:

$$\frac{P_{ni}}{P_{nk} }= \frac{e^{V_{ni}}}{ e^{V_{nk}}} \quad \text{(9)}$$

**Pros and Cons:**

- **Pro:** When the choice set is too large, IIA does not care; we only have to consider a subset of alternatives. Think about the decisions that occur each minute.

- **Con:** Consider commute options: blue bus (bb) and car (c). Assume for simplicity that $P_c=P_{bb}=1/2$ (and hence $P_c/P_{bb}=1$). Now, assume a new red bus (rb) is introduced. Adding a new alternative does not change our ratio from before, $P_c/P_{bb}=1$ (this is the assumption of the IIA). But if our commuter does not care about the bus color, then to him $P_{bb}=P_{rb}$, which means $P_c/P_{bb}=1=P_{rb}/P_{bb}$, hence $P_c=P_{rb}$. This yields that all probabilities are equal, and since they must sum to 1, we get that they all equal 1/3. For our commuter, the true decision hasn't changed; it's still car vs. bus, hence the car's probability should have remained 1/2. But in fact, the model says that now there is a 2/3 chance our commuter will pick a bus, which is a contradiction.

## Scale Parameters

Recall that as $\varepsilon$ is a type I extreme value distribution, the variance is $\pi^2/6$. But let us define a new random utility $\varepsilon^*_{nj}$ with variance $\sigma^2 \pi^2/6$, and an associated $U^*_{nj}=V_{nj}+\varepsilon^*_{nj}$.

Let $U_{nj}=U^*_{nj}/\sigma$ and $\varepsilon_{nj}=\varepsilon^*_{nj}/\sigma$:

$$U_{nj} = \frac{V_{nj}}{\sigma} + \varepsilon_{nj}$$

Since $\text{Var}(ax)=a^2\text{Var}(x)$, we have $\text{Var}(\varepsilon_{nj}) = \pi^2/6$. So our assumption on the distribution remains the same. But what about our scaled observed utility? Under the assumption of linear utility:

$$P_{ni}=\frac{e^{(\beta/\sigma)x_{ni}}}{\sum_je^{(\beta/\sigma)x_{nj}}}$$

These new $\beta/\sigma$ coefficients represent the effect of each observed variable relative to the variance of the unobserved factors. Concretely, it means that if the variance is larger, the observed variable will be smaller even if it has the same effect on utility.

### Heteroskedasticity and the Scale Parameter

In some cases, the variance of the unobserved utility differs between groups of decision makers. Suppose we have commute data for city $A$ and city $B$, with scale parameters $\sigma^A$ and $\sigma^B$, respectively.

We define the scale ratio:

$$k = \left( \frac{\sigma^B}{\sigma^A} \right)^2$$

To normalize the model, we fix $\sigma^A = 1$ (without loss of generality), and define $\beta = \beta^A = \text{true coefficients scaled by } \sigma^A$. Then:

- In city A, the choice probabilities are:
$$P^{(A)}_{ni} = \frac{e^{\beta x_{ni}}}{\sum_j e^{\beta x_{nj}}}$$

- In city B, the coefficients are scaled by $1/\sigma^B = 1/(\sigma^A \sqrt{k}) = 1/\sqrt{k}$, so:
$$P^{(B)}_{ni} = \frac{e^{(\beta / \sqrt{k}) x_{ni}}}{\sum_j e^{(\beta / \sqrt{k}) x_{nj}}}$$

This highlights that if the unobserved variance differs across groups, the estimated coefficients are not directly comparable unless the scale difference is accounted for.

## Counterfactual Simulations

We can compare outcomes in the observed empirical setting to outcomes in an alternate setting with some aspect manipulated - different attributes/data, different choice set, different structural parameters. For example, we want to estimate the demand for education in order to simulate the effects of a school voucher program.

Let $Y_{ni}$ be an indicator equal to 1 if and only if the decision maker chose $i$. From the definition of expectation, we have:

$$\mathbb{E}[Y_{ni}]= P_{ni}$$

Let us denote the probability $P_{ni}^0$ to represent it as derived from our observed data. Assume different probability set $P_{ni}^1$. The change in expected value is:

$$\Delta\mathbb{E}[Y_{ni}] = P_{ni}^1-P_{ni}^0$$

We are sometimes also interested in aggregated choices (over all decision makers):

$$\mathbb{E}[Y_i]=\sum_n \mathbb{E}[Y_{ni}]$$

So in essence we can play with our probability space by, for example, changing our $\beta$ and observe how it affects the expected number of people that will make a decision.