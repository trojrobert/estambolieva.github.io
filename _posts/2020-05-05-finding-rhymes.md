---
date: 2020-05-04 11:39:00
layout: post
title: Using AI to generate a poem
subtitle:
description: 
image: https://images.unsplash.com/photo-1518805672493-adcd9abdc9e0?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1489&q=80
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: programming
tags:
  - rhymes
  - poem
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

The data was clean (read more about some cleaning done [here](http://katstam.com/text-cleaning-in-pandas/)) and all poem lines were separates and inserted into a new row.

![Raw and Clean Poetry Foundry Data](https://raw.githubusercontent.com/estambolieva/estambolieva.github.io/master/assets/img/uploads/KagglePoetryFoundry_cleaned.png)


### Finding Rhymes

**Problem**
We needed to know which lines rhyme with which other lines - regardless of whether they belonged to the same poem or not. 

**Solution**
We assume that all the last words of every line which rhyme to each other belong to the same rhyme category. To find which lines rhyme with which other lines we needed to know to which rhyme category each line belonged to.


We decided to make a new column called *Rhyme_Categories*. In this column - a sub-group of lines which rhyme will the same value in the column *Rhyme_Categories* - that is the name of the rhyme category.

The `pronouncing` library in Python allows for an easy check on all the rhymes a single word has.

```python
[In]: pronouncing..rhymes('map')

[Out]:
 ['app',
 'cap',
 'capp',
  ...
 'yapp',
 'zap',
 'zapp']
```

#### Detect the last word of each line

We decided to use regular expressions to capture the last word of each line. This could have been done with tokenization and libraries such as `nltk` or `spacy` but we opted for the simplest and fastest solution.

We created the column *Last_Word* in the dataframe to write what the regular expression has extracted from each line:

```python
df['Last_Word'] = df.Lines.str.extract('(\w+)\s?[^a-zA-Z0-9]*$')
```

#### Bad Solution to finding rhymes

We can always use the `apply()` function to a dataframe and call `pronouncing.rhymes()` within this fuction. This is a neat column-wise solutions, which I usually prefer when doing data science problems and working with pandas.

```python
rhymes = df.apply(lambda row: (last_word in pronouncing.rhymes(row['Last_Word'])), axis=1) 
```

**PROBLEM**
The problem here is that to map all rhyming lines to a single one it took 9.4 seconds on average. The dataset has more than 317 000 lines, which meant that this 1 line of code would take about 36 days to complete. This is ridiculous and we as programmers should alway strive to do better. In this case - much much better.

#### Good Solution to finding rhymes

Let's flex our computer science brains and device a computer science algorithm to do this. Sure, it might not be a neat column-wise operations. What we want is something which runs very fast.

Here is how the solution looks like:
a. we iterate over each single last word in the dataframe (it takes 25 ms on average per word)
b. for each last word, we return the list of all possible rhymes to it (it takes about 15 ms per word)
c. add each of these rhyming words to a dictionary. The rhyming word in the key, and the last word it rhymes to is the value. This last word then becomes the rhyming category. (it takes 37 ns to add a new element to the dictionary)
d. do not add this rhyming word if it already exists as a key in the dictionary
e. make the dictionary into a separate column titled *Rhyming_Categories*

Let's expand this. Say that the first last word in the column *Last_Word* is *map*. Let's also imagine that the rhyming words `pronouncing` returns are only 3 - *cap*, *tap* and *slap*. What gets added to the originally empty dictionary is the following

```
{
	map: map,
	cap: map, 
	tap: map,
	slap: map
}
```

Imagine that the second last word in the column is *geography* and the rhyming words we get for it are - *oceanography* and *mammography* only. This is how the dictionary evolves:

```
{
	map: map,
	cap: map, 
	tap: map,
	slap: map, 
	geography: geography, 
	oceanography: geography, 
	mammograpgy: geography
}
```

Now imagine that the third last word is *slap*. Nothing gets added to the dictionary at this stage as *slap* exists in the dictionary and its rhyming category is already known - it is *map*.

Following this strategy, we end up with more dictionary words than the number of last words we have it the dataset AND we know the rhyming category for each last word.

Putting this all together in code looks like this:

```python
word_rhymecategory_dict = {}

cnt = 0
for last_word in df.Last_Word:
    if last_word not in word_rhymecategory_dict.keys():
        cnt += 1
        word_rhymecategory_dict[last_word] = last_word
        for rhyme in pronouncing.rhymes(last_word):
            word_rhymecategory_dict[rhyme] = last_word

df['Rhyme_Categories'] = df.Last_Word.map(word_rhymecategory_dict)
```

### Related Posts
* [Using AI to generate a poem](http://katstam.com/ai-poem-generation/)
* [FText Cleaning in Pandas](http://katstam.com/text-cleaning-in-pandas/)

### Reading

I find these articles very useful:


