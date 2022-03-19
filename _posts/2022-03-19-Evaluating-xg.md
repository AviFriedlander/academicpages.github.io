---
layout: single
title:  "Evaluating xG"
header:
  include-math: True
tags:
  - statistics
  - expected goals
  - hockey
permalink: /posts/2022/03/evaluating-xg
---
How to Evaluate Expected Goals Models?
=================================
### What is this article actually about?
First thing's first, this article isn't really about expected goals (xG) models but rather about evaluating and optimizing any model that assigns probability values to discrete events. This could apply to many things. In the context of sports, this article could just as easily be written about game prediction models and outside of sports it applies to any number of things from evaluating a weather channel's ability to predict whether it will rain to election forecasts. What this article is actually about then is (spoiler alert): log loss. Log loss is a metric that is often used is discussions of hockey analytics for example [here](https://twitter.com/HockeySkytte/status/1401547580425879555) it is used to compare game prediction models. In the past when I have seen “log loss”, I have known “small number good” but did not know where it comes from or why I should care about it. Statistics that are not understood can be counter productive and confusing so in this article I hope to provide that understanding. I am going to start off with some statistics basics and from there slowly build up to the most reasonable evaluation method and in doing so derive the meaning of log loss.

### What are Expected Goals?
For the purpose of this article it is not important to understand the inner workings of expected goal models, that will be discussed more in my next article. For now, the only important thing to know is that an xG model assigns a value to each shot based on the chance of that shot being a goal. This can depend on characteristics of the shot like where on the ice was the shot taken, what type of shot it was, and what happened before the shot. Those shot characteristics get fed into a "model" that then outputs the chance that the shot would end up as a goal. Therefore, for a perfect xG model, 15% of shots with an xG value of 0.15 would end up being a goal. To evaluate an xG model is then a task in determining whether the xG values accurately reflect the probability of a shot becoming a goal. Or alternatively, the main task in creating an xG model is tuning the parameters in the model to best reflect the true probability of a shot ending up as a goal.

### Statistical foundation
In order to build a method for evaluating how well an xG model performs over a large sample with thousands of shots, it is useful to establish a statistical foundation. Starting off with a single shot that has been assigned an xG value of $p$ we can say

$$
P(\text{goal} | p) = p
$$

and

$$
P(\text{not goal}|p) = 1-p
$$

where $P(\text{data}\vert\text{model})$, sometimes referred to as the likelihood, is the probability of observing the data assuming the model is correct. In this example with a single shot the "data" is whether the shot is a goal or not and the "model" is the xG value.

Extending this idea of probability to observations of more than a single shot, requires the two facts:  
1. The chance a shot has of becoming a goal or not does not depend on whether other shots were goals or not but rather is determined solely by properties of that specific shot.
2. If there are two events which respectively have probabilities of $p_1$ and $p_2$ of occurring, then the probability of both occurring is $p_1 \times p_2$.

Therefore, if we have a data set that contains a collection of shots and whether each one resulted in a goal or not and we have a model that provides a chance of each shot being a goal we can describe the probability of observing the exact sequence of goals and non-goals as

$$
P(\text{data} | \text{model}) = \prod_i 
\begin{cases}
p_i, & \text{goal} \\
1-p_i,& \text{not goal.}
\end{cases}
$$

Here the $\prod$ symbol simply means we are multiplying and $p_i$ is the xG value assigned to the $i$th shot.

This is all the statistics we need to know how to compare two xG models. For each xG model you can calculate the probability of observing the actual results and conclude that the model that is more likely to produce what was observed is the “better” model. Alternatively if you are in the process of building an xG model, you can tune the xG model parameters in order to maximize the probability in Equation 3.

