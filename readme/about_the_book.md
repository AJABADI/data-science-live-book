# Data Science Live Book

<br>

### An intuitive and practical approach to data analysis, data preparation and machine learning, suitable for all ages! 🚀

<img src="readme/data_science_live_book_cover.png" width="700px" alt="Data Science Live Book" align="center">

_Author: Pablo "Jausis" Casas._  (https://twitter.com/pabloc_ds)



<br>

## Why this book?

The book will facilitate the understanding of common issues when data analysis and machine learning are done. 

Building a predictive model is as difficult as one line of `R` code: `my_fancy_model=randomForest(target ~ var_1 + var_2, my_complicated_data)`. That's it. 

But, data has its dirtiness in practice. We need to sculp it, just like an artist does, to expose its information in order to find answers (and new questions).

There are many challenges to solve, some data sets requiere more _sculpting_ than others. Just to give an example, random forest does not accept empty values, so what to do then? Do we remove the rows in conflict? Or do we transfom the empty values into other values? **What is the implication**, in any case, to _my_ data?

Despite the empty values issue, we have to face other situations such as the extreme values (outliers) that tend to bias not only the predictive model itself, but the interpretation of the final results. It's common to "try and guess" _how_ the predictive model considers each variable (ranking best variables), and what the values that increase (or decrease) the likelihood of some event to happening (profiling variables) are.

Deciding the **data type** of the variables may not be trivial. A categorical variable _could be_ numerical and viceversa, depending on the context, the data, and the algorithm itself (some of which only handle one data type). The conversion also has its own implications in _how the model sees the variables_.

It is a book about data preparation, data anaylsis and machine learning. Generally in literature, data preparation is not as popular as the creation of machine learning models.

<br>

## The journey towards learning

The book has a highly practical approach, and tries to demonstrate what it states. For example, it says: _"Variables work in groups."_, and then you'll find a code that supports the idea.

Practically all chapters can be copy-pasted and be replicated by the reader to draw their own conclusions. Even more, whenever possible the code or script proposed (in R language) was thought generically, so it could be used in real scenarios, whether research or work.

The book's seed was the `funModeling` *R* library which started having a didactical documentation that quickly turned it into this book. Didactical because there is a difference between using a simple function that plots histograms to profile the target variable (`cross_plot`), and the explanation of how to get to semantical conclusions. The intention is to learn the inner concept, so you can _export that knowledge_ to other languages, such as Python, Julia, etc.

This book, as well as the development of a data project, is not linear. The chapters are related among them. For example, the **missing values** chapter can lead to the **cardinality reduction in categorical variables**. Or you can read the **data type** chapter and then change the way you deal with missing values.

You'll find references to other websites so you can expand your study, _this book is just another step in the learning journey_

<br>

## Is this book for me? Will I understand it?

If you already are in the Data Science field, probably you don't think so. You'll pick the code you need, copy-paste it if you like, and that's it.

But if you are starting a data science career, you'll face a common problem in education: _To have answers to the questions that have not been made._  

For sure you will get closer to the data science world. All the code is well commented so you don't even need to be a programmer. This is the challenge of this book, to try and be friendly when reading, using logic, common sense and intuition. 


### Programming language

You could learn some `R` but it can be tough to learn directly from this book. If you want to learn R programming, there are other books or courses specialized in programming

Time for next section.

<br>

## Will machines and artificial intelligence rule the world?

Although it is true that computing power is being increased exponentially, the machines rebellion is far from happening today.

This book tries to expose common issues when creating and handling predictive models. Not a free lunch. There is also a relationship to _1-click solutions_ and voilà! The predictive system is running and deployed. All the data preparation, transformations, table joins, timing considerations, tuning, _etc_ is solved in one step. 

Perhaps it is. Indeed as time goes by, there are more robust techniques that help us automatize tasks in predictive modeling. But just in case, it'd be a good practice not to trust blindly in black-box solutions without knowing, for example, how the system _picks up the best variables_, _what the inner procedure to validate the model is_, _how it deals with extremes or rare values_, among other topics covered in this book.

If you are evaluating some machine learning platform, some issues stated in this book can help you to decide the best option. Trying to _unbox the black-box_.

It's tough to have a solution that suits all the cases. Human intervention **is crucial** in order to have a successful project. Rather than worry about machines, the point is _what the use of this technology will be_. Technology is _innocent_. It is the data scientist who sets the inputs and gives the model the needed target to learn. Patterns will emerge, and some of them could be harmful for many people. We have to be aware of the final objective, like in any other technologies. 


> The machine is made by man, and it is what man does with it.

(Original quote in Spanish: "La maquina la hace el hombre, y es lo que el hombre hace con ella.")

_By Jorge Drexler (musician, actor and doctor). Extracted from the song "Mi guitarra y vos"_

Maybe, could this be the difference between **machine learning** and **data science**? A machine that learns vs. a human being doing science with data?

An open question.

<br>

## What do I need to start?

In general terms, time and patience. Most of the concepts are independent from the language, but when a technical example is required it is done in <a href="https://cloud.r-project.org" target="blank">**R language**</a>. 

The book uses -at least- the following libraries: `funModeling` (developed for the book), `dplyr`, `Hmisc`, `reshape2`, `ggplot2`, `caret`, `minerva`, `missForest`, `gridExtra`, `mice`, `lock5Data`, `corrplot`, `RColorBrewer` and `infotheo`.

Install any of these by doing: `install.packages("PACKAGE_NAME")`.

The recommended IDE is <a href="https://www.rstudio.com/products/rstudio/download/" target="blank">**Rstudio**</a>.

The book was created with Rstudio, using `knitr` library to create markdown documents (the web-pages in this case), and <a href="https://www.gitbook.com/" target="blank">**GitBook**</a> as the publishing platform.

It's all free and open-source, GitBook, R, Rstudio and this book 🙂.

Hope you enjoy it!

<br>

## How can I contact you? 📩

If you want to say _hello_, contribute by telling that some part is not well explained, suggest a new topic or share some good experience you had applying any concept explained here, you are welcome to drop me an email at: 

pcasas.biz (at) gmail.com. I'm constantly learning so it's nice to exchange knowledge and keep in touch with other colleagues.

My twitter: https://twitter.com/pabloc_ds

**Data Science Heroes Blog**: http://blog.datascienceheroes.com/

Also, you can check the **Github** repositories for both, the book and `funModeling`, so you can report bugs, suggestions, new ideas, etc:
* `funModeling` R package: https://github.com/pablo14/funModeling
* **Data Science Live Book**: https://github.com/pablo14/data-science-live-book

---


* **Book technical reviewer:** 🛠: **Pablo Seibelt** (https://twitter.com/sicarul).
* Art lovely cover 👩‍🎨: Bárbara Muñoz (https://twitter.com/barmercedes_)

_First published at: <a href="http://livebook.datascienceheroes.com">livebook.datascienceheroes.com</a>_

_This book is under <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/" target="blank">Attribution-NonCommercial-ShareAlike 4.0 International</a> license._




