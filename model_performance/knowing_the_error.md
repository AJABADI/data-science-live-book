# Methodological Aspects on Model Validation
## Knowing the Error

### What's this about?

Once we've built a predictive model, how sure we are about it's quality? Did it capture general patterns _-information-_ (excluding the _-noise-_)?




<br>

#### What sort of data?

Unlike <a href='http://livebook.datascienceheroes.com/model_performance/out_of_time_validation.html'>time dependant models</a> in this case you'll have an data's snapshot at certain point of time, no new information will be generated.

For example some health data research from a reduced amount of people, a survey, or some data available on internet for practicing purposes. It's either expensive, not practical, unethical or even impossible to add new cases.

The `heart_disease` data coming in `funModeling` package is such an example.


<br>

### Reducing unexpected behavior

When a model is trained it just sees a part of a reality. It's a sample from a population that cannot be totally seen.

There are lots of ways to validate a model (Accuracy / ROC curves / Lift / Gain / etc). Any of these metrics are **attached to variance**, which implies **getting different values**. If you remove some cases and then fit a new model, you'll see an _slightly_ different value.

Imagine we fit a model and achieve an accuracy of `81`, now remove 10% of the cases, fit a new model, the accuracy now is: `78.4`. **What is the real accuracy?** The one obtained with 100% of data or the other based on 90%? For example if the model will run live on a production environment, it will see **other cases** and the accuracy point will move to a new one.

_So what is the real value? The one to report?_ **Re-sampling** and **cross-validation** techniques will average -based on different sampling and testing criteria- in order to retrieve an approximation to the most trusted value.

<br>

**But why remove cases?**

There is no sense in removing cases like that, but it gets an idea about how sensible the accuracy metric is, remember you're working with a sample from an *_unknown population_*.

> If we'd have a fully deterministic model, a model that contains 100% of all cases we are studying, and predictions were 100% accurate in all cases, we wouldn't need all of this.

> As far as we always analyze samples, we just need to getting closer to the _real and unknown truthness_ of data through repetition , re-sampling, cross-validation, and so on...


<br>

### Let's illustrate this with Cross-Validation (CV)

<img src="k-fold_cross_validation.png" width="400px" alt="Cross-Validation">

_Image credit: Sebastian Raschka_ Ref. [1]

<br>

#### CV short summary

* Splits the data into random groups, let's say `10`, equally sized. These groups are commonly called `folds`, represented by the `'k'` letter.
* Take `9` folds, build a model, and then apply the model to the remaining fold (the one which was left-out). This will return the accuracy metric you want: accuracy, ROC, Kappa, etc. We're using accuracy in this example.
* Repeat this `k` times (`10` in our example). So we'll get `10` different accuracies. Final result will be the average of all of them.

This average will be the one to evaluate if a model is good or not, and also to include it in a report.

<br>

#### Practical example

There 150 rows in the `iris` data frame, using <a href="http://topepo.github.io/caret/index.html">caret package</a> to build a `random forest` with `caret` using `cross-validation` will end up in the -internal- construction of 10 random forest, each one based on 135 rows (9/10 * 150), and reporting an accuracy based on remaining 15 (1/10 * 150) cases. This procedure is repeated 10 times.

This part of the output:

<img src="caret_cross_validation_output.png">

`Summary of sample sizes: 135, 135, 135, 135, 135, 135, ... `, each 135 represents a training sample, 10 in total but the output is truncated.

Rather a single number -the average-, we can see a distribution:

<img src="accuracy_distribution_plot.png" width="200px">

<img src="accuracy_distribution.png" width="200px">

* The min/max accuracy will be between `~0.8` and `~1`.
* The mean is the one reported by `caret`.
* 50% of times it will be ranged between `~0.93 and ~1`.

<br>

### But what is Error?

The sum of **Bias**, **Variance** and the **_unexplained error_** -inner noise- in data, or the one that the model will never be able to reduce.

These 3 elements represent the error reported.

#### What is the nature of Bias and Variance ?

When the model doesn't work well, there may be several causes:

* **Model too complicated**: Let's say we have lots of input variables, which is related to **high variance**. The model will overfit on training data, having a low accuracy on unseen data due to its particularization.
* **Model too simple**: On the other hand, the model may not be capturing all the information from the data due to its simplicity. This is related to **high bias**.
* **Not enough input data**: Data forms shapes in a n-dimensional space (where `n` is all the input+target variables). If there are not enough points, this shape is not developed well enough.

