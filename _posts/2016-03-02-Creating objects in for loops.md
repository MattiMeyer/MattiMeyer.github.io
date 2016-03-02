---
layout: post
title: Creating objects in for loops
subtitle: R is one big modern family
---

Today I've learned a little trick that probably will save much of my time and nerves. Imagine you would like to transform your data or select some of it and you could do it very easily with a _for_ loop. But you need every of this transformation in a new object in your environment and with an useful name! 

### Packages 

```r
library(dplyr)
```

### Data 

Imagine data like this, with persons and how many children they have:


```r
modernfamily <- data.frame(people=c("Mitchel", "Alex", "Phil", "Jay"),children=c(1,0,3,4) )

modernfamily
```

```
##    people children
## 1 Mitchel        1
## 2    Alex        0
## 3    Phil        3
## 4     Jay        4
```

Now I want to filter this persons in four groups by the number of children and every group should have his own object in my R environment.

I could easily filter with dplyr:


```r
modernfamily %>%
 select(people, children)%>%
 filter(children==1)
```

```
##    people children
## 1 Mitchel        1
```

But I'm very lazy, so I want to use a _for_ loop which does it four times for me and creates four objects:


```r
for(i in 0:max(modernfamily$children)){
  nam <- paste("Children",i,sep="") ## creates the objects names for every iteration
 
 number <- modernfamily %>%
           select(people, children) %>%
           filter(children==i)
 
 assign(nam,number) ## leads the filtered data frame and the new object name together 
}

##The resulting objects

Children0
```

```
##   people children
## 1   Alex        0
```

```r
Children1
```

```
##    people children
## 1 Mitchel        1
```

```r
Children2
```

```
## [1] people   children
## <0 rows> (or 0-length row.names)
```

```r
Children3
```

```
##   people children
## 1   Phil        3
```

```r
Children4
```

```
##   people children
## 1    Jay        4
```

As we can see, the results are fuor objects containing every of the Number of children. 

Another problem that was frustrating me is nearly the same. But know I want to add new transformations of a data to the old data frame, also in a _for_ loop.
So maybe we want to see how many children everyone of the modern family has, if they would get one more and one more and one more...

```r
for(i in 1:3){
  nam <- paste("Children+",i,sep="") ## creates the objects names for every iteration
 
 number <-  modernfamily$children+i ## the calculation
 
 modernfamily <- cbind(modernfamily, number) ## getting the old data frame and 
                                             ## the new calculation together
          
 names(modernfamily)[names(modernfamily)=="number"] <- nam  ## renaming the new variable                                                                           ## in the data frame
}

modernfamily
```

```
##    people children Children+1 Children+2 Children+3
## 1 Mitchel        1          2          3          4
## 2    Alex        0          1          2          3
## 3    Phil        3          4          5          6
## 4     Jay        4          5          6          7
```

So after this much of R you should relax and watch some episodes of [modern family](http://abc.go.com/shows/modern-family)!!
