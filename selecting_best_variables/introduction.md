Introduction
===

This chapter will cover three types of plots which aim to understand what are the most correlated numeric variables against a target variable.


* **Overview**:
    + **Analysis purpose**: To identify if the input variable is a good/bad predictor through visual analysis. 
    + **General purpose**: To explain the decision of including -or not- a variable to a model to a *non-analyst* person. 

**Constraint:** Target variable must contain only 2 values. If it has `NA` values, they will be removed.



<br>

* **Key in mind this when using Histograms & BoxPlots** They are nice to see when the variable:
    + Has a good spread -not concentrated on a bunch of _3, 4..6.._ different values, **and**
    + It has not really extreme outliers... _(this point can be treated with `prep_outliers` function present in this package)_

<br>
<br>
