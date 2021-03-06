ST558 Project 2
================
Mu-Tien, Lee

-   [Require package](#require-package)
-   [Read in data](#read-in-data)
-   [Summarize the training data](#summarize-the-training-data)
-   [Training Model](#training-model)
    -   [Tree-based model](#tree-based-model)
    -   [Boosted tree model](#boosted-tree-model)
-   [Predicting using the best tree-base model](#predicting-using-the-best-tree-base-model)
-   [Predicting using the best boosted-tree model](#predicting-using-the-best-boosted-tree-model)

Require package
===============

``` r
#install.packages("")
library(knitr)
library(rmarkdown)
library(MuMIn)
library(tidyverse)
library(caret)
library(corrplot)
library(readxl)
library(caret)
library(ggiraphExtra)
library(knitr)
library(ggplot2)
library(ggpubr)
library(rpart.plot)
library(rpart)
library(DT)
```

Read in data
============

``` r
#read in hour data
HourData <- read.csv("hour.csv")
HourData<- HourData %>% select(-casual, -registered)
HourData$yr <- as.factor(HourData$yr)
HourData$holiday <- as.factor(HourData$holiday)
HourData$workingday <- as.factor(HourData$workingday)

#filter data by weekday
HourData <-HourData %>% filter(weekday==params$w)
#showing data
HourData <-HourData %>% select(-weekday, -workingday,-instant)
tbl_df(HourData)
```

    ## # A tibble: 2,475 x 12
    ##    dteday season yr     mnth    hr holiday weathersit  temp atemp   hum
    ##    <chr>   <int> <fct> <int> <int> <fct>        <int> <dbl> <dbl> <dbl>
    ##  1 2011-~      1 0         1     0 0                1  0.2  0.258  0.64
    ##  2 2011-~      1 0         1     1 0                1  0.16 0.197  0.74
    ##  3 2011-~      1 0         1     2 0                1  0.16 0.197  0.74
    ##  4 2011-~      1 0         1     4 0                1  0.24 0.227  0.48
    ##  5 2011-~      1 0         1     5 0                1  0.22 0.227  0.47
    ##  6 2011-~      1 0         1     6 0                1  0.2  0.197  0.47
    ##  7 2011-~      1 0         1     7 0                1  0.18 0.182  0.43
    ##  8 2011-~      1 0         1     8 0                1  0.2  0.182  0.4 
    ##  9 2011-~      1 0         1     9 0                1  0.22 0.197  0.37
    ## 10 2011-~      1 0         1    10 0                1  0.22 0.197  0.37
    ## # ... with 2,465 more rows, and 2 more variables: windspeed <dbl>, cnt <int>

``` r
#Separate dataset into train (70%) and test (30%) data set
set.seed(1997)
train <- sample(1:nrow(HourData), size = nrow(HourData)*0.7)
test <- dplyr::setdiff(1:nrow(HourData), train)
HourDataTrain <- HourData[train, ]
HourDataTest <- HourData[test, ]
```

Summarize the training data
===========================

Here I will show you some summary of my training dataset.
1. I conduct a histogram of the rental count, since this is my response variable.
2. I built up a summary table of all the weather measurement.
3. I also showing the weather summary via a boxplot.
4. I plot the rental count distributed by time.
5. I plot the rental count distributed by weather situation.

``` r
# plot the histogram of rental count
hist <- ggplot(data=HourDataTrain, aes(x=cnt))+geom_histogram(binwidth = 20, aes(color=yr))
hist <-hist+labs(title="Histogram of the retal count", x="rental count")
hist <-hist+scale_fill_discrete(labels=c(2011,2012))
hist
```

![](Wednesday_files/figure-markdown_github/summarizing%20data-1.png)

``` r
#prin out summary table for tempature humidity and windspeed
sum <- HourDataTrain%>% select(c(temp, atemp, hum, windspeed))
kable(apply(sum, 2,summary), caption="Numeric Summary for weather measurement")
```

|         |       temp|      atemp|        hum|  windspeed|
|:--------|----------:|----------:|----------:|----------:|
| Min.    |  0.0200000|  0.0152000|  0.1900000|  0.0000000|
| 1st Qu. |  0.3400000|  0.3333000|  0.4800000|  0.1045000|
| Median  |  0.5200000|  0.5000000|  0.6500000|  0.1642000|
| Mean    |  0.5056005|  0.4826364|  0.6409296|  0.1854523|
| 3rd Qu. |  0.6600000|  0.6212000|  0.8100000|  0.2836000|
| Max.    |  0.9400000|  0.8788000|  1.0000000|  0.6418000|

``` r
#plot the boxplot of tempature humidity and windspeed (not genralized amount)
#plot base
boxplot <- ggplot(data = HourDataTrain, aes(x=season))
#adding 4 variables
tem <-boxplot+geom_boxplot(aes(y=temp*41, group=season))+labs(y="Tempature (c)", title = "boxplot for weather measurement")
fetem <-boxplot+geom_boxplot(aes(y=atemp*50, group=season))+labs(y="Feeling Tempature (c)")
hum <-boxplot+geom_boxplot(aes(y=hum*100, group=season))+labs(y="Humidity")
wind <-boxplot+geom_boxplot(aes(y=windspeed*67, group=season))+labs(y="Wind Speed")
#combine 4 plots into 1
ggarrange(tem, fetem, hum , wind, ncol = 2, nrow = 2)
```

![](Wednesday_files/figure-markdown_github/plotting%20data-1.png)

``` r
# plot the count distribution among time and weather
# by time
barplot1<-ggplot(data = HourDataTrain, aes(x=hr))+geom_col(aes(y=cnt, fill=yr))+facet_wrap(~mnth)
barplot1 <- barplot1+labs(x="time", y="Rental Count", title="Retal count distribution by month" )
barplot1+scale_fill_discrete(name="year", labels=c(2011,2012))
```

![](Wednesday_files/figure-markdown_github/plotting%20data-2.png)

``` r
# by weather
barplot2 <-ggplot(data = HourDataTrain, aes(x=weathersit))+geom_col(aes(y=cnt, fill=yr))+facet_wrap(~mnth)
barplot2 <- barplot2+labs(x="Weather situation, 1: clear day, 2: misty day, 3:rain or snow", y="Rental Count", title="Retal count distribution by month" )
barplot2+scale_fill_discrete(name="year", labels=c(2011,2012))
```

![](Wednesday_files/figure-markdown_github/plotting%20data-3.png)

Training Model
==============

Here I use two different method to train my model. First method is using a tree-based models with leave one out cross validation. For the second method, I use the boosted tree model with cross validation. Both two training are done using the `train` function from `caret` package. The data was cantered and scaled before training.

Tree-based model
----------------

Since our respons variable is continuous. I use the regression tree model to training my data. The `method= "rpart"` was used in `train` function
Moreover, because I want to use the leave-one-out cross validation for this training, therefore,the `method= "LOOCV"` was used in `trainControl`.
We can adjust the grid parameter by ourselves. Since the default result shows that `cp` should be very small to have a lowest RMSE. I set a range \[0.0001,0.0005\] to fit for every weekday.
Something to notice, because the `cp` is too small, when I draw my regression tree, it seems like a mess.

``` r
# set up training control, using leave one out cross validation.
set.seed(615)
trctrl <- trainControl(method = "LOOCV", number = 1)

# getModelInfo("rpart")
# training using regression tree models with cp in [0.0001,0.0005]
# since the cp seems have to be really small when I used the default cp to train

model1 <- cnt~season+yr+mnth+hr+holiday+weathersit+temp+atemp+hum+windspeed

RegTree_fit1 <- train(model1, data = HourDataTrain, method = "rpart",
                 trControl=trctrl,
                 preProcess = c("center", "scale"),
                 tuneGrid=expand.grid(cp=seq(0.0001,0.0005,0.00004))
)

# show the training result
RegTree_fit1
```

    ## CART 
    ## 
    ## 1732 samples
    ##   10 predictor
    ## 
    ## Pre-processing: centered (10), scaled (10) 
    ## Resampling: Leave-One-Out Cross-Validation 
    ## Summary of sample sizes: 1731, 1731, 1731, 1731, 1731, 1731, ... 
    ## Resampling results across tuning parameters:
    ## 
    ##   cp       RMSE      Rsquared   MAE     
    ##   0.00010  69.34262  0.8689469  40.72888
    ##   0.00014  69.49525  0.8683654  41.07013
    ##   0.00018  69.71530  0.8675204  41.36262
    ##   0.00022  69.90408  0.8667575  41.82930
    ##   0.00026  69.57638  0.8679606  41.35149
    ##   0.00030  69.77883  0.8672371  41.72130
    ##   0.00034  69.78617  0.8671588  42.26889
    ##   0.00038  69.78943  0.8671168  42.32354
    ##   0.00042  70.18148  0.8655887  43.38002
    ##   0.00046  70.50494  0.8643981  43.89760
    ##   0.00050  70.57485  0.8641321  44.30955
    ## 
    ## RMSE was used to select the optimal model using the smallest value.
    ## The final value used for the model was cp = 1e-04.

``` r
# plot the RMSE of selected cp
plot(RegTree_fit1)
```

![](Wednesday_files/figure-markdown_github/tree%20based%20model-1.png)

``` r
# plot my final tree model
rpart.plot(RegTree_fit1$finalModel)
```

![](Wednesday_files/figure-markdown_github/tree%20based%20model-2.png)

Boosted tree model
------------------

Here I want to training my data using boosted tree model. The `method= "gbm"` was used in `train` function
Because I want to use thecross validation for this training, therefore,the `method= "cv"` was used in `trainControl`.
We can adjust the grid parameter by ourselves. I set a range of number of tree \[100,1250\] and interaction 5~11 to fit for every weekday.

``` r
# set up training control, using cross validation with 10 folder
set.seed(615)
trctrl <- trainControl(method = "cv", number = 10)

# training using boosted tree models with boosting interation in [700,1250] and try max tree depth 5~9
model2 <- cnt~season+yr+mnth+hr+holiday+weathersit+temp+atemp+hum+windspeed
RegTree_fit2 <- train(model2, data = HourDataTrain, method = "gbm",
                trControl=trctrl,
                preProcess = c("center", "scale"),
                tuneGrid=expand.grid(n.trees=seq(100,1250,25),
                                     interaction.depth=5:11,
                                     shrinkage=0.1, n.minobsinnode=10)
                 )
```

``` r
# show the training result of boosted tree
RegTree_fit2$bestTune
```

    ##     n.trees interaction.depth shrinkage n.minobsinnode
    ## 119     700                 7       0.1             10

``` r
# plot the RMSE of different parameters
plot(RegTree_fit2)
```

![](Wednesday_files/figure-markdown_github/boosted%20result-1.png)

Predicting using the best tree-base model
=========================================

Using the best boosted tree model to testing the data.

``` r
# predict use predict function
tree_pred <- predict(RegTree_fit1, newdata = HourDataTest)

#Calculate the Root MSE
RMSE_tree<- sqrt(mean((tree_pred-HourDataTest$cnt)^2))
label <- paste0("RMSE =", RMSE_tree)

# plot the prediction
count <- data.frame(true_count=HourDataTest$cnt,prediction=tree_pred )
predPlot <- ggplot(data=count, aes(x=true_count,y=prediction))
predPlot <- predPlot+labs(title="Prediction V.s. True Count using tree-base model")+geom_point()
predPlot <- predPlot+geom_smooth(color="orange")+geom_abline(aes(intercept=0,slope=1), color="blue")
predPlot <- predPlot+geom_text(x=200, y=600,label=label, color="brown")
predPlot
```

![](Wednesday_files/figure-markdown_github/predicting%20tree%20model-1.png)

Predicting using the best boosted-tree model
============================================

``` r
# predict use predict function
boosted_pred <- predict(RegTree_fit2, newdata = HourDataTest)

#Calculate the Root MSE
RMSE_boosted <- sqrt(mean((boosted_pred-HourDataTest$cnt)^2))
lab <- paste0("RMSE =", RMSE_boosted)
# plot the prediction
count2 <- data.frame(True_count=HourDataTest$cnt,prediction=boosted_pred )
pred_plot <- ggplot(data=count2, aes(x=True_count,y=prediction))
pred_plot <- pred_plot+labs(title="Prediction V.s. True Count using boosted model")+geom_point()
pred_plot <- pred_plot+geom_smooth(color="orange")+geom_abline(aes(intercept=0,slope=1), color="blue")
pred_plot <- pred_plot+geom_text(x=200, y=600,label=lab, color=" brown")
pred_plot
```

![](Wednesday_files/figure-markdown_github/predicting%20boosted%20model-1.png)
