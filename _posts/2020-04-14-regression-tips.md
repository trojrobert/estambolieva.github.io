---
date: 2020-03-25 13:17:00
layout: post
title: Linear Regression Tips and Tricks
subtitle:
description: 
image: https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Linear_regression.svg/400px-Linear_regression.svg.png
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: ml
tags:
  - machine learning
  - regression
  - python3
  - tips
author: estambolieva
---

ToC:
1. [Introduction](http://katstam.com/regression-tips/#intro)
2. [Tip 1: No missing values in training AND some missing values in prediction](http://katstam.com/regression-tips/#tip-1-no-missing-values-in-training-and-some-missing-values-in-prediction)
3. [Tip 2: Train Regression models when missing values exist](http://katstam.com/regression-tips/#tip-2-train-regression-models-when-missing-values-exist)
4. [Tip 3: Bin continuous values to categories](http://katstam.com/regression-tips/#tip-3-bin-continuous-values-to-categories)

### Intro

I recently completed a piece of data science piece of work in which the task was to understand how to offer better mortgage loan conditions to existing borrowers. What needed to be predicted was a continuous variable, which was a parameter in calculating the improved loan conditions.

The training data available ha around 68K data points - and was clean, with no missing values and almost ready to use. Such datasets are offered rarely in the real world and it was a pleasure working with these data :). 

The training data (I will call it the *training set*) had features such as the 1. age of the borrower, 2. the geographic location of the house, 3. the house price, 4. the close date on the loan, 5.the interest rate type along with 6. the actual interest rate. I will call these fields columns as well as the data was delivered in excel sheet files and each field had its separate column.

40K data points (I will call them the *prediction set*) needed this parameter to be predicted, using linear regression. It is interesting to mention here that the 40K had __*a lot of missing values in some of the features*__, and __*some missing values in other features*__.


### Tip 1: No missing values in training AND some missing values in prediction

The parameter which was to be predicted has values between -286 and 607.
When a linear regression model with the original training set was trained to predict it, the model's [mean absolute error](https://en.wikipedia.org/wiki/Mean_absolute_error), or MSE, was 32. This is a good result.

It is easy to imagine that it will not work quite well on the *prediction set* because the prediction set contains many missing values. 

What I did was to artificially mimic the amount of missing values in the prediction set for the training set.

For example, age was missing 77% of the time in the prediction set. I also checked how well age correlates (using pearson's correlation) to the parameter to be predicted - and they were very weakly positively correlated. This is good. I proceeded to randomly erasing 77% of the age values in the training set. This was done randomly as there appeared to be no strong relationship between age and the other features. Here is the code to inject missing values in such a way:

```python
X # the pandas dataframe in which the training set data is loaded 
total_number # the total number of data points in the training set
percentages = [77, 77, 64, 38] # the percentage of missing values for 4 columns in the prediction set
column_names = ['age', 'interest', 'house price', 'interest rate'] # the names of the columns in X which relate to the prercentages above

for i in range(len(percentages)):
    percentage = percentages[i]
    column_name = column_names[i]
    num_values_to_null = int(percentage * total_number / 100)
    indices_to_null = random.sample(range(total_number), num_values_to_null)
    X.at[X.index[indices_to_null], column_name] = np.nan # first argument is the list of indices, and the second element is the name of the column
```

After artificially injecting missing values in the training set, a regression model was again trained on it. The mean absolute error then increased slightly to 34. I was happy with the results and it seems like the columns with missing values were not too important for the model as the performance did not decrease much.

### Tip 2: Train Regression models when missing values exist

The training set had missing values introduced, and `sklearn.LinearRegression` cannot be trained before all missing values are [imputed](https://scikit-learn.org/0.16/modules/generated/sklearn.preprocessing.Imputer.html) in a way.

Most missing values in numerical columns related to the loan interest were replaces with 0. This was safe as we know that if the loan interest field has a value - it is greater than 0. 

I chose to treat age differently because even though it was a numerical value, we - humans - know that age has its limitation. For example, it is very difficult for the value of it to be greater than 100. Houses can be very expensive and as humans - we would need to think a lot about setting the upper limit for it. However - it is easier to set the upper limit for age to be 100. It comes intuitively to us that very few people who are above 100 would buy a house on their own name and receive a mortgage loan for that. 
A good practice is to impute the missing value with the average, or the mean. I did that so all missing values in my age column were replaced with 64.  

The missing values of the categorical features were replaced by *missing* - thus creating one more category for it.

This is the example code I used to impute the missing values:

```python
X.fillna({ 'interest_ate': 0.0, 'age': int(X.age.mean()), 'location': 'missing', inplace = True})

```


### Tip 3: Bin continuous values to categories

An extra column, *age_binned* was created, which transformed the continuous values of *age* to categories, or bins. Each bin contained ages such as the age between (60, 65]. The youngest borrower was 20, and the oldest - 80 years old. Thus 11 bins were created - (20,25], (25,30], (30,35], (35,40], (40,45], (45,50], (50,55], (55,60], (60,65], (65,70], (70,75] and (75,80].

Here is the code to creat bins of a column in a dataframe:

```python
bucket_size # the size of the bin. Bucket size is set to 5 in the case of age

def bin_column_values(column_name, new_column_name, bucket_size):
    min_value = X[column_name].min()
    max_value = X[column_name].max()
    bins = np.arange(int(math.floor(min_value / bucket_size)) * int(bucket_size), int(math.ceil(max_value / bucket_size)) * int(bucket_size) + int(bucket_size), int(bucket_size))
    X[new_column_name] = pd.cut(X[column_name], bins)
```

Whenever the age was a missing value, it got assigned the value of 64. The binned age value was automatically set to (60,65] - which is the 9th bin with index 8 of all the bins. 

```python
X.fillna({'age_binned': X.age_binned.cat.categories[8]}, inplace = True)
```

### Reading

I find these articles very useful:
- [Evaluating a Linear Regression Model](https://www.ritchieng.com/machine-learning-evaluate-linear-regression-model/)
