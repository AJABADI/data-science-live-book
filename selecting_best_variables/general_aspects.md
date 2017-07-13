

# General Aspects in Selecting Best Variables

## What is this about?

This chapter covers the following topics:

* The best variables ranking from conventional machine learning algorithms, either predictive or clustering.
* The nature of selecting variables with and without predictive models.
* The effect of variables working in groups (intuition and information theory).
* Exploring the best variable subset in practice using R.

_Selecting the best variables is also known as feature selection, selecting the most important predictors, selecting the best predictors, among others._

<img src="dark_matter_simulation.png" width="250px" alt="The Millennium Simulation Project">

_Image: Is it a neural network? Nope. Dark matter, from the "The Millennium Simulation Project"._

<br>

## Intuition

Selecting the best variables is like doing a summary of a story, we want to focus on those few details that best describe what we're talking about. The balance threads between talking _too much_ about unnecessary details (overfitting) and talking _too little_ about the essence of the story (underfitting).

Another example can be the decision of buying a new laptop: _what are the features that we care about the most? Price, color and shipping method? Color and battery life? Or just price?_

From the **Information Theory** point of view -a key point in machine learning-, the data that we are working on has **entropy** (chaos). When we select variables, we are are reducing the entropy of our system by adding information. 

<br>

## The "best" selection?

The chapter says "best", but we'd better mention a conceptual point, in general terms _there is no unique best variable selection._ 

To start from this perspective is important, since in the exploration of many algorithms that _rank_ the variables according to their predictive power, we can find different -and similar- results. That is:

* Algorithm 1 has chosen as the best variable `var_1`, followed by `var_5` and `var_14`.
* Algorithm 2 did this ranking: `var_1`, `var_5` and `var_3`.

Let's imagine, based on algorithm 1, the accuracy is 80%, while the accuracy based on algorithm 2 is 78%. Considering that every model has its inner variance, the result can be seen as the same. 

This perspective can help us to reduce time in pursuing the perfect variable selection.

However going to the extremes, there will be a set of variables that will rank high across many algorithms, and the same goes for those with little predictive power. After several runs most reliable variables will emerge quickly, so: 

**Conclusion**: If results are not good the focus should be on improving and checking the **data preparation** step. _The next section will exemplify it._ 

<br>

### Going deeper into variable ranking

It's quite common to find in literature and algorithms, that covers this topic an univariate analysis, which is a ranking of variables given a particular metric.

We're going to create two models: random forest and gradient boosting machine (GBM) using `caret` R package to cross-validate the data. Next, we'll compare the best variable ranking that every model returns.


```r
library(caret)
library(funModeling)
library(dplyr)

## Excluding all NA rows from the data, in this case, NAs are not the main issue to solve, so we'll skip the 6 cases which have NA (or missing values).
heart_disease=na.omit(heart_disease)

## Setting a 4-fold cross-validation
fitControl = trainControl(method = "cv",
                           number = 4,
                           classProbs = TRUE,
                           summaryFunction = twoClassSummary)

## Creating the random forest model, finding the best tuning parameter set
set.seed(999)
fit_rf = train(x=select(heart_disease, -has_heart_disease, -heart_disease_severity),
             y = heart_disease$has_heart_disease,
             method = "rf",
             trControl = fitControl,
             verbose = FALSE,
             metric = "ROC")

## Creating the gradient boosting machine model, finding the best tuning parameter set
fit_gbm = train(x=select(heart_disease, -has_heart_disease, -heart_disease_severity),
             y = heart_disease$has_heart_disease,
             method = "gbm",
             trControl = fitControl,
             verbose = FALSE,
             metric = "ROC")
```

Now we can proceed with the comparison. 

The columns `importance_rf` and `importance_gbm` represent the importance measured by each algorithm. Based on each metric, there are `rank_rf` and `rank_gbm` which represent the importance order, finally `rank_diff` (`rank_rf` - `rank_gbm`) represents how different each algorithm rank the variables.



