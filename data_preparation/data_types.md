

Handling Data Types
===

### What is this about?

One of the first things to do when we start a data project is to assign the correct data type for each variable. Although this seems a straightforward task, some algorithms work with certain data types. Here, we'll try to cover these conversions while explaining with examples the implications in each case.


<br>

**What are we going to review in this chapter?**

* Detecting the correct data type
* How to convert from categorical to numerical
* How to convert from numerical to categorical (discretization methods)
* Theoretical and practical aspects (examples in R)
* How a predictive model looks at numerical variables

<br>


### The universe of data types

There are two main data types, **numerical** and **categorical**. Other names for categorical are **string** and **nominal**.

A subset of categorical is the ordinal or, as it is named in R, an **ordered** factor. At least in R, this type is only relevant when plotting categories in a certain order. An example in R:


```r
# Creating an ordinal or ordered factor
var_factor=factor(c("3_high", "2_mid", "1_low"))
var_ordered=factor(var_factor, ordered = T)
var_ordered
```

```
## [1] 3_high 2_mid  1_low 
## Levels: 1_low < 2_mid < 3_high
```

Don't pay too much attention to this data type as numerical and categorical are the most needed.

<br>

#### Binary variable, numerical, or categorical?

This book suggests using binary variables as numeric when `0` is `FALSE` and `1` is `TRUE.` This makes it easier to profile data. 

<br>

### Data types per algorithm

Some algorithms work as follows:

* Only with categorical data
* Only with numerical data
* With both types

Moreover, not every predictive model can handle **missing value**. 

The **Data Science Live Book** tries to cover all of these situations.

<br>

### Converting categorical variables into numerical 

Using the `caret` package in R is a straightforward task that converts every categorical variable into a **flag one**, also known as a _dummy_ variable.

If the original categorical variable has thirty possible values, then it will result in 30 new columns holding the value `0` or `1`, where `1` represents the presence of that category in the row.

If we use the caret package from R, then this conversion only takes two lines of code:


```r
library(caret) # contains dummyVars function
library(dplyr) # data munging library
library(funModeling) # df_status function
  
## Checking categorical variables
status=df_status(heart_disease, print_results = F)
filter(status,  type %in% c("factor", "character")) %>% select(variable)
```

```
##              variable
## 1              gender
## 2          chest_pain
## 3 fasting_blood_sugar
## 4     resting_electro
## 5                thal
## 6        exter_angina
## 7   has_heart_disease
```

```r
## It converts all categorical variables (factor and character) into numerical variables
## It skips the original variable, so no need to remove it after the conversion, the data is ready to use.
dmy = dummyVars(" ~ .", data = heart_disease)
heart_disease_2 = data.frame(predict(dmy, newdata = heart_disease))

# Checking the new numerical data set:
colnames(heart_disease_2)
```

```
##  [1] "age"                    "gender.female"         
##  [3] "gender.male"            "chest_pain.1"          
##  [5] "chest_pain.2"           "chest_pain.3"          
##  [7] "chest_pain.4"           "resting_blood_pressure"
##  [9] "serum_cholestoral"      "fasting_blood_sugar.0" 
## [11] "fasting_blood_sugar.1"  "resting_electro.0"     
## [13] "resting_electro.1"      "resting_electro.2"     
## [15] "max_heart_rate"         "exer_angina"           
## [17] "oldpeak"                "slope"                 
## [19] "num_vessels_flour"      "thal.3"                
## [21] "thal.6"                 "thal.7"                
## [23] "heart_disease_severity" "exter_angina.0"        
## [25] "exter_angina.1"         "has_heart_disease.no"  
## [27] "has_heart_disease.yes"
```

Original data `heart_disease` has been converted into `heart_disease_2` with no categorical variables, only numerical and dummy. Note that every new variable has a _dot_ followed by the _value_.

If we check the before and after for the 7th patient (row) in variable `chest_pain` which can take the values `1`, `2`, `3` or `4`, then


```r
# before
as.numeric(heart_disease[7, "chest_pain"])
```

