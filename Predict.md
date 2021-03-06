# Prediction of barbell lift method

## Reading data

We use the code shown below.


```r
# read training and test data sets
training_data <- read.csv("pml-training.csv", header = TRUE, sep = ",")
testing_data <- read.csv("pml-testing.csv", header = TRUE, sep = ",")
```

## Creating of a validation set

The training data is partitioned into two sets, one for training and one for validation.


```r
# partition into training and validation sets
inTrain <- createDataPartition(y=training_set$classe,p=0.75, list=FALSE)
training_set <- training_data[inTrain,]
validation_set <- training_data[-inTrain,]
```

## Cleaning up the data

The first 7 columns are not used for prediction, and are removed from the data set. Columns which are largely
NA or blank are also removed.



```r
# remove unwanted columns from training set
training_set <- training_set[,-1:7]

# remove columns that are mostly NA values
numna <- function(x) sum(is.na(x))
sumna <- colwise(numna)(training_set)
cols <- which(sumna >= 10000)
training_set <- training_set[,-cols]

# remove mostly blank columns
numblank <- function(x) sum(x == 0 | x == "")
sumblank <- colwise(numblank)(training_set)
cols_blank  <- which(sumblank >= 3000)
training_set <- training_set[,-cols_blank]
```
## Model selection and accuracy

Now we train some models on the training set and evaluate their accuracy on the validation set. Two methods are shown below - random forest 
and linear discriminant analysis.


```r
tc <- trainControl(method = "oob", number = 5, repeats = 4)

ldamodel <- train(formula("classe ~ ."), data=training_set, method="rf", trControl = tc)
ldaprediction <- predict(ldamodel, validation_set)

rfmodel <- train(formula("classe ~ ."), data=training_set, method="rf", trControl = tc)
rfprediction <- predict(rfmodel,validation_set)
```
Out of sample error can be estimated by looking at the confusion matrix for each model. This output is not shown in this document to keep its
size limited. It can be seen from the confusion matrix that the random forest method provides lower error.


```r
confusionMatrix(validation_set$classe, ldaprediction)
confusionMatrix(validation_set$classe, rfprediction)
```

## Applying the selected model to the test cases

The random forest algorithm is applied to the 20 given test cases.


```r
testpredict <- predict(rfmodel, testing_set)
```