```r
## Here we manipulate to show a nice the table described before
var_imp_rf=data.frame(varImp(fit_rf, scale=T)["importance"]) %>% dplyr::mutate(variable=rownames(.)) %>% dplyr::rename(importance_rf=Overall) %>% dplyr::arrange(-importance_rf) %>% dplyr::mutate(rank_rf=seq(1:nrow(.))) 

var_imp_gbm=as.data.frame(varImp(fit_gbm, scale=T)["importance"])  %>% dplyr::mutate(variable=rownames(.)) %>% dplyr::rename(importance_gbm=Overall) %>% dplyr::arrange(-importance_gbm) %>% dplyr::mutate(rank_gbm=seq(1:nrow(.)))                                                                                                                             
final_res=merge(var_imp_rf, var_imp_gbm, by="variable")

final_res$rank_diff=final_res$rank_rf-final_res$rank_gbm

# Printing the results!
final_res
```
<img src="ranking_best_vars_comparison.png" alt="Comparison across two methods for variable ranking">


We can see that there are variables which are not important at all to both models (`fasting_blood_sugar`). There are others that maintain a position at the top of importance like `chest_pain` and `thal`.

Different predictive model implementations have their criteria to report what are the best features, according to that particular model. This ends up in different ranking across different algorithms. _More info about the inner importance metrics  can be found at the <a href=https://topepo.github.io/caret/variable-importance.html" target="blank">caret documentation</a>._

Even more, in tree based models like GBM and Random Forest there is a random component to picking up variables, and the importance is based on prior -and automatic- variable selection when building the trees. The importance of each variable depends on the others, not only on its isolated contribution: **Variables work in groups**. We'll back on this later on this chapter.

Although the ranking will vary from algorithm to algorithm, in general terms there is a correlation between all of these results as we mentioned before. 

**Conclusion:** Every ranking list is not the _"final truth"_, it gives us orientation about where the information is. 

<br>


## The nature of the selection

There are two main approaches when doing variable selection:

**Predictive model dependent**: 

Like the ones we saw before, this is the most common. The model will rank variables according to one intrinsic measure of accuracy. In tree-based models, metrics such as information gain, Gini index, node impurity. Ref [4], [5].

**Not predictive model dependent**: 

This is interesting since they are not as popular as the other ones, but they are proved to perform really well in areas related to genomic data. They need to find those _relevant_ genes (input variable) that are correlated with some disease, like cancer (target variable).

Data from this area is characterized by having a huge number of variables (in the order of thousands), which is much bigger than problems in other areas.

One algorithm to perform this is <a href="http://home.penglab.com/proj/mRMR/" target="blank">mRMR</a>, acronym for Minimum Redundancy Maximum Relevance Feature Selection. It has its own implementation in R in <a href="https://cran.r-project.org/web/packages/mRMRe/vignettes/mRMRe.pdf" target="blank">mRMRe</a> package.

<br> 

## Improving variables

Variables can increase their predictive power by treating them. 

This book covers by now:

* <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_predictive_modeling.html" target="blank">Improvement of categorical variables</a>.
* Reducing the noise in numerical variables through binning in the <a href="http://livebook.datascienceheroes.com/selecting_best_variables/cross_plot.html"  target="blank">cross_plot</a> function.
* <a href="http://livebook.datascienceheroes.com/data_preparation/outliers_treatment.html" target="blank">Preparing outliers</a> for predictive modeling.
* <a href="http://livebook.datascienceheroes.com/data_preparation/treating_missing_data.html" target="blank">Analysis, Missing Data: Analysis, Handling and Imputation</a> for predictive modeling. 

_And more to come..._

<br>

## Cleaning by domain knowledge

It's not related to algorithmic procedures, but to the area from which the data comes.

Imagine data coming from a survey. This survey has one year of history, and during the first three months there was no good process control. When inserting data users could type whatever they wanted. Variables during this period will probably be spurious. 

It's easy to recognize it when during a given period of time, the variable comes empty, null or with extreme values. 

We should then ask a question: 