```
## [1] 4
```

```r
# after
heart_disease_2[7, c("chest_pain.1", "chest_pain.2", "chest_pain.3", "chest_pain.4")]
```

```
##   chest_pain.1 chest_pain.2 chest_pain.3 chest_pain.4
## 7            0            0            0            1
```

Having kept and transformed only numeric variables while excluding the nominal ones, the data `heart_disease_2` are ready to be used.

More info about `dummyVars`: http://amunategui.github.io/dummyVar-Walkthrough/

<br>


### Is it categorical or numerical? Think about it.

Consider the `chest_pain` variable, which can take values `1`, `2`, `3`, or `4`. Is this variable categorical or numerical?

If the values are ordered, then it can be considered as numerical as it exhibits an **order** i.e., 1 is less than 2, 2 is less than 3, and 3 is less than 4. 

If we create a decision tree model, then we may find rules like: "`If chest_pain > 2.5, then...`". Does it make sense? The algorithm splits the variable by a value that is not present (`2.5`); however, the interpretation by us is "if `chest_pain` is equal or higher than 3, then…”

<br>

#### Thinking as an algorithm

Consider two numerical input variables and a target binary variable. The algorithm will _see_ both input variables as dots in a rectangle, considering that there are infinite values between each number. 

For example, a **Supported Vector Machine** (SVM) will create _several_ vectors in order to separate the target variable class. It will **find regions** based on these vectors. How would it be possible to find these regions based on categorical variables? It isn't possible and that's why SVM only supports numerical variables as with artificial neural networks.

<img src="svm.png" alt="Support Vector Machine" width="200px">

_Image credit: ZackWeinberg_

The last image shows three lines, representing three different decision boundaries or regions.

For a quick introduction to this SVM concept, please go to this short video: <a href=" https://www.youtube.com/watch?v=1NxnPkZM9bc" target="blank">SVM Demo</a>.

However, if the model is tree-based, like decision trees, random forest, or gradient boosting machine, then they handle both types because their search space can be regions (same as SVM) and categories. Like the rule "`if postal_code is AX441AG and age > 55, then...`".

Going back to the heart disease example, the variable `chest_pain` exhibits order. We should take advantage of this because if we convert this to a categorical variable, then **we are losing information** and this is an important point when handling data types.

<br>

#### Is the solution to treat all as categorical?

No... A numerical variable carries more information than a nominal one because of its order. In categorical variables, the values cannot be compared. Let's say it's not possible to make a rule like `If postal code is higher than "AX2004-P"`.

The values of a nominal variable can be compared if we have another variable to use as a reference (usually an outcome to predict).  

For example, postal code "AX2004-P" is _higher_ than "MA3942-H" because there are more people interested in attending photography lessons.

In addition, **high cardinallity** is an issue in categorical variables, e.g., a `postal code` variable containing hundreds of different values. This book has addressed this in both chapters: handling high categorical variable for <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_descriptive_stats.html" target="blank">descriptive statistics</a> and when we do <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_predictive_modeling.html" target="blank">predictive modelling</a>.

Anyway, you can do the _free test_ of converting all variables into categorical ones and see what happens. Compare the results with the numerical variables. Remember to use some good error measure for the test, like Kappa or ROC statistic, and to cross-validate the results.

<br>

#### Be aware when converting categorical into numerical variables

Imagine we have a categorical variable that we need to convert to numerical. As in the previous case, but trying a different **transformation** assign a different number to each category.

We have to be careful when doing such transformations because we are **introducing order** to the variable. 

Consider the following data example having four rows. The first two variables are `visits` and `postal_code` (this works as either two input variables or `visits` as input and `postal_code` as output).

The following code will show the `visits` depending on `postal_code` transformed according to two criteria:

* `transformation_1`: Assign a sequence number based on the given order.
* `transformation_2`: Assign a number based on the number of `visits`.


