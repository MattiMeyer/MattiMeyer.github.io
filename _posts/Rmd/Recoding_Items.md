---
layout: post
title: Recoding items in psychological questionnaires using R
subtitle: Psychology and R, R for beginners
---


In most psychological questionnaires are reversed items, usually to test if people filled it correctly and read the items. But when we want to analyze the results of the questionnaire we need to be sure, that this negative items are reversed.
Let's look at an example. I've created four items fo check if somebody loves R (sure everybody does, but why not):


Item  |I disagree a lot |I disagree a little | I agree a little | I agree a lot
------|----------------|---------------------|------------------|----------------
I like R a lot        |1|2|3|4
In ggplot2 we believe |1|2|3|4
Never heard of R (it’s just a letter?) |1|2|3|4
SPSS as it's best |1|2|3|4

As you can see there are two items that are negative. So if somebody answers this two items on a high rank, he doesn't love R (sacrilege), but if we calculate the mean we would interprete a high mean combined with the first two items as a sign for R love. So we have to reverse the items.

##The data
To visualize this I create four fake results in a data frame (recognize that we mark negative items with a R at the end):


~~~
loveR <- data.frame(item1=c(1,2,4,3),item2=c(2,1,3,3),item3R=c(4,4,1,1),item4R=c(3,4,2,1))

loveR
~~~

~~~
##   item1 item2 item3R item4R
## 1     1     2      4      3
## 2     2     1      4      4
## 3     4     3      1      2
## 4     3     3      1      1
~~~

##ifelse
There are some possibilities to reverse this items, some people maybe use the *ifelse* function:

~~~
##the items before ifelse
loveR[3:4]
~~~

~~~
##   item3R item4R
## 1      4      3
## 2      4      4
## 3      1      2
## 4      1      1
~~~

~~~
items_rev <- ifelse(loveR[3:4]==1,4,ifelse(loveR[3:4]==2,3,ifelse(loveR[3:4]==3,2,ifelse(loveR[3:4]==4,1,NA))))

##items after ifelse
items_rev
~~~

~~~
##      item3R item4R
## [1,]      1      2
## [2,]      1      1
## [3,]      4      3
## [4,]      4      4
~~~

But for me it looks too complicated and maybe somebody who wants to read my code later, won't understand a word.

##recode

In R there is a package for everything, so there is function in the _car_ package that is build for recoding reversed items, called *recode*. By using such a small item set it is unecessary, but if we have more than hundred of items, it can be difficult to find them! So we will use dplyr to get the items with an R at the end:

~~~
library(dplyr)

itemsR <- select(loveR, ends_with("R"))
~~~

This trick seems here very poor, but when you have many items and have good items names it can safe you hours of searching for negative items in your data.
So let`s reverse them (recognize the ; in the function, which is unusual for a R function):

~~~
library(car)

items_3 <- recode(itemsR$item3R,"1=4;2=3;3=2;4=1")

loveR$item3R
~~~

~~~
## [1] 4 4 1 1
~~~

~~~
items_3
~~~

~~~
## [1] 1 1 4 4
~~~

As you can see we reversed item 3 very easily and anybody can redo and understand what we have done. But we've just done it with one item. We should do it with all:

~~~
items_rev <- recode(itemsR[1:2],"1=4;2=3;3=2;4=1")
~~~

~~~
Error in :  recode(itemsR[1:2], "1=4;2=3;3=2;4=1") : 
  (list) object cannot be coerced to type 'double'
~~~
Oh there is one of this R errors, which doesn’t give us a hint what to do! But I'll show you the problem. There is a difference in the output in the way you choose the items:

~~~
itemsR$item3R
~~~

~~~
## [1] 4 4 1 1
~~~

~~~
itemsR[1]
~~~

~~~
##   item3R
## 1      4
## 2      4
## 3      1
## 4      1
~~~

To resolve this we will use the *apply* function. For people who have not heard about it, apply is build like this: `apply(X,MARGIN,FUN)`. As the X you can choose a data frame and for MARGIN you can choose 1 or 2, while 1 stands for the rows and 2 for the column. So you choose between the rows or the columns of a data frame and use a function on it. I will show it to you with calculating the mean:

~~~
apply(itemsR,2,mean)
~~~

~~~
## item3R item4R 
##    2.5    2.5
~~~

Understood? So for recoding it goes a step farther, because we must build our own  `function(x)`, while x is the X in the apply function, therefore our negative items:

~~~
items_rev <- apply(itemsR, 2, function(x) recode(x,"1=4;2=3;3=2;4=1"))

items_rev
~~~

~~~
##      item3R item4R
## [1,]      1      2
## [2,]      1      1
## [3,]      4      3
## [4,]      4      4
~~~
So we reversed our items and we can build a data frame with all four items:

~~~
loveR_rev <- cbind(loveR[1:2], items_rev)

loveR_rev
~~~

~~~
##   item1 item2 item3R item4R
## 1     1     2      1      2
## 2     2     1      1      1
## 3     4     3      4      3
## 4     3     3      4      4
~~~

Done! We can start to analyze the data set!