_Is this data reliable?_ Keep in mind the predictive model will learn _as a kid_, it will not judge the data, just learn from it. If data is spurious in a given period of time, then we may remove these input cases.

To go further on this point, we should do a deeper exploratory data analysis. Both numerically and graphically.

<br>

## Variables work in groups

<img src="variable_groups.png" width="300px" alt="Variable work in groups">

When selecting the _best_ variables, the main aim is to get those variables which carry the most information regarding a target, outcome or dependent variable. 

A predictive model will find its weights or parameters based on its 1 to 'N' input variables.

Variables usually don't work isolated when explaining an event. Quoting Aristotle: 

> “The whole is greater than the sum of its parts.” 

This is also true when selecting the _best_ features: 

_Building a predictive model with two variables may reach a higher accuracy than the models built with only one variable._

For example: Building a model based on variable `var_1` could lead to an overall accuracy of 60%. On the other hand, building a model based on `var_2` could reach an accuracy of 72%. But when we combine these two `var_1` and `var_2` variables, we could achieve an accuracy above 80%.

<br>

### Example in R: Variables working in groups

<img src="aristotle.png" width="300px" alt="Aristotle: philosopher and data scientist">

The following code illustrates what Aristotle said _some_ years ago. 

It creates 3 models based on different variable subsets:

* model 1 is based on `max_heart_rate` input variable
* model 2 is based on `chest_pain` input variable
* model 3 is based on `max_heart_rate` **and** `chest_pain` input variables

Each model returns the metric ROC, and the result contains the improvement of considering the two variables at the same time vs. taking each variable isolated.


```r
library(caret)
library(funModeling)
library(dplyr)

## setting cross-validation 4-fold
fitControl = trainControl(method = "cv",
                          number = 4,
                          classProbs = TRUE,
                          summaryFunction = twoClassSummary)

create_model<-function(input_variables) {
  ## create gradient boosting machine model based on input variables
  fit_model = train(x=select(heart_disease, one_of(input_variables)),
              y = heart_disease$has_heart_disease,
              method = "gbm",
              trControl = fitControl,
              verbose = FALSE,
              metric = "ROC")
  
  # returning the ROC as the performance metric
  max_roc_value=max(fit_model$results$ROC)
  return(max_roc_value)
}

roc_1=create_model("max_heart_rate")
roc_2=create_model("chest_pain")
roc_3=create_model(c("max_heart_rate", "chest_pain"))

avg_improvement=round(100*(((roc_3-roc_1)/roc_1)+((roc_3-roc_2)/roc_2))/2,2)
avg_improvement_text=sprintf("Average improvement: %s%%", avg_improvement)

results=sprintf("ROC model based on 'max_heart_rate': %s.; based on 'chest_pain': %s; and based on both: %s", round(roc_1,2), round(roc_2,2), round(roc_3, 2))

# printing the results!
cat(c(results, avg_improvement_text), sep="\n\n")
```

```
## ROC model based on 'max_heart_rate': 0.74.; based on 'chest_pain': 0.76; and based on both: 0.82
## 
## Average improvement: 8.93%
```

<br>

### Tiny example (based on Information Theory)

Consider the following _big data_ table 😜. 4 rows, 2 input variables (`var_1`, `var_2`) and one outcome (`target`):

<img src="variables_work_in_gropus.png" alt="Toy data showing the power of combining two variables" width="250">

If we build a predictive model based on `var_1` only, what it will _see_?, the value `a` is correlated with output `blue` and `red` in the same proportion (50%):

* If `var_1='a'` then likelihood of target='red' is 50% (row 1)
* If `var_1='b'` then likelihood of target='blue' is 50% (row 2)

_Same analysis goes for `var_2`_

When the same input is related to different outcomes it's defined as **noise**. The intuition is the same as one person telling us: _"Hey it's going to rain tomorrow!"_, and another one saying: _"For sure tomorrow it's not going to rain"_.  
We'd think... _"OMG! do I need the umbrella or not?"_
 😱
Going back to the example, taking the two variables at the same time, the correspondence between the input and the output in unique: "If `var_1='a'` and `var_2='x'` then the likelihood of being `target='red'` is 100%". You can try other combinations. 

