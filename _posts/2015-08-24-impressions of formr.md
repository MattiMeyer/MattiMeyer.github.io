---
layout: post
title: Impressions of formr
---
A few weeks ago, I was introduced to an new webside combining the features of a survey software online tool with R, called [__formr__](https://formr.org/). Together with this webside the psychologists who provide this webside, developed the [formr R package](https://github.com/rubenarslan/formr) on Github.  
Take a look at formr:

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/formr-side2.png)

Here are the features of formr that I like:

* you can do simple surveys with an excel spreedsheat, so you can reproduce the items

* the layout of the surveys is simple but effective

* it is for freeeeeeee

* making personal feedback just with using R

* the settings of your survey (e.g.when does participants get the survey) is also done just with R

* there is a google help mailing list, where you can contact the provider of formr directly

* there is a nice way to download your data directly into R with your login account (I will show it below)

* for more features look [here](https://formr.org/public/documentation#features)

Althoug there are nice features, formr is just in the beta phase, so you have to mail the provider personally to get an admin account to create surveys and there can be errors and problems by creating a survey. To see what is possible with formr take a look at [surveys created with formr](https://formr.org/public/studies).

##The formr package

I like to introduce you to the formr package, because I think it is an easy way to get your survey data into R and you can do funny things with it.
For my purpose I've done a simple survey in formr that looks like this:
![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/formr-question.png)

I name this survey __dylR__ (do you love R?)! 

If you like you can do this survey too and get a short personal feedback!

__Show up [here](https://formr.org/Do_you_love_R)__

###Installing formr
To install formr you need the devtools package, because formr is made on Github:



~~~
# library(devtools)
# install.packages("devtools")
  # devtools::install_github("rubenarslan/formr")

library(formr)
~~~

###Connect with formr
You can connect yourself very easily with formr, just with you email and your formr-password:

~~~
formr_connect(email = "mymailadress.de", password = "mypassword" )
~~~
###Getting the data

Now you can get the results of people done your survey, just with the surveyname and the host:

~~~
question_data <- formr_raw_results(survey_name = "blogaboutformr",host = "https://formr.org")

head(question_data)
~~~

~~~
##   session session_id             created            modified
## 1    <NA>     161883 2015-08-21 15:22:07 2015-08-21 15:22:36
## 2    <NA>     161884 2015-08-21 15:22:51 2015-08-21 15:23:09
## 3    <NA>     161885 2015-08-21 15:23:12 2015-08-21 15:23:29
## 4    <NA>     161886 2015-08-21 15:24:06 2015-08-21 15:24:56
## 5    <NA>     162006 2015-08-21 19:03:29                <NA>
## 6    <NA>     162007 2015-08-21 19:03:34 2015-08-21 19:03:38
##                 ended Age Geschlecht dylR_rscale1R dylR_rscale2
## 1 2015-08-21 15:22:36  33          1             1            4
## 2 2015-08-21 15:23:09  44          2             5            2
## 3 2015-08-21 15:23:29  45          3             1            4
## 4 2015-08-21 15:24:56  33          3             1            4
## 5                <NA>  NA         NA            NA           NA
## 6                <NA>  22          1            NA           NA
##   dylR_rscale3 dylR_rscale4 dylR_rscale5 dylR_rscale6 dylR_statisticscale1
## 1            5            4            4            3                    3
## 2            1            1            2            2                    1
## 3            5            4            4            5                    5
## 4            4            5            5            4                    5
## 5           NA           NA           NA           NA                   NA
## 6           NA           NA           NA           NA                   NA
##   dylR_statisticscale2 dylR_statisticscale3 dylR_statisticscale4
## 1                    4                    4                    5
## 2                    3                    2                    2
## 3                    5                    5                    3
## 4                    4                    5                    5
## 5                   NA                   NA                   NA
## 6                   NA                   NA                   NA
~~~

~~~
##You can also download your items into a data frame, to take a look at the questions 
items.data.frame <- as.data.frame(formr_items(survey_name = "blogaboutformr" ))

items.data.frame[6,]
~~~

~~~
##       id study_id          type  choice_list type_options         name
## 6 437763     2807 rating_button dylR_rscale2            5 dylR_rscale2
##                                                         label
## 6 I like to make a tattoo of a ggplot2 graphic on my forehead
##                                                  label_parsed optional
## 6 I like to make a tattoo of a ggplot2 graphic on my forehead        0
##              class showif value order                            choices
## 6 label_align_left           NA     9 1=Do not agree,2=2,3=3,4=4,5=Agree
##   index
## 6     6
~~~

~~~
### if you are interested in the exact time your participants answered some questions, you can look here
item_displays <- formr_item_displays("blogaboutformr", host = "https://formr.org")

head(item_displays)
~~~

~~~
##   session                 name answer             created
## 1    <NA>                  Age     33 2015-08-21 15:22:07
## 2    <NA>           Geschlecht      1 2015-08-21 15:22:07
## 3    <NA>               submit      1 2015-08-21 15:22:07
## 4    <NA> request_questionaire      1 2015-08-21 15:22:23
## 5    <NA>        dylR_rscale1R      1 2015-08-21 15:22:23
## 6    <NA>         dylR_rscale2      4 2015-08-21 15:22:23
##                 saved               shown shown_relative
## 1 2015-08-21 15:22:23 2015-08-21 13:22:19       776.1592
## 2 2015-08-21 15:22:23 2015-08-21 13:22:19       776.1592
## 3 2015-08-21 15:22:23 2015-08-21 13:22:19       776.1592
## 4 2015-08-21 15:22:36 2015-08-21 13:22:35       694.3491
## 5 2015-08-21 15:22:36 2015-08-21 13:22:35       694.3491
## 6 2015-08-21 15:22:36 2015-08-21 13:22:35       694.3491
##              answered answered_relative displaycount
## 1 2015-08-21 13:22:33         15180.403            1
## 2 2015-08-21 13:22:34         15278.469            1
## 3 2015-08-21 13:22:19                NA            1
## 4 2015-08-21 13:22:35                NA            1
## 5 2015-08-21 13:22:38          4342.310            1
## 6 2015-08-21 13:22:42          7385.483            1
~~~

###Reversed items and scales
Like the most surveys, we have some items that need to be reversed, like the item: "*I hate R*". Formr gives us a nice tool to do that, because every Item that ends with a *R* will be reversed (do not confuse it with the language R :P). And also you can filter the scales of the survey. I used a scale to test if people love R and a scale if they love statistic. So formr will give me the means of the scales. To get items into formr you have to make an excel spreadsheet where you can name the items yourself. I've named them like __dylR_rscale1R__ and formr recognizes the scale (here: rscale), the number of the scale item and the *R* to reverse the item.


~~~
aggreg <- formr_aggregate(survey_name = "blogaboutformr",host = "https://formr.org", fallback_max = 5)

###here are the means of the scales
aggreg[,19:20]
~~~

~~~
##   dylR_rscale dylR_statisticscale
## 1    4.166667                4.00
## 2    1.500000                2.00
## 3    4.500000                4.50
## 4    4.500000                4.75
## 5          NA                  NA
## 6          NA                  NA
## 7    4.833333                4.50
~~~

~~~
###there is another function to get just the reversed items

##We need the items again, but they must be a list
items <- formr_items(survey_name = "blogaboutformr" )
reverse <- formr_reverse(question_data,item_list = items, fallback_max = 5)
~~~

###Missing values
Use the __miss_frac__ function to get the percentage of missing values in a data frame: 

~~~
miss_frac(question_data)
~~~

~~~
##              session           session_id              created 
##            0.8571429            0.0000000            0.0000000 
##             modified                ended                  Age 
##            0.1428571            0.2857143            0.1428571 
##           Geschlecht        dylR_rscale1R         dylR_rscale2 
##            0.1428571            0.2857143            0.2857143 
##         dylR_rscale3         dylR_rscale4         dylR_rscale5 
##            0.2857143            0.2857143            0.2857143 
##         dylR_rscale6 dylR_statisticscale1 dylR_statisticscale2 
##            0.2857143            0.2857143            0.2857143 
## dylR_statisticscale3 dylR_statisticscale4 
##            0.2857143            0.2857143
~~~


##Graphics
###Barplots 
The graphics of formr are designed to give your participants a personal feedback, because you can implement them on the formr webside.  Therefore the Graphics will display just one person, but you could also change the graphics that they show all participants. Do what you want.
For my sake I will build a data frame with the means of the two scales of the first person and after plotting that, I will add the standard deviation:

~~~
library(dplyr)
barplot <- data.frame(variable=c("R","Statistic"),value=c(aggreg$dylR_rscale[1],aggreg$dylR_statisticscale[1]))
qplot_on_bar(normed_data = barplot, ylab="Means")
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/barplot.png) 

~~~
##Standard deviation
itemsrscale <- question_data %>% select(contains("rscale"))
itemsstatisticscale <- question_data %>% select(contains("statisticscale"))
sdrscale <- sd(itemsrscale[1,])
sdstatscale<- sd(itemsstatisticscale[1,])

barplot <- data.frame(variable=c("R","Statistic"),value=c(aggreg$dylR_rscale[1],aggreg$dylR_statisticscale[1]), se=c(sdrscale,sdstatscale)) 

qplot_on_bar(normed_data =barplot)
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/barplotsd.png) 

###Comparing using a normal distribution
Most people are curious if they are better or worse than other people, so they wan't to compare themselve with others. Formr gives you the possibility to create a normal distribution equipped with a line, showing the personal value of the participant. You can also display all participants, to visualise a trend. If you wonder about the ++ and - at the x-axis, I wan't to remind that this is for personal feedback and every normal person should understand this plot, not just freaky statisticians.


~~~
scaled <- scale(aggreg$dylR_statisticscale)
qplot_on_normal(scaled[3], xlab = "Do you love statistic?")
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/normalone.png) 

