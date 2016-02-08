---
layout: post
title: ANOVA vs. Regression
subtitle: Evil statistic stuff and much more evil statistic stuff, psychology
author: Matti Meyer
---

Did you ever heard of ANOVA and Regression?   
If you have heard of them, there are some possibilities why you have heard of them. The first I can imagine, is that you have nothing to do with your life and waste it with such terrible things and the second possibilitie is that you are a psychology student (which is nearly the same as the first one).   
If you are a psychology student like me you have maybe heard from ANOVA and Regression as two seperated things and your brain build up two areas: one for ANOVA and the other one for Regression. But that is a poor status quo, because both are the same and it will become easier for you if you can connect this two lonely brain areas to one big happy smiling brain area, just waiting for more statistic stuff coming in (If you are afraid of brain damages do not read any farther!).   

Maybe you are wondering why you have learned this two methods seperated? This has some historical background, which you can read in [Jacob Cohen's article "Multiple Regression as a general data-analytic system" (1968)](http://www.unt.edu/rss/class/Jon/MiscDocs/Cohen_1968.pdf). In a nutshell you can say that there were the experimental science using ANOVA on the one side and behavioral science using Regression on the other side. 

In this post I would like to show you that ANOVA and Regression are the same using R and statistic! So if you you don't want to read the statistic stuff you can read the R stuff, because there you see the similarities very clearly and the other way around. 
I would like to begin with the t-test, because it is better to get an overview. 

###Packages you need to get the R-stuff running

~~~
library(multcomp)
~~~

###Data

Probably you know that we compare two means from two groups with a t-test, so we need two groups and data to simulate one. My fellow students gave me a great idea for an example. They said that there is no other workshop like the R workshop that awakes thoughts of hurting themselfes like knocking the forehead down on the table. Because I'm a very sceptical person I thought that there must be one workshop that is just harder than the R workshop and registered some others of my fellow students for a statistic workshop! (Please do not worry, the following data is simulated. For this post no student or animal has been harmed or tortured with R and statistic!). So after each workshop my imaginary fellow students had to answer one question: From 1 to 7, how much do you want to hurt yourself?

So let's simulate such a data frame in R:


~~~
set.seed(23) ##use this to get the same data.frame as me
data_t <- data.frame(
  workshop=sort(sample(1:2,50,replace = TRUE,prob=c(25,25))),
  hurt=sample(1:7,50,replace = TRUE)
  )
~~~

As you can see there is one variable numbered as 1 and 2. We have to change this vector into a factor, because 1 stands for the R Workshop and 2 for the statistic Workshop.


~~~
data_t$workshop <- factor(data_t$workshop,levels = c(1,2),labels = c("R Workshop","Statistic Workshop"))

head(data_t)
~~~

~~~
##     workshop hurt
## 1 R Workshop    4
## 2 R Workshop    6
## 3 R Workshop    3
## 4 R Workshop    6
## 5 R Workshop    7
## 6 R Workshop    1
~~~

~~~
tail(data_t)
~~~

~~~
##              workshop hurt
## 45 Statistic Workshop    4
## 46 Statistic Workshop    3
## 47 Statistic Workshop    5
## 48 Statistic Workshop    4
## 49 Statistic Workshop    3
## 50 Statistic Workshop    1
~~~

Now we have two groups with avaliable data and can start to analyze the data!

###The usual way: t-test

Fully excited of the result of our little study we can do a t-test, to get the results. Remember that we want to predict the thougths of hurting themselves with the criterion of the workshop in which the person participated.   
If you wonder of the argument *var.equal* in the *t.test* command, this means that we assume that the variances are equal. If we would change it on FALSE than a Welch test would be calculated, which corrects the _t_ and _p_ value in a way that we can't compare it later. Usually you would have to do a Levene-test for testing this, but we won't interprete the results of this test and just use it to get an example:


~~~
t.test(data_t$hurt ~ data_t$workshop,var.equal=TRUE)
~~~

~~~
## 
## 	Two Sample t-test
## 
## data:  data_t$hurt by data_t$workshop
## t = 0.70984, df = 48, p-value = 0.4812
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.8004041  1.6739673
## sample estimates:
##         mean in group R Workshop mean in group Statistic Workshop 
##                         4.103448                         3.666667
~~~

Please remember the _t-value_= 0.70 and the _p-value_= 0.48.

### The unusual way: linear Regression

Now we've done it the very normal way, but there is another way of doing this. Please remember the normal equation of linear regression which looks like:
$$ y_i = (b_0 + b_1 x_i) + \epsilon_i $$

Like in linear regression we also want to predict the wish of hurting themselves with the group variable, so we can change the equation like this:
$$ hurt = (b_0 + b_1 \text{group}_i) + \epsilon_i $$

As you can see our dependent variable *hurt* is on the left side and is predicted from the intercept *b*0, the weighting of the predictor *b*1 and the predictor *group*. 
If you are a little bit afraid of this equation look at the *group* variable. This is maybe the most uncommon for you, but it is really a cute variable, because it can just have to manifestations as 0 for the R Workshop and 1 as the statistic Workshop. Why 0 and 1? This is called *dummy coding*. We determine the R Workshop as the baseline, because we know that it is the worst for all students and want to compare it with the other Workshop. You can imagine it like a control group which we want to compare to experimental effects.

We just can visualize it if we set *group* as 0 in the equation. Because of this *b*1 will also be 0, the *hurt* variable becomes the mean of the people in the R Workshop, and we can ignore the eror term: 

$$ \overline{X}_R = b_0 + (b_i * 0)$$
$$ \overline{X}_R = b_0 $$

Now we can calculate the intercept *b*0 when we calculate the mean of people in the R Workshop: 

~~~
mean(data_t$hurt[data_t$workshop == "R Workshop"])
~~~

~~~
## [1] 4.103448
~~~

$$ b_0 = 4.1 $$

The next step is to use the intercept *b*0 to calculate the rest of the equation, by setting *group* to 1, which stands for the statistic Workshop: 

$$ \overline{X}_\text{stat} = b_0 + (b_1 * 1)$$
$$ \overline{X}_\text{stat} = \overline{X}_R + b_1$$
$$ \overline{X}_\text{stat} - \overline{X}_R = b_1$$

Here we also need the mean of people done the statistic Workshop:

~~~
mean(data_t$hurt[data_t$workshop == "Statistic Workshop"])
~~~

~~~
## [1] 3.666667
~~~

$$3.7 - 4.1 = b_1$$
$$-0.4 = b_1$$

We can rewrite our first equation of the whole regression term as:
$$ hurt = (4.1 + -0.4*\text{group}_i) + \epsilon_i $$


In R we would do this:


~~~
regression <- lm(data_t$hurt ~ data_t$workshop)

summary(regression)
~~~

~~~
## 
## Call:
## lm(formula = data_t$hurt ~ data_t$workshop)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.1034 -1.6667 -0.1034  1.8966  3.3333 
## 
## Coefficients:
##                                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                         4.1034     0.3988   10.29 9.85e-14 ***
## data_t$workshopStatistic Workshop  -0.4368     0.6153   -0.71    0.481    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.147 on 48 degrees of freedom
## Multiple R-squared:  0.01039,	Adjusted R-squared:  -0.01023 
## F-statistic: 0.5039 on 1 and 48 DF,  p-value: 0.4812
~~~

As you can see in the output, we have the same intercept as in our equation and also the estimated slope for the statistic Workshop. Yeah! But much more interesting is the fact that we have a _t-value_= 0.70 and a _p-value_= 0.48. As you can see just the same result as a t-test! 



###Doing more evil statistic stuff: ANOVA

As I said at the beginning I wanted to show you that ANOVA and regression are just the same. Because ANOVA is just an omnibus test that ends with a post-hoc procedure that is like doing much more t-tests (which are hopefully adjusted by some corrections), we have seen the most difficult things above.  
To calculate an ANOVA you mostly have more than two groups you want to compare and because of the familywise error you can't do much t-tests. 
Getting back to our Workshop hypothesis I thought that if a R Workshop and a statistic Workshop is so bad I can register some of my fellow students to a positive psychology Workshop which I code as *smile* and compare it to the other two. Unfortunately I've lost the data we used for the t-test so everyone had to do the workshops again to reproduce the data (Please remember that the data is not real. I would never torture my fellow students with R and statistic. Never.):



~~~
set.seed(23)
data_aov <- data.frame(
  workshop=sort(sample(1:3,75,replace = TRUE,prob = c(25,25,25))),
  hurt=sample(1:7,75,replace = TRUE)
  )
~~~

As before we have to build a factor:


~~~
data_aov$workshop <- factor(data_aov$workshop,levels = c(1,2,3),labels = c("R Workshop","Statistic Workshop","Smile Workshop"))
~~~

You would like to have another big, very unfriendly and confusing equation to build an ANOVA like a linear model? As you wish:

$$ hurt_i = (b_0 + b_1 \text{statistic}_i + b_2 \text{smile}_i) + \epsilon_i$$

It is very similiar to the t-test equation, just with one added group. If we would like to calculate the intercept *b*0 which is the mean of people done the R Workshop, we just have to set *statistic* and *smile* as 0. If we want the other slopes we have to set one 0 and the other one as 1 and the other way around. We would get the same as above. So I will save our time and do it in R (which is also faster and better like everytime using R).  
For doing a normal ANOVA you would use the *aov* commmand and then doing a posthoc Tukey Test using the *multcomp* package:



~~~
aov_model <- aov(hurt ~ workshop,data = data_aov)


tukey_test <- glht(aov_model,linfct = mcp(workshop = "Tukey"))

summary(tukey_test)
~~~

~~~
## 
## 	 Simultaneous Tests for General Linear Hypotheses
## 
## Multiple Comparisons of Means: Tukey Contrasts
## 
## 
## Fit: aov(formula = hurt ~ workshop, data = data_aov)
## 
## Linear Hypotheses:
##                                          Estimate Std. Error t value
## Statistic Workshop - R Workshop == 0       0.8008     0.5287   1.515
## Smile Workshop - R Workshop == 0          -0.4869     0.5430  -0.897
## Smile Workshop - Statistic Workshop == 0  -1.2878     0.5799  -2.221
##                                          Pr(>|t|)  
## Statistic Workshop - R Workshop == 0       0.2896  
## Smile Workshop - R Workshop == 0           0.6436  
## Smile Workshop - Statistic Workshop == 0   0.0743 .
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## (Adjusted p values reported -- single-step method)
~~~


Now doing it the regression way:


~~~
regression <- lm(data_aov$hurt ~ data_aov$workshop)

summary(regression)
~~~

~~~
## 
## Call:
## lm(formula = data_aov$hurt ~ data_aov$workshop)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -2.6774 -1.5778 -0.1905  1.5217  3.8095 
## 
## Coefficients:
##                                     Estimate Std. Error t value Pr(>|t|)
## (Intercept)                           3.6774     0.3451  10.657   <2e-16
## data_aov$workshopStatistic Workshop   0.8008     0.5287   1.515    0.134
## data_aov$workshopSmile Workshop      -0.4869     0.5430  -0.897    0.373
##                                        
## (Intercept)                         ***
## data_aov$workshopStatistic Workshop    
## data_aov$workshopSmile Workshop        
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.921 on 72 degrees of freedom
## Multiple R-squared:  0.06618,	Adjusted R-squared:  0.04025 
## F-statistic: 2.552 on 2 and 72 DF,  p-value: 0.085
~~~

Remember that the intercept is our R Workshop and that we compare each other Workshop with it! So you can watch at the post-hoc test and the regression output and see that there are the same _t_-values, but the _p_-values differ because they were adjusted in the post-hoc procedure.

###Why??

Because some people like me get a better understanding of ANOVA and Regression if we can stick it together. Also there are some calculations you can do using a Regression Model but not with an ANOVA, like structural equation modeling and so on.

If you want a even deeper understanding of Regression Models and their connections to other statistical tests I recommend you to read [Andy Field's "Discovering Statistic using R (2012)"](http://www.amazon.com/gp/product/1446200469?keywords=andy%20field&qid=1445770279&ref_=sr_1_2&sr=8-2) from which I've got most of the things represented here and because he can explain that in much better words than I can and does it with every test for which it is possible and he has nice humor.