**Summing-up:** 

That was an example of **variables working in groups**, considering `var_1` and `var_2` at the same time increases the predictive power.  

Nonetheless, it's a deeper topic to cover, considering the last analysis; how about taking an `Id` column (every value is unique) to predict something? The correspondence between input-output will also be unique... but is it a useful model? There'll be more to come about information theory in this book.

<br>

### Conclusions  

* The proposed R example based on `heart_disease` data shows an average **improvement of 9%** when considering two variables at a time, not too bad. 😉 This percentage of improvement is the result of the variables working in groups.
* This effect appears if the variables contain information, such is the case of `max_heart_rate` and `chest_pain` (or `var_1` and `var_2`). 
* Putting **noisy variables** next to good variables **will usually affect** overall performance.
* Also the **work in groups** effect is higher if the input variables **are not correlated between** them. This is difficult to optimize in practice. More on this on the next section...


<br>

## Correlation between input variables

The ideal scenario is to build a predictive model with only variables not correlated between them. In practice, it's complicated to keep such a scenario for all variables. 

Usually there will be a set of variables that are not correlated between them, but also there will be others that have at least some correlation.

**In practice** a suitable solution would be to exclude those variables with a **remarkably high-level** of correlation.

Regarding how to measure correlation. Results can be highly different based on linear or non-linear procedures. More info at the <a href="http://livebook.datascienceheroes.com/selecting_best_variables/correlation.html">correlation chapter</a>.

<br>

_What is the problem with adding correlated variables?_

The problem is that we're adding complexity to the model: it's usually more time-consuming, harder to understand, less explainable, less accurate, etc. This is an effect we reviewed in <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_predictive_modeling.html#dont-predictive-models-handle-high-cardinality-part-2">reducing cardinality in categorical variables</a>. 


The general rule would be: Try to add the top N variables that are correlated with the output but not correlated between them. This leads us to the next section. 


<br>

## Keep it simple

<img src="fractals_nature.png" alt="Nature operates in the shortest way possible. -Aristotle." width="250px">

> Nature operates in the shortest way possible. -Aristotle.

The principle of **Occam's razor**: Among competing hypotheses, the one with the fewest assumptions should be selected.

Re-interpreting this sentence for machine learning, those "hypotheses" can be seen as variables, so we've got: 

**Among different predictive models, the one with fewest variables should be selected.**

Of course, there is also the trade-off of adding-substracting variables and the accuracy of the model. 

A predictive model with a _high_ number of variables will tend to do **overfitting**. While on the other hand, a model with a _low_ number of variables will lead to doing **underfitting**.

The concept of _high_ and _low_ is **highly subjective** to the data that is being analyzed. In practice, we may have some accuracy metric, for example, the ROC value. i.e. we would see something like:

<img src="variable_selection_table.png" alt="Quantity of variables vs ROC value trade-off"> 

The last picture shows the ROC accuracy metric given different subsets of variables (5, 10, 20, 30 and 58). Each dot represents the ROC value given a certain number of variables used to build the model.

We can check that the highest ROC appears when the model is built with 30 variables. If we based the selection only on an automated process, we might be choosing a subset which tends to overfit the data. This report was produced by library `caret` in R (ref. [2]), but is analogous to any software.

Take a closer look at the difference between the subset of 20 and the 30; there is only an improvement of **1.8%** -from 0.9324 to 0.95- choosing **10 more variables.** In other words: _Choosing 50% more variables will impact in less than 2% of improvement._

Even more, this 2% may be an error margin given the variance in the prediction that every predictive model has [3].

**Conclusion:**

In this case, and being consequent with Occam's Razor principle, the best solution is to build the model with the subset of 20 variables.

Explaining to others -and understanding- a model with 20 variables is easier than the similar one with 30.

<br> 

## Variable selection in Clustering?

<img src="cluster.png" width="300px" alt="feature engineering in clustering"> 