~~~
qplot_on_normal(scaled[1:4], xlab = "Do you love statistic?",colour = rainbow(4))
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/normalmore.png) 

We can see that the green person is an outlier, a real outlaw, while the other persons love statistic. A good trend...?

###Plotting licert scales
Most surveys use licert scales to get knowledge about the participants. So it would be nice if we could visualise licert scales and formr can do this very nicely:

~~~
##Now we choose the items from the list that are likert scales and combine it with the results of the survey 
likert_items <- formr_likert(item_list = items[5:10], results = question_data)
plot(likert_items)
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/likertold.png) 

There are very much participants who do not agree with hating R...what wonderful people.

Because the likert scale with my survey data do not look so impressive we will simulate much more data with the formr *simulate* funtion:

~~~
##We simulate data, taking our items as the base
sim_results <- formr_simulate_from_items(items)

likert_items <- formr_likert(item_list = items[5:10], results = sim_results)
plot(likert_items)
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/likertsim.png) 

###Writing a personal feedback

Formr gives you a tool to write feedback to the participants of your survey, based on if-statements. You can do that with z-standardised values, like we build them before:

~~~
feedback_chunk(normed_value = scaled[1,], chunks = c("You don't love statistic...poor you","You are interested, that's nice!  ","You are a statistic freak...get out here!"))
~~~

