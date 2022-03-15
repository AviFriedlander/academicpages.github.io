---
layout: single
title:  "Evaluating xG"
header:
  teaser: "unsplash-gallery-image-2-th.jpg"
categories: 
  - Jekyll
tags:
  - statistics
  - expected goals
  - hockey
---
Evaluating Expected Goals Models
=================================
What is this article actually about?
------------------------------------
First thing's first, this article isn't really about expected goals (xG) models but rather about evaluating and optimizing any model that assigns probabillity values to discrete events. This could apply to many things. In the context of sports, this article could just as easily be applied to game prediction models and outside of sports it applies to any number of things from evaluating a weather channe’s abillity to predict rain to election forecasts. What this article is actually about then is (spoiler alert): log loss. Log loss is a metric that is often used is discussions of hockey analytics for example \[here is an example] of a comparison of game prediction models using log loss. I suspect many other enjoyers of hockey analytics are like me in the sense that they see “log loss” and can know “small number good” but do not know where it comes from or why we care about it.
