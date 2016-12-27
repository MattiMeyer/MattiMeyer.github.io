---
layout: post
title: Substitution Rule Mining
subtitle: In this blogpost I introduce my implementation of the substitution Rule Mining Algorithm from Teng, Hsieh and Chen (2002) in R.
---





Imagine being a seller who wants to know what his customer often buy together. He wants some _rules_ like "When people buy X they also often buy Y", to get to know how e.g. he has to build up his store, so customers can find all they want very easily. A data scientist would know that the seller is searching for so called, _Association Rules_, which are often used in a Market Basket Analysis. The most known algorithm to extract such rules is the _Apriori_ algorithm, founded by [Agrawal & Srikant (1994)](https://www.it.uu.se/edu/course/homepage/infoutv/ht08/vldb94_rj.pdf). 

The concept of  Association Rules is very intuitive, because they count how often some variations of products are bought together, which is called _Support_. For example if we would have 10 different customers and 3 of them would buy apples and bananas, the combination of _{apples, bananas}_ would have a Support of 0.3. And maybe apples are bought by 5 people, because they want to take the doctor away, it would have a Support of 0.5. There are some other indices like _Confidence_ and _Interest_, which are described [here](https://en.wikipedia.org/wiki/Association_rule_learning), but all depends on the suppport of the products. 

The [_arules_ package](https://cran.r-project.org/web/packages/arules/index.html) provides the most of the presented features for mining Association Rules. But now imagine again you are a seller and you want to know: "What if Banana is sold out? What would my customers buy then?".
These question asks in some other direction as the first, because it asks for a _Substitution_ of one product and is not so easy to answer, because we want something to know for what we do not have data. Nobody goes into the shop and tell the seller what he would have bought when something would be sold out. Maybe you could start a questionnaire, but it could also happen that people give you biased answers. Nevertheless there are some data mining approaches that wants to handle this question and they are somehow related to the Association Rules [(overview here)](https://webdocs.cs.ualberta.ca/~zaiane/postscript/nar.pdf), because they also use the support. 

For this blogpost I will focus on the article from [Teng, Hsieh and Chen (2002)](https://www.researchgate.net/publication/279125220_On_the_mining_of_substitution_rules_for_statistically_dependent_items) which present an own algorithm for mining _Substitution Rules_ that I want to implement in R. I don't want to get deep into the theory because that is all written in the article. It is important to know that the authors want to add to every transaction made from customers all complements of all missing product that could have been bought. So an itemset could consist of e.g. {apple, banana, not oranges}. Further the authors introduce Chi-squared tests and Pearson's Correlation in a way that they could be used with the supports of products. The algorithm uses several conditions, which can be adjusted by the user, to extract the important rules. So Rules have to be above a minimum Support (_MinSup_), minimum Confidence (_MinConf_) and below the threshold of Correlation (_MinConf_). Other arguments in the function below are _pChi_, which is the indication of the alpha niveau of the Chi-squared test, _itemLabels_, which should be a character vector of all item labels to get the complements and _nTID_ which is the number of all taken transactions.

The R code below won't be the best way to handle this problem. There are some parts like the use of the arules package and the way of subsetting the transaction data that are not perfect, but it is a practical solution. Also it uses for loops which are a taboo by programming with R and maybe I will have later time to fix this parts. In addition to that, the calculation for big transaction data sets can need several hours and just combinations for three products are calculated. 

Here comes the code that I used for the function and below that I will show that it works well with the example used in the article by Teng et. al (2002).

```r
# Used packages:
library(arules)


SRM <- function(TransData, MinSup, MinConf, pMin, pChi, itemLabel, nTID){
 
 # Packages ----------------------------------------------------------------
 
 if (sum(search() %in% "package:arules") == 0) {
  stop("Please load package arules")
 }  
 
 # Checking Input data -----------------------------------------------------
 if (missing(TransData)) {
  stop("Transaction data is missing")
 }
 
 if (is.numeric(nTID) == FALSE) {
  stop("nTID has to be one numeric number for the count of Transactions")
 }
 
 if (length(nTID) > 1) {
  stop("nTID has to be one number for the count of Transactions")
 }
 
 if (is.character(itemLabel) == FALSE) {
  stop("itemLabel has to be a character")
 }
 # Concrete Item sets  ---------------------------------------------------
 
 # adding complements to transaction data
 compl_trans <- addComplement(TransData,labels = itemLabel)
 compl_tab <- crossTable(compl_trans,"support")
 compl_tab_D <- as.data.frame(compl_tab)
 # ordering matrix
 compl_tab_D <- compl_tab_D[order(rownames((compl_tab))),order(colnames((compl_tab)))]
 
 
 # Chi Value ---------------------------------------------------------------
 
 
 # empty data frame for loop
 
 complement_data <- data.frame(Chi = as.numeric(),
                               Sup_X.Y = as.numeric(),
                               X = as.character(),
                               Sup_X = as.numeric(),
                               Y = as.character(),
                               Sup_Y = as.numeric(),
                               CX = as.character(),
                               SupCX = as.numeric(),
                               CY = as.character(),
                               Sup_CY = as.numeric(),
                               Conf_X.CY = as.numeric(),
                               Sup_X.CY = as.numeric(),
                               Conf_Y.CX = as.numeric(),
                               SupY_CX = as.numeric())
 
 
 
 # first loop for one item
 for ( i in 1 : (length(itemLabel) - 1)) {
  # second loop combines it with all other items
  for (u in (i + 1) : length(itemLabel)) {
   
   
   # getting chi value from Teng
   a <-  itemLabel[i]
   b <-  itemLabel[u]
   ca <- paste0("!", itemLabel[i])
   cb <- paste0("!", itemLabel[u])
   
   chiValue <- nTID * (
    compl_tab[ca, cb] ^ 2 / (compl_tab[ca, ca] * compl_tab[cb, cb]) +
     compl_tab[ca, b] ^ 2 / (compl_tab[ca, ca] * compl_tab[b, b]) +
     compl_tab[a, cb] ^ 2 / (compl_tab[a, a] * compl_tab[cb, cb]) +
     compl_tab[a, b] ^ 2 / (compl_tab[a, a] * compl_tab[b, b]) - 1)
   
   
   
   # condition to be dependent
   if (compl_tab[a, b] > compl_tab[a, a] * compl_tab[b, b] && chiValue >= qchisq(pChi, 1) && 
       compl_tab[a, a] >= MinSup && compl_tab[b, b] >= MinSup ) {
    
    
    
    chi_sup <- data.frame(Chi = chiValue,
                          Sup_X.Y = compl_tab[a, b],
                          X = a,
                          Sup_X = compl_tab[a, a],
                          Y = b,
                          Sup_Y = compl_tab[b, b],
                          CX = ca,
                          SupCX = compl_tab[ca, ca],
                          CY = cb,
                          Sup_CY = compl_tab[cb, cb],
                          Conf_X.CY = compl_tab[a, cb] / compl_tab[a, a],
                          Sup_X.CY = compl_tab[a, cb],
                          Conf_Y.CX = compl_tab[ca, b] / compl_tab[b, b],
                          SupY_CX = compl_tab[ca, b])
    
    
    try(complement_data <- rbind(complement_data, chi_sup))
    
   }
   
   
  }
 }
 if (nrow(complement_data) == 0) {
  stop("No complement item sets could have been found")
 }
 
 
 #  changing mode of 
 complement_data$X <- as.character(complement_data$X)
 complement_data$Y <- as.character(complement_data$Y)
 
 
 # calculating support for concrete itemsets with all others and their complements -------------------
 
 
 ## with complements
 matrix_trans <- as.data.frame(as(compl_trans, "matrix"))
 
 sup_three <- data.frame(Items = as.character(),
                         Support = as.numeric()) 
 
 
 setCompl <- names(matrix_trans)
 # 1. extracts all other values than that are not in the itemset
 for (i in 1 : nrow(complement_data)) {
  value <- setCompl[ !setCompl %in% c(complement_data$X[i], 
                                      complement_data$Y[i], 
                                      paste0("!", complement_data$X[i]), 
                                      paste0("!",complement_data$Y[i]))]
  
  
  # 2. calculation of support
  for (u in value) {
   count <- sum(rowSums(matrix_trans[, c(complement_data$X[i], complement_data$Y[i], u )]) == 3)
   sup <- count / nTID  
   sup_three_items <- data.frame(Items = paste0(complement_data$X[i], complement_data$Y[i], u),
                                 Support=sup) 
   sup_three <- rbind(sup_three, sup_three_items)
  }
  
 }
 
 # Correlation of single items-------------------------------------------------------------
 
 
 # all items of concrete itemsets should be mixed for correlation
 combis <- unique(c(complement_data$X, complement_data$Y))
 
 # empty object
 rules<- data.frame(
  Substitute = as.character(),
  Product = as.character(),
  Support = as.numeric(),
  Confidence = as.numeric(),
  Correlation = as.numeric())
 
 # first loop for one item
 for (i in 1 : (length(combis) - 1)) {
  # second loop combines it with all other items
  for (u in (i + 1) : length(combis)) {
   
   first <- combis[i]
   second <- combis[u]
   
   corXY <- (compl_tab[first, second] - (compl_tab[first, first] * compl_tab[second, second])) /
    (sqrt((compl_tab[first, first] * (1 - compl_tab[first,first])) *
           (compl_tab[second, second] * (1 - compl_tab[second, second]))))
   
   
   # confidence
   conf1 <- compl_tab[first, paste0("!", second)] / compl_tab[first, first]
   conf2 <- compl_tab[second, paste0("!", first)] / compl_tab[second, second]
   
   two_rules <- data.frame(
    Substitute = c(paste("{", first, "}"), 
                   paste("{", second, "}")),
    Product = c(paste("=>", "{", second, "}"),
                paste("=>", "{", first, "}")),
    Support = c(compl_tab[first, paste0("!", second)], compl_tab[second, paste0("!", first)]),
    Confidence = c(conf1, conf2),
    Correlation = c(corXY, corXY)
   )
   
   # conditions
   try({
    if (two_rules$Correlation[1] < pMin) {
     if (two_rules$Support[1] >= MinSup && two_rules$Confidence[1] >= MinConf) {
      rules <- rbind(rules, two_rules[1, ])
     }
     if (two_rules$Support[2] >= MinSup && two_rules$Confidence[2] >= MinConf) {
      rules <- rbind(rules, two_rules[2, ])
     }
     
    } })
   
  }
 }
 
 
 # Correlation of concrete item pairs with single items --------------------
 # adding variable for loop
 complement_data$XY <- paste0(complement_data$X, complement_data$Y)
 
 # combination of items
 for (i in 1 : nrow(complement_data)){
  
  # set of combinations from dependent items with single items
  univector <- c(as.vector(unique(complement_data$X)), as.vector(unique(complement_data$Y)))
  univector <- univector[!univector %in% c(complement_data$X[i], complement_data$Y[i])]
  
  combis <- c(complement_data[i,"XY"], univector)
  
  
  
  for (u in 2 : length(combis)) {
   corXYZ <-(sup_three[sup_three$Items == paste0(combis[1], combis[u]),2] - 
              complement_data[complement_data$XY == combis[1],"Sup_X.Y"] *
              compl_tab[combis[u],combis[u]]) /
    (sqrt((complement_data[complement_data$XY == combis[1],"Sup_X.Y"] * 
             (1 - complement_data[complement_data$XY == combis[1],"Sup_X.Y"]) *
            compl_tab[combis[u],combis[u]] * (1 - compl_tab[combis[u],combis[u]]))))
   
   dataXYZ <- data.frame(
    Substitute = paste("{", combis[1], "}"), 
    Product = paste("=>", "{", combis[u], "}"),
    Support = sup_three[sup_three$Items == paste0(combis[1], "!", combis[u]),2],
    Confidence = sup_three[sup_three$Items == paste0(combis[1], "!", combis[u]),2] /
     complement_data[complement_data$XY == combis[1],"Sup_X.Y"],
    Correlation = corXYZ)
   
   
   # conditions
   if (dataXYZ$Correlation < pMin && dataXYZ$Support >= MinSup && dataXYZ$Confidence >= MinConf) {
    
    try(rules <- rbind(rules, dataXYZ))
   }
  }
 }
 if (nrow(rules) == 0) {
  message("Sorry no rules could have been calculated. Maybe change input conditions.")
 } else {
  return(rules)
 }
 
 # end
}
```

To show how the SRM function works, I reproduce the example in the article below. As you will see the output shows at the left side the substitution and right beside it the product which it can replace.  


```r
# Example
itemset <- data.frame(TID = c(1, 1, 2, 2, 3, 4, 4, 4, 5, 5, 5, 6, 7, 7, 7, 8, 8, 8, 9, 9, 9, 10, 10),
                      Item = c("b", "f", "b", "c", "c", "a", "b", "f", "a", "c", "d", "e", "a", "c", "d","b", "c", "f", "a", "b", "e", "a", "d"))

splitData <- split(itemset$Item, itemset$TID)

TransData <- as(splitData, "transactions")


SRM(TransData = TransData, MinSup = 0.2, MinConf = 0.7, pMin = -0.5, pChi = .95, itemLabel = as.character(unique(itemset$Item)), nTID =length(unique(itemset$TID)))
```

```r
##   Substitute  Product Support Confidence Correlation
## 1      { b } => { d }     0.5          1  -0.6546537
## 2      { d } => { b }     0.3          1  -0.6546537
## 3     { ad } => { b }     0.3          1  -0.6546537
```

