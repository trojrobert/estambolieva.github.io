---
date: 2020-05-01 11:39:00
layout: post
title: Using AI to generate a poem
subtitle:
description: 
image: https://images.unsplash.com/photo-1515104882246-521e5ba18f5e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: portfolio
tags:
  - machine learning
  - poem
  - python
author: estambolieva
---

### Intro

I was recently introduced to Carole, one of the ladies behind the A.I. & Arts initiative called [Open Taggle](http://opentaggle.com/). By the time we spoke they had already used A.I. (read machine learning here) to generate a digital painting which matched each individual's feelings and expressed them in shapes and colors. Pretty neat, huh? 

They had an exhitibion in Amsterdam and I would have loved to have been there. I imagine myself walking up to a blank canvas shown on a computer screen just to see if come alive with my thoughts before me. I am sure that there will be more opportunities like this in the future, and even more exciting art projects to follow.

Coming back to my concersation with Carole. She works in tech and is a keen artist. Her exposure to new technologies - much of her experience coming from IoT, inspired her to imagine all the different ways A.I. can be used to create genuine art. Carole also loves reading, especially poetry. She and her team had already generated paintings from feelings, so she asked herself the question - how can we use A.I. to generate poems from feelings. It was not long until her team decided to take on the challenge and do this. 

They developed an impressive device to read EGG brain waves and transform them into numbers. These numbers are used to understand what a person is feeling right now (while hooked to this device) at something like a scale from 1 to 10 - 1 being very sad and 10 being very happy. Based on this simplified scale of numbered feelings, the number of lines for the poem to be generated is selected and A.I. is used to select a line after another line from a poem dataset.

The current abilities of this technology is to generate Quatrains (4 lines), Tercets (6 lines) and Isometrics (8 lines). The poem title is also automatically generated - and thus a personalized poem is presented to each person who wis curious enough what art can this A.I. algorithm create for them. 

I come in this project as the person to develop the A.I. algorithm to generate the poems. I am not going to explain to you the full methodology and logic that we followed. I discovered some interesting Pythong tricks which made my work easier and I am going to share them with you.


### Data

The data we use is a scrapped list from the Poetry Foundry's website available for [download](https://www.kaggle.com/tgdivy/poetry-foundation-poems) on Kaggle.

When loaded into a pandas dataframe looks something like this:
![Raw Poetry Foundry Data](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/KagglePoetryFoundry.png)

This is what feels worth documenting:
1. [Text Cleaning in Pandas](http://katstam.com/text-cleaning-in-pandas/)
2. [Finding Rhymes](http://katstam.com/finding-rhymes)
