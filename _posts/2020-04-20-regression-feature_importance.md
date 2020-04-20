---
date: 2020-04-20 15:00:00
layout: post
title: Feature Importance with OneHotEncoder and Pipelines in Scikit-learn
subtitle:
description: 
image: https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Linear_regression.svg/400px-Linear_regression.svg.png
<!-- optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825288/theme17_nlndhx.jpg -->
category: ml
tags:
  - machine learning
  - regression
  - python3
  - sklearn
  - feature importance
  - OneHotEncoder
author: estambolieva
---

I recently played around with linear regression and scikit-learn. 

I discovered how to use pipelines to stream-line the model training process, and now I quite like using Pipelines. I could not, however, discover how to see the most important features for the models trained when I have used a pipeline. Let me first explain how the pipelines work, and then I will tell you what prevented me from easily seeing the most important features.

### Training Pipelines

Here is how a training pipeline looks like:

1. First I want to understand what type of variables are held in each variable (a dataframe column) - numeric, continuous, categorical or boolean. For a description of the problem I have chosen - read [here](http://katstam.com/regression-tips/#intro).

```python
X_train # training dataframe

# a. check the types of the columns
[In]: X_train.dtype

[Out]:
house price      float64
location         category 
age              int64
interest         float64
interest rate    object
year             period[M]

```

2. After I know this I create two arrays in which I hold the names of the dataframe columns with *numeric* and *categorical* values.


```python
# b. get the names of the numeric anc categorical columns
numeric_features = X_train.select_dtypes(include=['int64', 'float64']).columns
categorical_features = X_train.select_dtypes(include=['object', 'category', 'period[M]']).columns
```

3. Then I need to encore the categorical valiables - and I choose [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html) to do this. OneHotEncoder works beautifully - here it goes:

Let's imagine a very simplistic scenario. Let's say that I have data only for houses purchased in 2017, 2018 and 2019. I have chosen to treat the variable *year* as a datetime type instead of numerical. Let's also assume that I have data for only 5 houses. The values of the variable *year* would look something like this:

```python
[In]: X_train.year

[Out]:
index    year
0        2017
1        2019
2        2019
3        2018
4        2017 
```

OneHotEncoder transforms the column *year* into another 3 binary (and numeric for False is 0, and True is 1) columns names *year_2017*, *year_2018* and *year_2019*. Each of this columns will have the value of 1 when the house is bought during the year in its title, and 0 otherwise. The column *year* would then not be used during training (but is listed below for simplicity).

```python
[In]: X_train.year

[Out]:
index    year    year_2017    year_2018    year_2019
0        2017            1            0            0
1        2019            0            0            1
2        2019            0            0            1
3        2018            0            1            0
4        2017            1            0            0        
```

Let's finally apply OneHotEncoder to the data set, while imputing (or filling) missing values. Because I will do 2 things here already - encoding and imputing, I will do these using a pipeline.
*Note*: Scikit-learn does not allow for datasets to have missing values - and if they do, throws an error during training.

```python
# import the needed libraries first
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

# create a transformer for the categorical values
categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('one_hot', OneHotEncoder())])

# create a transformed for the numerical values
numeric_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())])
```

*Note*: The numerical columns are also encored here - they are scaled and all missing values replaced wit the median value for each variable.

Finally, apply these transformations to call columns (variables) in the dataset

```python
from sklearn.compose import ColumnTransformer

preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)
    ])
```

4. Train a Linear Regression (or a model of your choise) using a pipeline

```python
clf = Pipeline(steps=[('preprocessor', preprocessor),
                      ('classifier', LinearRegression())])
```

Perfect, I now have a trained Linear Regression model, which I'd like to see feature important for. This is how it is done :).

### Feature Importance in Pipelines: Problem

Usually, I would get the `coef_` of a scikit-learn trained model, which will give me the weights for each of the features (variables or columns). The problem here is that OneHotEncoder creates new features, as shown above. I am not aware of what the names of these features is and which of the weights of `coef_` corresponds to each of the new feature set. 

This cannot be right as a data scientist I must know which features and important for the model, and find out which features can be dropped from the training as they bring little information.

### Feature Importance in Pipelines: Solution

1. Get OneHotEncoder column names using `named_steps` of the training pipeline

As you see above, I have named the OneHotEncoding step as *one_hot*, the Linear Regression training step and *classifier*. `named_steps` allows me to get only one of all pipeline steps and inspect it alone. What I want to do first is to get the names of the columns which were created during OneHotEncoding.

To do this, I need to get to the *preprocessor* step, call the categorical transformer, which I have titled *cat*, get the OneHotEncoder step *one_hot*

```python3
[In]: onehot_columns = list(clf.named_steps['preprocessor'].named_transformers_['cat'].named_steps['one_hot'].get_feature_names(input_features=categorical_features))

[Out]:
['year_2017', 'year_2018', 'year_2019', 'location_USA', ...., 'location_Brazil']
```

2. Join the lists of the numerical column names and the OneHotEncoder column names

```python3
numeric_features_list = list(numeric_features)
numeric_features_list.extend(onehot_columns)
```

3. Use the magic library [ELI5](https://eli5.readthedocs.io/) (or 'explain like I’m 5')

```python3
eli5.explain_weights(clf.named_steps['classifier'], top=50, feature_names=numeric_features_list, feature_filter=lambda x: x != '<BIAS>')
```

The a very beautiful list with the weight + column name is printed to show feature importance - all positive weights are in green, while the negative weights are visualized in red. 

Here is a sample output of `eli5.explain_weights`:

![Sample output](https://miro.medium.com/max/550/1*KkcCm7-mxrFk-d5muOUTdQ.png)

### Ralted Posts:

This post is a follow up to the [Regression Tips anc Tricks](http://katstam.com/regression-tips/) Series I have made.

### Reading

I find these articles very useful:
- [Evaluating a Linear Regression Model](https://www.ritchieng.com/machine-learning-evaluate-linear-regression-model/)
- [Extracting Feature Importance from SKlearn Pipelines](https://towardsdatascience.com/extracting-feature-importances-from-scikit-learn-pipelines-18c79b4ae09a)