More info here in [4].

<img src="bias_variance.png" width="300px">

_Image credit: Scott Fortmann-Roe_ [3]

<br>

Bias and variance are related in the sense that if one goes down the other goes up, so it's a **tradeoff** between them.

* **Bootstrapping** is mostly used when estimating a parameter.
* **Cross-Validation** is the choice when choosing among different predictive models.
* Coming soon a post in Data Science Heroes Blog explaining their differences

Note: For a deeper coverage about bias and variance please go to [3] and [4] at the bottom of the page.

### Any advices on practice?

It depends on the data, but it's common to find examples cases `10 fold CV`, plus repetition: `10 fold CV, repeated 5 times`. Other: `5 fold CV, repeated 3 times`.

And using the average of the desired metric. It's also recommended to use the `ROC` for being less biased to unbalanced target variables.

Since these validation techniques are **time consuming**, consider choosing a model which will run fast, allowing model tunning, testing different configurations, trying different variables in a "short" amount of time. **<a href="https://www.stat.berkeley.edu/~breiman/RandomForests/" target="blank">**Random Forest</a>** are an excellent option which gives **fast** and **accurate** results. Ref. [2].

Another good option is: **gradient boosting machines**, it has more paramaters to tune than random forest, but at least in R it's implementation works really fast.

#### Going back to bias and variance

* Random forest focuses on decreasing bias, while...
* Gradient boosting machine focuses on decreasing variance. [5]

<br>


### Don't forget: Data Preparation

Tweaking input data by transforming and cleaning it, will impact on model quality. Sometimes more than optimizing the model through its parameters.
The <a href="http://livebook.datascienceheroes.com/data_preparation/introduction.html">Data Preparation</a> chapter of this book is under heavy development. Coming soon.


### Final toughts

* Validating the models through re-sampling / cross-validation helps us to estimate the "real" error present in the data. If the model will run in the future, that will be the expected error to have.
* Another advantage is **model tuning**, avoiding the overfitting in selecting best parameters for certain model, <a href="https://topepo.github.io/caret/model-training-and-tuning.html" target="blank"> Example in `caret`</a>. The equivalent in **Python** is included in <a href="http://scikit-learn.org/stable/modules/cross_validation.html"  target="blank">Scikit Learn</a>.
* The best test is the one made by you, suited to your data and needs. Try different models and analyze the tradeoff between time consumption and any accuracy metric.

> These re-sampling techniques could be among the powerful tools behind the sites like stackoverflow.com or collaborative open-source software. To have many opinions in order to produce a less-biased solution.


<br>

#### Further reading
 
* Tutorial: <a href="http://www.milanor.net/blog/cross-validation-for-predictive-analytics-using-r" target="blank">Cross validation for predictive analytics using r</a>
* Tutorial by Max Kahn (caret's creator): <a href="http://appliedpredictivemodeling.com/blog/2014/11/27/vpuig01pqbklmi72b8lcl3ij5hj2qm"  target="blank">Comparing Different Species of Cross-Validation</a> 
* The cross-validation approach can also be applied to time dependant models, check the other chapter: <a href="http://livebook.datascienceheroes.com/model_performance/models_time_dependant.html" target="blank">Methodological aspects on time dependant models</a>.

<br>

**References:**

* [1] Image source: <a href="http://sebastianraschka.com/faq/docs/evaluate-a-model.html" target="blank">Machine Learning FAQ by Sebastian Raschka</a>
* [2] More on Random Forest overall performance: <a href="http://jmlr.csail.mit.edu/papers/volume15/delgado14a/delgado14a.pdf"  target="blank">Do we Need Hundreds of Classiers to Solve Real World Classication Problems?</a>
* <a href="http://robjhyndman.com/hyndsight/crossvalidation/">Why every statistician should know about cross-validation?</a> by Rob Hyndman, creator of `forecast` package.
* [3] Image source: <a href="http://scott.fortmann-roe.com/docs/BiasVariance.html" target="blank">Understanding the Bias-Variance Tradeoff</a>. It contains an intutitive way of understanding error through bias and variance through a animation.
* [4] <a href="http://www.kdnuggets.com/2015/06/machine-learning-more-data-better-algorithms.html" target="blank">In Machine Learning, What is Better: More Data or better Algorithms</a> 
* [5] <a href="http://stats.stackexchange.com/questions/173390/gradient-boosting-tree-vs-random-forest" target="blank">Gradient boosting machine vs random forest</a>