~~~
## [1] "You are interested, that's nice!  "
~~~

~~~
feedback_chunk(normed_value = scaled[2,], chunks = c("You don't love statistic...poor you","You are interested, that's nice!  ","You are a statistic freak...get out here!"))
~~~

~~~
## [1] "You don't love statistic...poor you"
~~~

###Multi-Trait-Multi-Method Matrix
A very nice function is _mtmm_, because it builds a multi trait multi method matrix. With the data I used before, I can't show that to you, so I will just cite the examples of the formr package:

~~~
data.mtmm = data.frame(
`Ach_self_report` = rnorm(200), `Pow_self_report` = rnorm(200), `Aff_self_report`= rnorm(200),
`Ach_peer_report` = rnorm(200),`Pow_peer_report`= rnorm(200),`Aff_peer_report` = rnorm(200),
`Ach_diary` = rnorm(200), `Pow_diary` = rnorm(200),`Aff_diary` = rnorm(200))
reliabilities = data.frame(scale = names(data.mtmm), rel = runif(length(names(data.mtmm))))
mtmm(data.mtmm, reliabilities = reliabilities)
~~~

![](https://raw.githubusercontent.com/MattiMeyer/MattiMeyer.github.io/master/_posts/images/formr/mtmm.png) 

##Diary Studies
Formr offers some specials for diary studies like sending SMS via Clickatell, massenversand or twilio. As well there are R functions to handle the sending of the SMS. I haven't test this feature yet, but I wan't to show you the sceleton of the functions:

~~~
# text_message_clickatell(To, Body, Token, return_result = F)

# text_message_massenversand(To, From, Body, id, pw, time = "0",
  # msgtype = "t", tarif = "OA", test = "0", return_result = F)

# text_message_twilio(To, From, Body, Account, Token, return_result = F)
~~~

##Disconnect with formr

~~~
formr_disconnect(host = "https://formr.org")
~~~
There are lots of other functions in the formr Package, but some of them are just useful in special survey situations or I don't know what to do with them. What I wanted to show was the big potential of the formr webside and the formr package, because I think we need some webside like this in the future to combine the thousands possibilities and packages of R with the creating of surveys.

Thanks to the developers of formr! 
