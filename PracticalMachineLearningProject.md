---
title: "Practical Machine Learinig Project"
author: "Yuval Shachaf"
date: "Tuesday, December 16, 2014"
output: html_document
---
# The libraries required to run the project
```
library(lattice)
library(ggplot2)
library(caret)
library(randomForest)
library(rpart)
setwd("C:\\Users\\Yoval\\Documents")
```
This project investigates the misclassification accuracy for the Rpart and RandomForests algorithms. First part includes Rpart which will be followed and compared to the RandonForest algorithm. Subsequently, it will be shown that the latter outperformes the former.

# Part 1 - Using Rpart
## Importing data


```r
training = read.csv("pml-training.csv")
```
## Cleaning and partitioning the data
the dataset coulumns containing NA and blank cells are excluded and 80% of the dataset is utilized for modeling.

```r
newtraining <- training[c(-1:-7,-12:-36,-50:-59,-69:-83,-87:-101,-103:-112,-125:-139,-141:-150)]
inTrain <- createDataPartition(y=newtraining$classe, p=0.8, list=FALSE)
newtrain <- newtraining[inTrain, ]
newtest <- newtrain[-inTrain, ]
```
## Fitting the model
Rpart algorithm from the Caret package is used based on the *Class* as an outcome

```r
modFitRpart<-train(classe ~.,data=newtrain,method="rpart")
```
## Testing the model
The model fit and Confusion Matrix data exhibit an avg accuracy of approx 50%, meaning we cannot use this model for prediction.


```r
predRpart <- predict(modFitRpart, newtest)
confusionMatrix(predRpart, newtest$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   A   B   C   D   E
##          A 819 247 274 236  65
##          B  17 198  17 102  75
##          C  66 139 269 174 160
##          D   0   0   0   0   0
##          E   3   0   0   0 278
## 
## Overall Statistics
##                                           
##                Accuracy : 0.4982          
##                  95% CI : (0.4806, 0.5159)
##     No Information Rate : 0.2883          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.3421          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9050  0.33904   0.4804   0.0000  0.48097
## Specificity            0.6321  0.91742   0.7910   1.0000  0.99883
## Pos Pred Value         0.4991  0.48411   0.3329      NaN  0.98932
## Neg Pred Value         0.9426  0.85861   0.8752   0.8369  0.89503
## Prevalence             0.2883  0.18605   0.1784   0.1631  0.18414
## Detection Rate         0.2609  0.06308   0.0857   0.0000  0.08856
## Detection Prevalence   0.5228  0.13030   0.2574   0.0000  0.08952
## Balanced Accuracy      0.7685  0.62823   0.6357   0.5000  0.73990
```

# Part 2 - Using Random Forests
## Importing and cleaning train data
Here we do the same as before. In addition we import the test set, as we will use it to predict the outcome as required.
We dont need to split the data into train and test since the RandomForest algorithm takes care of the validation internally.

```r
test = read.csv("pml-testing.csv")
newtrain <- training[c(-1:-7,-12:-36,-50:-59,-69:-83,-87:-101,-103:-112,-125:-139,-141:-150)]
```

Prior cleaning the test data, near-zero values has been checked. Since the function returns integer(0), no coloumns have been removed apart from those empty or containing NA

```r
newtrainNZV <- nearZeroVar(newtrain, saveMetrics=FALSE)
newtrainNZV
```

```
## integer(0)
```


```r
newtest <- test[c(-1:-7,-12:-36,-50:-59,-69:-83,-87:-101,-103:-112,-125:-139,-141:-150,-160)]
```
# Dataset resampling
The entire data set have been used for fitting to provide accuracy of approx 0.3%, with fewer entires such as 4000 the accuracy drops to approx. 2.75%.

```r
mysample <- newtrain[sample(1:nrow(newtrain),19622 ,replace=FALSE),]
```
## Fitting the model
ntree of 200 was chosen as a trade-off between speed and accuracy

```r
modFitRF <- randomForest(classe ~.,data=mysample, ntree=200)
modFitRF
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = mysample, ntree = 200) 
##                Type of random forest: classification
##                      Number of trees: 200
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.33%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 5579    1    0    0    0 0.0001792115
## B   13 3781    3    0    0 0.0042138530
## C    0   14 3403    5    0 0.0055523086
## D    0    0   20 3193    3 0.0071517413
## E    0    0    0    5 3602 0.0013861935
```
## Testing the model
Results has been verified yielding 100% prediction success 

```r
newtest$predClass<-predict(modFitRF,newtest)
answers <- newtest$predClass
answers
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
