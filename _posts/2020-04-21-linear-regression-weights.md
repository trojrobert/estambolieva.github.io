---
date: 2020-04-21 11:39:00
layout: post
title: How to read feature importance weights for Linear Regression
subtitle:
description: 
image: https://images.unsplash.com/photo-1521805103424-d8f8430e8933?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1050&q=80
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: ml
tags:
  - machine learning
  - linear regression
  - feature importance
author: estambolieva
---

Let's get a simplistic view of the output of feature weights for the house pricing dataset. The top 3 most important linear regression features when training to predict a house price are *year=2018*, *location=USA* and *age*.

#### Categorical Variables

*year_2018* is a **categorical** variable. The weight 0.3145 tells us that the average price of a house is higher by 0.3145 units when the years is 2018 compared to 2017 and 2019 given all other variables are constant.

#### Continuous Variables

*age* is a **continuous** variable. The weight 0.0488 tells us that a unit increase in age (e.g. when the person is one year older) results in an average increase of 0.0488 of the house price.

```python3
Weight     Feature
0.3145     year_2018
0.1503     location_USA
0.0488     age
0.0312     year_2017
0.0057     interest
....       ....
```

### Ralted Posts:

This post is a follow up to the [Feature Importance with OneHotEncoder and Pipelines in Scikit-learn](http://katstam.com/regression-feature_importance/).

Another related post is the [Regression Tips anc Tricks](http://katstam.com/regression-tips/) Series I have made.

### Reading

I find these articles very useful:
- [Interpreting the coefficients of linear regression](https://towardsdatascience.com/interpreting-the-coefficients-of-linear-regression-cc31d4c6f235)
