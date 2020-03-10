---
title: "Prediction"
output:
  html_document:
    keep_md: yes
---



## Data

We load the data.


```r
training <- read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv")
testing <- read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv")
```

We remove all columns that contain missing values, encoded by either NA or empty factor variables.
The prediction is supposed to use data from accelerometers on the belt, forearm, arm, and dumbell. So we only keep those predictors.

```r
training <- training[,!apply(training,2,function(x) any(is.na(x)) | any(as.character(x) == ""))]

training <- training[,grep("arm|belt|dumbbel|classe",names(training))]
```

## Training/Validation Split

Test the training set into the training set used for model training and a cross validation set. We use a 80/20 split.

```r
set.seed(20)
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
inTrain <- createDataPartition(training$classe, p = 0.8)[[1]]
trainingS <- training[ inTrain,]
validationS <- training[-inTrain,]
```

## Model Training

We train a SVM.

```r
library(e1071)
model <- svm(factor(classe) ~ . , data = trainingS)
```

## Model assessment

In sample confusion matrix

```r
confusionMatrix(trainingS$classe, predict(model))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 4443    9   11    0    1
##          B  202 2779   53    1    3
##          C    6   66 2645   19    2
##          D    5    0  235 2332    1
##          E    0    9   62   54 2761
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9529          
##                  95% CI : (0.9495, 0.9562)
##     No Information Rate : 0.2966          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9404          
##                                           
##  Mcnemar's Test P-Value : < 2.2e-16       
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9543   0.9707   0.8799   0.9692   0.9975
## Specificity            0.9981   0.9798   0.9927   0.9819   0.9903
## Pos Pred Value         0.9953   0.9147   0.9660   0.9063   0.9567
## Neg Pred Value         0.9810   0.9934   0.9721   0.9944   0.9995
## Prevalence             0.2966   0.1824   0.1915   0.1533   0.1763
## Detection Rate         0.2830   0.1770   0.1685   0.1485   0.1759
## Detection Prevalence   0.2843   0.1935   0.1744   0.1639   0.1838
## Balanced Accuracy      0.9762   0.9752   0.9363   0.9756   0.9939
```

Out of sample confusion matrix

```r
confusionMatrix(validationS$classe, predict(model,validationS))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1110    4    1    0    1
##          B   61  679   17    1    1
##          C    1   14  664    5    0
##          D    0    0   49  593    1
##          E    0   10   20   12  679
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9495          
##                  95% CI : (0.9422, 0.9562)
##     No Information Rate : 0.2988          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9361          
##                                           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9471   0.9604   0.8842   0.9705   0.9956
## Specificity            0.9978   0.9751   0.9937   0.9849   0.9870
## Pos Pred Value         0.9946   0.8946   0.9708   0.9222   0.9417
## Neg Pred Value         0.9779   0.9912   0.9731   0.9945   0.9991
## Prevalence             0.2988   0.1802   0.1914   0.1557   0.1738
## Detection Rate         0.2829   0.1731   0.1693   0.1512   0.1731
## Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
## Balanced Accuracy      0.9725   0.9678   0.9389   0.9777   0.9913
```

The expected out of sample accuracy is 95%, the same as the cross validation accuracy.


