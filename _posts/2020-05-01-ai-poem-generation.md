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

#### Removing opening and trailing unwanted characters for all rows in a column

As you can see each line in the *Poem* column starts with *\r\r\n*. Each of these lines finishes with this character sequence too.

Here is a column-wise way to remove them

```python
df # the pandas dataframe
df.Poem # the Poem column in the dataframe

df.Poem.str.lstrip('\r\r\n') # removes \r\r\n fromt the beginning of the string for each row in the Poem column
df.Poem.str.rstrip('\r\r\n') # removes \r\r\n fromt the end of the string for each row in the Poem column
```

#### Removing duplciates

Some poems are repeated several times and in order to remove these duplicates, we need to call:

```python
df = df.drop_duplicates()
```

#### Split a column into multiple columns

The *Poem* column contains the whole poem with all lines included. In order to automatically generate a new poem which contains many carefully chosen lines from multiple poems - we need to separate the value of each *Poem* into Lines. The character sequence *\r\r\n* is also used as a new line delimiter - e.g. as a separator betweeb lines.

The goal here then is to split each Poem in its constituent lines and keep all other associated information to the Poem, associated to its lines as well.

The first link I listed in the section below called *Reading* is the one that helped me to this. It explains the approach with a simple example.

a. Create a dataframe in the following way:

```python
a=pd.DataFrame({"var1":"a,b,c d,e,f".split(),"var2":[1,2]})
```

![Example Dataframe a](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/Split_String_Rows_1.png)

b. Split all the values in column *var1* at *,* and create a separate row for each string:

```python 
s = a.var1.str.split(",").apply(pd.Series, 1).stack()
```

![The result s](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/Split_String_Rows_2.png)

c. remove the extra index column which is created by *stack()*

```python
s.index = s.index.droplevel(-1)
```

![The result s without the extra index column](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/Split_String_Rows_3.png)

d. give a name of the only column in the series *s*

```python
s.name = 'var 1 separated' 
```

e. add the column *s* to the dataframe and verify results

```python
a.join(s)
```

![The rowed-out column added to a](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/Split_String_Rows_4.png)


I kept the *var1* column in the dataframe for clarity. I also kept the original index which helps me keep track from which original row in the dataframe *a* did the separated string come from.

Putting it altogether, this is the code to separate the poems into lines.

```python
lines = df.Poem.str.split("\r\r\n").apply(pd.Series, 1).stack()
lines.index = lines.index.droplevel(-1)
lines.name = 'Lines'
df = df.join(lines)
```

### Reading

I find these articles very useful:
- [Split strings in a column at comma and separate into multiple columns](https://stackoverflow.com/questions/12680754/split-explode-pandas-dataframe-string-entry-to-separate-rows) [the answer which starts with *Similar question as: pandas: How do I split text in a column into multiple rows?*]