This concept usually appears only in predictive modeling, i.e. having some variables to predict a target one. In clustering there is no target variable, we let the data speak, and the natural segments arise according to some distance metric.

However, **not every variable contributes in the same way to the dissimilarity in the cluster model**. Keeping it brief, if we have 3 clusters as output, and we measure the average of each variable, we expect to have these averages _quite_ dissimilar between them, right? 

Having built 2 cluster models, in the first one the averages of the `age` variable is 24, 33 and 26 years; while on the second one we have: 23, 31 and 46. In the second model the variable `age` is having more variability, thus it is more relevant to the model. 

This was just an example considering two models, but it's the same considering just one. Those variables with **more distance** across averages will tend to **define better** the cluster than the others. 

Unlike predictive modeling, in clustering _less important_ variables shouldn't be removed, those variables aren't important in that particular model, but they could be if we build another one with other parameters. The cluster models' quality is highly subjective.

Finally, we could run, for example, a random forest model with the cluster as a target variable and in this way quickly collect the most important variables.

<br>

## Selecting the best variables in practice 

### The short answer 

Pick up the top _N_ variables from the algorithm you're using and then re-build the model with this subset. Not every predictive model retrieves variable rankings, but if it does, use the same model (for example gradient boosting machine) to get the ranking and to build the final model.

For those models like k-nearest neighbors which don't have a built-in select best features procedure, it's valid to use the selection of another algorithm. It will lead to better results than using all the variables. 

<br>

### The long answer 

* When possible, **validate** the list with someone who knows about the context, the business or the data source. Either for the top _N_ or the bottom _M_ variables. As regards those _bad_ variables, we may be missing something in the data munging that could be destroying their predictive power.
* Understand each variable, its meaning in context (business, medical, other).
* Do **exploratory data analysis** to see the distributions of the most important variables regarding a target variable, _does the selection make sense?_ If the target is binary then the function <a href="http://livebook.datascienceheroes.com/selecting_best_variables/cross_plot.html" target="blank">cross_plot</a> can be used. 
* Does the average of any variable _significantly_ change over time? Check for abrupt changes in distributions.
* Suspect about high cardinality top-ranked variables (like postal code, let's say above +100 categories). More information at <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_predictive_modeling.html" target="blank">High Cardinality Variables in Predictive Modeling</a> chapter.
* When making the selection -as well as a predictive modeling-, try and use methods which contain some mechanism of re-sampling (like bootstrapping), and cross-validation. More information in Ref. [3].
* Try other methods to find **groups of variables**, like the one mentioned before: mRMR.
* If the selection doesn't meet the needs, try creating new variables, you can check the **data preparation** chapter. Coming soon: Feature engineering chapter.

<br>

### Generate your own knowledge

It's difficult to generalize when the nature of the data is so different, from **genetics** in which there are thousands of variables and a few rows, to web-navigation when new data is coming all the time.

The same applies to the objective of the analysis. Is it to be used in a competition where precision is highly necessary? Perhaps the solution may include more varcorrelated iables thn comparison to with ad-hoc study in which the primary goal is a simple explanation.

There is no one-size-fits-all answer to face all possible challenges; you'll find powerful insights using your experience. It's just a matter of practice. 



<> 

**References:**

* [1] <a href="https://en.wikipedia.org/wiki/Occam's_razor#Probability_theory_and_statistics">Occam's razor in statistics</a>.
* [2] <a href="https://topepo.github.io/caret/recursive-feature-elimination.html">Recursive feature elimination in caret</a>
* [3] It is covered in the <a href="http://livebook.datascienceheroes.com/model_performance/knowing_the_error.html">Knowing the error</a> chapter.
* [4] Understanding <a href="http://stackoverflow.com/questions/1859554/what-is-entropy-and-information-gain" target="blank">Entropy and Information Gain</a>.
* [5] Understanding the <a href="http://stats.stackexchange.com/questions/197827/how-to-interpret-mean-decrease-in-accuracy-and-mean-decrease-gini-in-random-fore" target="blank">accuracy and gini index</a> used in random forest variable ranking.


