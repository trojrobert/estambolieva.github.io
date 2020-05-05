---
date: 2020-05-01 11:39:00
layout: post
title: Using AI to generate a poem
subtitle:
description: 
image: https://images.unsplash.com/photo-1576269483449-3b694997b362?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: programming
tags:
  - text cleaning
  - nlp
  - python
author: estambolieva
---

Read more about the backstory of this project and its dream [here](http://katstam.com/ai-poem-generation/)

**tldr;**
I participated in one of the projects of [Open Taggle](http://opentaggle.com/) - to use A.I. to generate a poem from a person's feelings. The feelings are captured using IoT to read EGG waves. I come in this project as the person to develop the A.I. algorithm to generate the poems. 

### Data

The data we use is a scrapped list from the Poetry Foundry's website available for [download](https://www.kaggle.com/tgdivy/poetry-foundation-poems) on Kaggle.

When loaded into a pandas dataframe looks something like this:
![Raw Poetry Foundry Data](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/KagglePoetryFoundry.png)

Here are some smart ways to clean the poem's textual data in python:
1. [Removing opening and trailing unwanted characters for all rows in a column](http://katstam.com/text-cleaning-in-pandas/#removing-opening-and-trailing-unwanted-characters-for-all-rows-in-a-column)
2. [Removing duplciates](http://katstam.com/text-cleaning-in-pandas/#removing-duplciates)
3. [Split a column into multiple columns](http://katstam.com/text-cleaning-in-pandas/) _____
4. [Remove non-ascii characters from text in columns](http://katstam.com/text-cleaning-in-pandas/)

#### 1. Removing opening and trailing unwanted characters for all rows in a column

As you can see each line in the *Poem* column starts with *\r\r\n*. Each of these lines finishes with this character sequence too.

Here is a column-wise way to remove them

```python
df # the pandas dataframe
df.Poem # the Poem column in the dataframe

df.Poem.str.lstrip('\r\r\n') # removes \r\r\n fromt the beginning of the string for each row in the Poem column
df.Poem.str.rstrip('\r\r\n') # removes \r\r\n fromt the end of the string for each row in the Poem column
```

#### 2. Removing duplciates

Some poems are repeated several times and in order to remove these duplicates, we need to call:

```python
df = df.drop_duplicates()
```

#### 3. Split a column into multiple columns

The *Poem* column contains the whole poem with all lines included. In order to automatically generate a new poem which contains many carefully chosen lines from multiple poems - we need to separate the value of each *Poem* into Lines. The character sequence *\r\r\n* is also used as a new line delimiter - e.g. as a separator between lines.

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


**Note**: Keeping the original index is very important here. As you can see the index value for *a*, *b* and *c* is 0 as they all originate from the initial row with index 0. As we want the values of *var2* to be associated in the original way to the values of *a*, *b* and *c*, it is important that all the values *a*, *b* and *c* have index 0 so that pandas *join* can attach to them the same row values for the other columns as if they were still in one row - *a,b,c*


d. give a name of the only column in the series *s*

```python
s.name = 'var 1 separated' 
```

e. add the column *s* to the dataframe and verify results

```python
a.join(s)
```

![The rowed-out column added to a](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/Split_String_Rows_4.png)


I kept the *var1* column in the dataframe for clarity. 


Putting it altogether, this is the code to separate the poems into lines.

```python
lines = df.Poem.str.split("\r\r\n").apply(pd.Series, 1).stack()
lines.index = lines.index.droplevel(-1)
lines.name = 'Lines'
df = df.join(lines)
```

#### 4. Remove non-ascii characters

I discovered that some values in *Poem* and some values in *Title* contains non-ascii characters such as *\ufeff*. To remove them from each value in these two columns, the method *remove_non_ascii()* is applied to the columns

```python
def remove_non_ascii(text): # removes all non-ascii characters
    return ''.join(i for i in text if ord(i)<128)

df.Poem = df.Poem.apply(remove_non_ascii)
df.Title = df.Title.apply(remove_non_ascii)
```

### Related Posts
* [Using AI to generate a poem](http://katstam.com/ai-poem-generation/)
* [Finding Rhymes](http://katstam.com/finding-rhymes/)

### Reading

I find these articles very useful:
- [Split strings in a column at comma and separate into multiple columns](https://stackoverflow.com/questions/12680754/split-explode-pandas-dataframe-string-entry-to-separate-rows) [the answer which starts with *Similar question as: pandas: How do I split text in a column into multiple rows?*]

