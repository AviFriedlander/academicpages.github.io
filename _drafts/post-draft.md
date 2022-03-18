---
layout: single
title:  "Evaluating xG"
header:
  include-math: True
  teaser: "unsplash-gallery-image-2-th.jpg"
categories: 
  - Jekyll
tags:
  - statistics
  - expected goals
  - hockey
permalink: /drafts/abc123
---
Evaluating Expected Goals Models
=================================
### What is this article actually about?
First thing's first, this article isn't really about expected goals (xG) models but rather about evaluating and optimizing any model that assigns probabillity values to discrete events. This could apply to many things. In the context of sports, this article could just as easily be applied to game prediction models and outside of sports it applies to any number of things from evaluating a weather channe’s abillity to predict rain to election forecasts. What this article is actually about then is (spoiler alert): log loss. Log loss is a metric that is often used is discussions of hockey analytics for example [here is an example](https://twitter.com/HockeySkytte/status/1401547580425879555) of a comparison of game prediction models using log loss. I suspect many other enjoyers of hockey analytics are like me in the sense that they see “log loss” and can know “small number good” but do not know where it comes from or why we care about it. So in this article I am going to start off with some statistics basics and from there slowly build up to the most reasonable evaluation method and in doing so derive the meaning of log loss.

### What are Expected Goals?
For the purpose of this article it is not important to understand the inner workings of expected goal models, that will be discussed more in the next article. For now, the only important thing to know is that an xG model assigns a value to each shot based on the chance of that shot being a goal. This can depend on charecteristics of the shot like where on the ice was the shot taken, what type of shot was it, and what happened before the shot. Those shot charecteristics get fed into a "model" that then outputs the chance that the shot would end up as a goal. Therefore, for a perfect xG model, a shot with an xG value of 0.15 would have a 15% chance of being a goal. To evaluate an xG model is then a task in determining whether the xG values accurately reflect the probability of a shot becoming a goal. Or alternatively, the main task in creating an xG model is tuning the parameters in a the model to best reflect the true probabillity of a shot ending up as a goal.

### Bernoulli Trials and Probabillity
In order to build a method for evaluating how well an xG model performs over a large sample size like a season where you have thousands of shots, it is useful to establish a statistical foundation. Starting off with a single shot that has been assigned an xG value of $p$ we can say
$$
P(\text{goal} | p) = p
$$
and
$$
P(\text{not goal}|p) = 1-p
$$
where P(data|model), sometimes reffered to as the likelihood, is the probabillity of observing the data assuming the model is correct. In this example with a single shot the "data" is whether the shot is a goal or not and the "model" is the xG value.

Extending this idea of probabillity to observations of more than a single shot, requires the two facts:  
1. The chance a shot has of becoming a goal or not does not depend on whether other shots were goals or not but rather is determined solely by conditions at the time of that specfic shot.
2. If there are two events which respectively have probabillities of $p_1$ and $p_2$ of occuring. The probabillity of both occuring is $p_1 \times p_2$.

Therefore, if we have a dataset that contains a collections of shots and whether each one resulted in a goal or not and we have a model that provides a chance of each shot being a goal we can describe the probabillity of observing the exact sequence of goals and non-goals as
$$
P(data | model) = \prod_i \begin{cases}
p_i, \text{goal} \\
1-p_i, \text{not goal.}
\end{cases}
$$
Here the $\prod$ symbol simply means we are multiplying and $p_i$ is the xG value assigned to the $i$th shot.

At this point we are able to see how to compare two xG models. For each xG model you can calculate the probability of observing the actual results and conclude that the model that is more likely to produce what was obesrved is the “better” model. Alternatively if you are in the process of building an xG model, you can tune the xG model parameters in order to maximize the probabillity in equation 3.

If you look at the above statement and say “Wait, that is the probability of the data being observed not the probability of the model being correct!” Then congratulations, you are a Bayesian! In the next section I will justify why what I said above is correct _enough_ and what caveats exist. If you do not care or you are [frequentist](https://xkcd.com/1132/) then feel free to skip the next section which will not be necessary for anything that follows.

### Turning around the probability

### Maximizing the Likelihood 


### Conclusions
