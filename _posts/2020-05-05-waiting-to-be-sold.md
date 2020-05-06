---
date: 2020-05-05 11:15:00
layout: post
title: Predict: how many days before a house is sold
subtitle:
description: 
image: https://images.unsplash.com/photo-1571512050766-0ff3b842dca0?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: portfolio
tags:
  - machine learning
  - prediction
  - house
  - survival analysis
author: estambolieva
---

### Intro

A cleint approached me to replicate the work done in [this article](https://www.researchgate.net/publication/309827179_Waiting_to_Be_Sold_Prediction_of_Time-Dependent_House_Selling_Probability) - *Waiting to be sold: Prediction of time-Dependent House Selling Porbability*. The awesome things about this job was that the piece of software I would produce would be incorporated in my cleint's platform and used on a daily basis by their users.

I quite like it when some academic research can be translated into a successful real-life application. 

*tldr*

*The problem* Real estate webpages such as [trulia.com](trulia.com) fail to inform on how long it would take for a house to sell after it gets listed there.

> This information is equally important for both a potential buyer and the seller. With this information the seller will have an understanding of what she can do to expedite the sale, i.e. reduce the asking price, renovate/remodel some home features, etc. On the other hand, a potential buyer will have an idea of the available time for her to react i.e. to place an offer.

In data science terms, this seems like a linear regression problem at frist glance. The predicted variable is the unit of time it takes for a house to sell - regardless on whether the unit is days, weeks or months.

We expect that the data is collected in the following way: it is crawled which covers a certain amount of time - say 2 months. 

*What is likely to happen?* For example, the data contains houses which are listed and not sold - e.g. we are missing the label of this data point. 


### Technically Speaking

I have 9+ years of experience in machine learning, and I have worked on regression problems numerous times. 

The data set here suggests that a simple regression can be used to predict the number of days a house would stay listed on a webpage before it gets sold. However - for many houses, we do not know the actual sell date. Say that we take a window of 1 month of observations fro this website. Chances are high that few houses would get sold during this month and many houses listed before we started observing would still be for sale. In addition to this, new houses would be added, which also would not likely get sold during our month of observation. This allows us to have a lot of data points, e.g. houses, but few labels - e.g. the number of days it took for a house to get sold after its original listing day. This means that we:

* either discard the data points we cannot get the label for

* or impute the missing label value with something of our choosing.

This all is great however this intervention takes us farther from reality and allows us to train a regression algorithm, which we expect not to work well in real life. For years I thought that since we have missing labels and uncomplete data, there is only so much we can do to solve problems like this and that the best strategy is simply to wait patiently and collect more data diligently.


**Enter medicine.**

> divine music plays

Doctors and researchers often face the following problem: a new treatment is tested on a group of patients to see the effectiveness of it. In many cases it is a life or death situation.

Imagine that a new lung cancer drug is tested and the researchers need to understand whether it *saves* the lifes of the patients or not. It is important to note here that *saves* is a very mild way to put things in, and depending on the clinical trial might mean many things such as *cures*, *reliefs symptoms*, *extends life expectancy*, among others.

Clinical trials are time-boxed - say to 1 year. 100 patients might enroll for the trail at the beginning of the year. 50 could drop out from the trail during the year. The doctors might lose contact with 7 more patients, and sadly, 23 might have died from lung cancer during the trail year.

If the goal is to predict the number of days a patient might live after they start a new treatment, we would have only 23 data points for which we know the exact number of days. This is a 77% reduction of the original data set. We as data scientists might try to impute some of the missing labels so to expand the data set which we would use to train a regression model on.

**Doctors are not data scientists**. When what is predicted is so serious - as to how many days a patients has more to live, it is dangerous to impute data and make assumptions, to put it mildly. To work with problems like this, *survival analysis* has been created. 

**Survival analysis** takes into consideration that 20 patients have lived through the 1 year trail. This means that the number of days to live for them is bigger than 365 and still counting. This is an important piece of information which needs to be taken into account. The fact that no further information can be collected for 57 patients also does not mean that we know the exact days to live for them either. This is something which is also valued in survival analysis. 

Instead of predicting how many days would a patient live, what is predicted is the percentage of all patients who are still alive at incremental time periods - e.g. % of patients alvie after 6 months, 1 year, 1 year 2 months, 1 year 4 motnhs, etc., from the beginning of the trail.

Survival analysis then can be applied to many other mundane real-life problems such as house sale (real estate), employee retention (HR), customer subscirption lifespan (sales), economics, insurance and others. Read more about it in the [links](http://katstam.com/waiting-to-be-sold/#reading) I have shared at the bottom of the page.


### Data

The data used for the experiments in predicting the probability of a house being sold at a given time has been provided to me and I cannot share online. 

A good starting point is always to email the authors of the paper and ask personally for the data to be shared. :)


### Related

Such a wonderful introduction to survival analysis needs its practical tutorial with Python. Here goes my take on it. 

1. [Survival Analysis in Python](http://katstam.com/survival-analysis/)


### Further Reading

* [Applications in Survival Analysis](https://www.researchgate.net/publication/8193428_Applications_in_survival_analysis)

* [Statistical Survival Analysis with Applications](https://link.springer.com/referenceworkentry/10.1007%2F978-1-84628-288-1_19)