```r
# creating data -toy- sample 
df_pc=data.frame(visits=c(10, 59, 27, 33), postal_code=c("AA1", "BA5", "CG3", "HJ1"), transformation_1=c(1,2,3,4), transformation_2=c(1, 4, 2, 3 ))

# printing table
knitr::kable(df_pc)
```



| visits|postal_code | transformation_1| transformation_2|
|------:|:-----------|----------------:|----------------:|
|     10|AA1         |                1|                1|
|     59|BA5         |                2|                4|
|     27|CG3         |                3|                2|
|     33|HJ1         |                4|                3|

```r
library(gridExtra)

# transformation 1
plot_1=ggplot(df_pc, aes(x=transformation_1, y=visits, label=postal_code)) +  geom_point(aes(color=postal_code), size=4)+ geom_smooth(method=loess, group=1, se=FALSE, color="lightblue", linetype="dashed") + theme_minimal()  + theme(legend.position="none") + geom_label(aes(fill = factor(postal_code)), colour = "white", fontface = "bold")
  

# transformation 2
plot_2=ggplot(df_pc, aes(x=transformation_2, y=visits, label=postal_code)) +  geom_point(aes(color=postal_code), size=4)+ geom_smooth(method=lm, group=1, se=FALSE, color="lightblue", linetype="dashed") + theme_minimal()  + theme(legend.position="none") + geom_label(aes(fill = factor(postal_code)), colour = "white", fontface = "bold")
  
# arranging plots side-by-side
grid.arrange(plot_1, plot_2, ncol=2)
```

<img src="figure/data types in machine learning-1.png" title="plot of chunk data types in machine learning" alt="plot of chunk data types in machine learning" width="600px" />

To be sure, nobody builds a predictive model using only four rows; however, the intention of this example is to show how the relationship changes from non-linear (`transformation_1`) to linear (`transformation_2`). This makes things easier for the predictive model and explains the relationship.

This effect is the same when we handle millions of rows of data and the number of variables scales to hundreds. Learning from small data is a right approach in these cases.


<br>

### Discretizing numerical variables 

This process converts data into one category by splitting it into bins. For a fancy definition, we can quote _Wikipedia_: _Discretization concerns the process of transferring continuous functions, models, and equations into discrete counterparts._ 

Bins are also known as buckets or segments. Let's continue with the examples.

#### About the data 

The data contain information regarding the percentage of children that are stunted. The ideal value is zero.

> The indicator reflects the share of children younger than 5 years who suffer from stunting. Children with stunted growth are at greater risk for illness and death.

Data source: <a href="https://ourworldindata.org/hunger-and-undernourishment/#undernourishment-of-children" target="blank">ourworldindata.org, hunger and undernourishment</a>. 

First of all, we have to do a quick **data preparation**. Each row represents a country–year pair, so we have to obtain the most recent indicator per country. 


```r
data_stunting=read.csv(file = "https://raw.githubusercontent.com/pablo14/data-science-live-book/master/data_preparation/share-of-children-younger-than-5-who-suffer-from-stunting.csv", header = T, stringsAsFactors = F)

## ranaming the metric
data_stunting=rename(data_stunting, share_stunted_child=WHO....Share.of.stunted.children.under.5.)

## doing the grouping mentioned before
data_stunting_grouped = group_by(data_stunting, Entity) %>% filter(Year == max(Year)) %>% summarise(share_stunted_child=max(share_stunted_child))
```

The most standard binning criteria are:

<br>




#### Equal range

The range is commonly found in histograms looking at distribution, but is highly susceptible to outliers. To create, for example, four bins, requires the min and max values divided by 4. 



```r
# funModeling contains equal_freq (discretization)
library(funModeling)

# ggplot2 it provides 'cut_interval' function used to split the variables based on equal range criteria
library(ggplot2) 

## Creating equal range variable, add `dig.lab=9` parameter to deactivate scientific notation as with the `cut` function.
data_stunting_grouped$share_stunted_child_eq_range=cut_interval(data_stunting_grouped$share_stunted_child, n = 4)

## The ‘describe’ function from Hmiscpackage is extremely useful to profile data
describe(data_stunting_grouped$share_stunted_child_eq_range)
```