If you look at the above statement and say “Wait, that is the probability of the data being observed not the probability of the model being correct” then congratulations, you are a Bayesian! In the next section I will justify why what I said above is correct _enough_ and what caveats exist. If you do not care or you are [frequentist](https://xkcd.com/1132/) then feel free to skip the next section which will not be necessary for anything that follows.

### Turning around the probability
In the discussion above and in what follows, I talk about maximizing $P(\text{data}|\text{model})$ which is the probability of a sequence of observations happening assuming that a certain model is correct. There are contexts where this is the quantity that is needed. For example, when making a projection for what will happen in the future, you are implicitly assuming a model and then making a prediction for the probability of different "data" results occurring. However, when evaluating or training an xG model, all we have is the data that has occurred and we are using that to determine the probability that a model is correct. For that we need to determine $P(\text{model}|\text{data})$. In order to make that conversion, we need to use Bayes' theorem. I am not going to derive or explain the origin of Bayes' theorem (for a very nice explanation watch [this 3Blue1Brown video](https://youtu.be/HZGCoVF3YvM).) Rather I will just present the result:

$$
P(\text{model} | \text{data}) = \frac{P(\text{data}|\text{model}) P(\text{model}) }{ P(\text{data}) } 
$$

where $P(\text{model})$ is known as the “prior” and $P(\text{data})$ is a normalization constant that does not depend on the model. 

When optimizing a model on a specific data set or comparing two models ability to explain the same data, the $P(\text{data})$ normalization constant will not change and therefore can be safely ignored. The prior, however, is more important. As the name suggests, this factor quantifies any “prior” knowledge about what we expect the true model to be. There are situations where the prior is very important. An intuitive example that illustrates why priors are important is if you own dice for a board game which you have used for years and one day you start a game rolling five 1’s in a row. The most reasonable assumption is that the dice have not changed and that the five 1’s in a row is just a random coincidence. The prior is a way of consistently incorporating expectations and prior knowledge in a systematic manner. In the case of xG models, it makes sense to compare models without any bias for what the model should be. Therefore, the prior will be “flat”, without bias towards any given model, and maximizing $P(\text{model}\vert\text{data})$ is the same as maximizing $P(\text{data}\vert\text{model})$[^1].

### Maximizing the Likelihood 
At this point we have established everything that is theoretically needed to optimize and evaluate xG models. All that is left is turning what we have into a format that is slightly easier to use and looking at a simple example to help illustrate how this all works. As discussed above the probability of independent events occurring multiplies. It is sometimes easier to deal with addition rather than multiplication. That change can be made by taking the logarithm of Equation 3. Probabilities are always less than one so the logarithm is negative. In order to only deal with positive numbers, it is common to flip the sign and minimize the negative logarithm instead of maximizing the probability itself. For this type of problem (determining the chance of an event occurring or not), the negative logarithm of the likelihood is known as the log loss and can be written as

$$
-\log\bigg(P(\text{data} | \text{model})\bigg) = -\sum_i 
\begin{cases}
\log(p_i),& \text{goal} \\
\log(1-p_i),& \text{not goal}
\end{cases}
\equiv \text{log loss}
$$

where $\sum$ indicates that we are adding the logarithm of the probabilities instead of multiplying. It is important to emphasize that whether you maximize the likelihood or minimize the log loss is only a matter of convenience and does not change the result.

### A short example
At this point it is nice to work through a simple example to see how this can work in practice. This can be done with the simplest possible xG model where all shots have the same value, $p$. This is a model that knows nothing about an individual shot and will just determine what the chance the average shot has of becoming a goal. If in a data set there are $G$ goals and $S$ total shots then the log loss can be easily calculated because there are $G$ events that had a $p$ chance of occurring and $(S-G)$ events that had a $1-p$ chance of occurring. Therefore the total log loss is

$$
\text{log loss} = -G \log(p) - (S-G)\log(1-p) .
$$

The log loss will be minimized when its derivative is equal to zero so

$$
-\frac{G}{p} + \frac{S-G}{1-p} = 0
$$

and by isolating for $p$, this leads to

$$
p = \frac{G}{S}.
$$ 

As might be expected this method shows that the best value for $p$ in this simple xG model is just the shooting percentage of all the shots in the data set. For more complicated xG models, a simple equation may not be able to be found like this but the same principles apply and optimal parameters can be chosen by using numerical methods to find the minimal log loss.

### Takeaways
In this article I have shown why log loss is the best method for comparing xG models. This isn’t a new idea. When xG models have been developed in the past ([for example](https://fooledbygrittiness.blogspot.com/2018/01/expected-goals-model.html) from Harry Shomer and [another example](https://evolving-hockey.com/blog/a-new-expected-goals-model-for-predicting-goals-in-the-nhl/) from Evolving-Hockey) log loss has been presented as a number to validate the model, often along with another metric called Area Under Curve (AUC). If you are someone looking to develop a new xG model (or anything else that assigns probabilities to events happening) then I has helped you understand how to quantify the accuracy of your model. If you have no interest in making or evaluating these models yourself then I hope that next time you see someone else discussing their model or using log loss you will have a better sense of what that number is actually telling you.

[^1]: There is an additional complications here. In this article I have avoided discussing what the “model” actually is. The model is typically a function that depends on some number of parameters. A flat prior is flat over those model parameters but this framework is not as straightforward when comparing two models that have different parameters or inputs, especially if the number of inputs and parameters are different between models. In physics, which is the context I am most familiar with, [Occam’s Razor](https://en.wikipedia.org/wiki/Occam%27s_razor) is used which translates to a preference of models with fewer parameters. In that context more work has to be done to compare different models with different numbers of inputs. For an example of how this can be done in physics, [this is a recent paper](https://arxiv.org/abs/2107.10291) that performs this sort of comparison on different models that try to solve the [Hubble tension](https://www.scientificamerican.com/article/hubble-tension-headache-clashing-measurements-make-the-universes-expansion-a-lingering-mystery/). With that being said, how to deal with models with different numbers of parameters is somewhat subjective. I do not believe that the bias towards fewer parameters is justified in the context of xG models where adding new parameters should not be penalized so long as the model performs better on data sets it was not trained on. Therefore the methods discussed in this article work even when comparing very different xG models. For a much more in-depth discussion of Bayesian statistical methods in physics (specifically cosmology), check out [these notes](https://arxiv.org/pdf/1701.01467.pdf) which have been extremely valuable to my understanding of statistics.