```
## data_stunting_grouped$share_stunted_child_eq_range 
##       n missing  unique 
##     154       0       4 
## 
## [1.3,15.8] (62, 40%), (15.8,30.3] (45, 29%) 
## (30.3,44.8] (37, 24%), (44.8,59.3] (10, 6%)
```

```r
# Plotting the variable
p2=ggplot(data_stunting_grouped, aes(share_stunted_child_eq_range)) + geom_bar(fill="#009E73") + theme_bw()
p2
```

<img src="figure/equal_range_discretization-1.png" title="plot of chunk equal_range_discretization" alt="plot of chunk equal_range_discretization" width="600px" />

The `describe` output tells us that there are four categories in the variable and, between parenthesis/square bracket, the total number of cases per category in both absolute and relative values, respectively. For example, the category `(15.8,30.3]` contains all the cases that have `share_stunted_child` from `15.8` (not inclusive) to `30.3` (inclusive). 
It appears `45` times and represents `29%` of total cases.

<br>

#### Equal frequency

This technique groups the same number of observations using criteria based on percentiles. More information about percentiles at <a href=”http://livebook.datascienceheroes.com/exploratory_data_analysis/annex_1_profiling_percentiles.html” target=”blank”> Annex 1: The magic of percentiles</a> chapter.

The `funModeling` package includes the `equal_freq` function to create bins based on these criteria:


```r
data_stunting_grouped$share_stunted_child_eq_freq=equal_freq(var = data_stunting_grouped$share_stunted_child, n_bins = 4)

## profiling variable 
describe(data_stunting_grouped$share_stunted_child_eq_freq)
```

```
## data_stunting_grouped$share_stunted_child_eq_freq 
##       n missing  unique 
##     154       0       4 
## 
## [ 1.3, 9.5) (40, 26%), [ 9.5,20.8) (37, 24%) 
## [20.8,32.9) (39, 25%), [32.9,59.3] (38, 25%)
```

```r
p3=ggplot(data_stunting_grouped, aes(share_stunted_child_eq_freq)) + geom_bar(fill="#CC79A7") + theme_bw()
p3
```

<img src="figure/equal_frequency_discretization-1.png" title="plot of chunk equal_frequency_discretization" alt="plot of chunk equal_frequency_discretization" width="600px" />

In this case, we select four bins so that each bin will contain an approximate 25% share.

<br>

#### Custom bins

If we already have the points for which we want the segments, we can use the `cut` function.


```r
# parameter dig.lab "disable" scientific notation 
data_stunting_grouped$share_stunted_child_custom=cut(data_stunting_grouped$share_stunted_child, breaks = c(0, 2, 9.4, 29, 100))

describe(data_stunting_grouped$share_stunted_child_custom)
```

```
## data_stunting_grouped$share_stunted_child_custom 
##       n missing  unique 
##     154       0       4 
## 
## (0,2] (5, 3%), (2,9.4] (35, 23%), (9.4,29] (65, 42%) 
## (29,100] (49, 32%)
```

```r
p4=ggplot(data_stunting_grouped, aes(share_stunted_child_custom)) + geom_bar(fill="#0072B2") + theme_bw()
p4
```

<img src="figure/discretization_custom_bins-1.png" title="plot of chunk discretization_custom_bins" alt="plot of chunk discretization_custom_bins" width="600px" />

Please note it’s only needed to define the maximum value per bucket.

In general, we don’t know the minimum nor maximum value. In those cases, we can use the values `-Inf` and `Inf`. Otherwise, if we define a value out of the range, `cut` will assign the `NA` value.

It's good practice to assign the minimum and maximum using a function. In this case, the variable is a percentage, so we know beforehand its scale is from 0 to 100; however, _what would happen if we did not know the range?_

The function will return `NA` for those values below or above the cut points. One solution is to get variable min and max values:



```r
# obtaining the min and max 
min_value=min(data_stunting_grouped$share_stunted_child)
max_value=max(data_stunting_grouped$share_stunted_child)

# `include.lowest=T` is necessary to include the min value, otherwise it will be assigned as NA.
data_stunting_grouped$share_stunted_child_custom_2=cut(data_stunting_grouped$share_stunted_child, breaks = c(min_value, 2, 9.4, 29, max_value), include.lowest = T)

describe(data_stunting_grouped$share_stunted_child_custom_2)
```

```
## data_stunting_grouped$share_stunted_child_custom_2 
##       n missing  unique 
##     154       0       4 
## 
## [1.3,2] (5, 3%), (2,9.4] (35, 23%), (9.4,29] (65, 42%) 
## (29,59.3] (49, 32%)
```

<br>

### Discretization with new data

All of these transformations are made given a training dataset based on the variables’ distributions. Such is the case of equal frequency and equal range discretization. _But what would it happen if new data arrive?_

If a new min or max value appears, then it will affect the bin range in the **equal range** method.
If any new value arrives, then it will move the points based on percentiles as we saw in the **equal frequency** method.

As an example, imagine that in the proposed example we add four more cases with values `88`, `2`, `7` and `3`:


```r
## Simulating that four new values arrive
updated_data=c(data_stunting_grouped$share_stunted_child, 88, 2, 7, 3)

## discretization by equal frequency
updated_data_eq_freq=equal_freq(updated_data,4)

## results in...
describe(updated_data_eq_freq)
```

```
## updated_data_eq_freq 
##       n missing  unique 
##     158       0       4 
## 
## [ 1.3, 9.3) (40, 25%), [ 9.3,20.6) (39, 25%) 
## [20.6,32.9) (40, 25%), [32.9,88.0] (39, 25%)
```

Now we compare with the bins we created before:


```r
describe(data_stunting_grouped$share_stunted_child_eq_freq)
```

```
## data_stunting_grouped$share_stunted_child_eq_freq 
##       n missing  unique 
##     154       0       4 
## 
## [ 1.3, 9.5) (40, 26%), [ 9.5,20.8) (37, 24%) 
## [20.8,32.9) (39, 25%), [32.9,59.3] (38, 25%)
```


**All the bins changed!** Because these are new categories, the predictive model will fail to handle them because they are all new values.

The solution is to save the cut points when we do data preparation. Then, when we run the model on production, we use the custom bin discretization and, thereby, force every new case in the proper category. This way, the predictive model will always _sees_ the same.




<br>

### Final thoughts

As we can see, **there is no free lunch** in discretization or data preparation. How do you think that an _automatic or intelligent system_ will handle all of these situations without human intervention or analysis? 

To be sure, we can delegate some tasks to automatic processes; however, **humans are indispensable** to monitor and adjust the model, giving the correct input data to process.

The assignment of variables as categorical or numerical, the two most used data types varies according to the nature of the data and the selected algorithms as some only support one data type.

The conversion **introduces some bias** to the analysis. A similar case exists when we deal with missing values: <a href="http://livebook.datascienceheroes.com/data_preparation/treating_missing_data.html" target="blank">Handling and Imputation of Missing Data</a>.

When we work with categorical variables, we can change their distribution by re-arranging the categories according to a target variable in order to **better expose their relationship**. Converting a non-linear variable relationship, into one linear. 

<br> 

### Bonus track

Let's go back to the discretization variable section and plot all the transformations we've seen so far:


```r
grid.arrange(p2, p3, p4, ncol = 3)
```

<img src="figure/discretization_methods-1.png" title="plot of chunk discretization_methods" alt="plot of chunk discretization_methods" width="600px" />

The input data is always the same. However, all of these methods **exhibit different perspectives of the same _thing_**.

Some perspectives are more suitable than others for certain situations, such as the use of **equal frequency** for **predictive modeling**.

Although this case is only considering one variable, the reasoning is the same if we have several variables at once, i.e., an `N-dimensional` space.

When we build predictive models, we describe the same bunch of points in different ways as when people give an opinion regarding some object.

