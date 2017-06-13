# "Practical Machine Learning -Project"
Author: "Farzad Ravari"

# Executive Summary
This project attempts to evelauate effect of using the devices such as Jawbone Up, Nike FuelBand, and Fitbit for collecting data information during activities for measurment of vital signe and health factors for modification of health behavior and enhance activities . Data collected from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. Exploratory analysis of data includes implementation of random forests for cross predication befor and after cross validation and test set with removal of columns less tha 60% of data and thereforedata validation and building a model with cross validation and predication .
Data source:https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv
# Set Directory
        setwd("C:/Users/FARZAD/Desktop/Data Science/Course 8/Project")
        getwd()
        [1] "C:/Users/FARZAD/Desktop/Data Science/Course 8/Project"
  
  # Install necessary packages
  
        install.packages("ElemStatLearn")
        install.packages("caret") 
        library(ElemStatLearn) 
        library(caret) 
        install.packages("rpart") 
        library(rpart) 
        install.packages("randomForest")
        library(randomForest)
  # Loading Data
        UrlTRN<- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
        UrlTST  <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
        fileTRN <- "C:/Users/farza/Desktop/Data Science/Course 8/Project/pml-training.csv"
        fileTST <- "C:/Users/farza/Desktop/Data Science/Course 8/Project/pml-testing.csv"
   
   # Download the files
        download.file(url=UrlTRN, destfile=fileTRN)
        trying URL 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv'
        Content type 'text/csv' length 12202745 bytes (11.6 MB)
        downloaded 11.6 MB
   
        download.file(url=UrlTST, destfile=fileTST)
        trying URL 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv'
        Content type 'text/csv' length 15113 bytes (14 KB)
        downloaded 14 KB
 # Reading Data
        TRN <- read.csv("pml-training.csv",na.strings = "")
        TST <- read.csv("pml-testing.csv",na.strings = "NA")
        dim(TRN)
        [1] 19622   160
        dim(TST)
        [1]  20 160
   # Exploratory Data Analysis
   Removal of missing value in Training and Testing Files

        TRN_REM_na <- TRN[,(colSums(is.na(TRN)) == 0)]
        TST_REM_na <- TST[,(colSums(is.na(TST)) == 0)]
    
    Removal of Unnecessry columns in Training and Testing files
    
        colRm_TRN <- c("user_name","raw_timestamp_part_1","raw_timestamp_part_2","cvtd_timestamp","num_window")
        colRm_TST <- c("user_name","raw_timestamp_part_1","raw_timestamp_part_2","cvtd_timestamp","num_window","problem_id")
    
        TRN_colRm <- TRN_REM_na[,!(names(TRN_REM_na) %in% colRm_TRN)]
        TST_colRm <- TST_REM_na[,!(names(TST_REM_na) %in% colRm_TST)]
     
     Determination of Dim
        dim(TRN_colRm)
        [1] 19622   121
        dim(TST_colRm)
        [1] 20 53
     
 # Evaluation of Training & Validation Set
     Remove 1st column of ID

        TRNRaw <- TRN[,-1]
        TSTRaw <- TST[,-1]

        table(TRN$classe)

            A    B    C    D    E 
          5580 3797 3422 3216 3607 

         dim(TRNRaw)
         [1] 19622   158
         dim(TSTRaw)
         [1]  20 158
         
         inTrain <- createDataPartition(y=TRN$classe, p=0.7, list=FALSE)
  
         TRN_clean = TRNRaw[inTrain,]
         validation_clean = TRNRaw[-inTrain,]
         TST_clean<-TRN[-inTrain, ]
      
Remaining of Data after removal of missing value < 60%

        RMN<- c((colSums(!is.na(TRN_clean[,-ncol(TRN_clean)])) >= 0.6*nrow(TRN_clean)))
        TRN_clean<-TRN_clean[,RMN]
        validation_clean<- validation_clean[,RMN]
  
    
Both created datasets have 158 variables. Those variables have plenty of NA, that can be removed with the cleaning procedures below. The Near Zero variance (NZV) variables are also removed and the ID variables as well.

        nsv <- nearZeroVar(TRN_clean,saveMetrics=TRUE)
        TRN <- TRN[,!nsv$nzv]
        TST<- TST[,!nsv$nzv]

        TRN_clean <- TRN_clean[,!nsv$nzv]
        TST_clean<- TST_clean[,!nsv$nzv]
  
        dim(TRN_clean)
        [1] 13737    49
        dim(TST_clean)
        [1] 5885   62
  
        TRN_clean <- TRN_clean[, -(1:5)]
        TST_clean<- TST_clean[, -(1:5)]
  
        dim(TST_clean)
        [1] 5885   57
        dim(TRN_clean)
        [1] 13737    44
  
# Modeling In random forests
there is no need for cross-validation or a separate test set to get an unbiased estimate of the test set error. It is estimated internally, during the execution. Therefore, the training of the model (Random Forest) is proceeded using the training data set

       model <- randomForest(classe~.,data=TRN_clean)
       model

        Call:
        randomForest(formula = classe ~ ., data = TRN_clean) 
             Type of random forest: classification
                   Number of trees: 500
        No. of variables tried at each split: 7

            OOB estimate of  error rate: 0.53%
        Confusion matrix:
            A    B    C    D    E class.error
        A 3901    4    0    0    1 0.001280082
        B   12 2642    4    0    0 0.006019564
        C    0   16 2377    3    0 0.007929883
        D    0    0   26 2224    2 0.012433393
        E    0    0    1    4 2520 0.001980198
  
Model Evaluation Verification of the variable importance measures as produced by random Forest is as follows:

       importance(model)
          
                              MeanDecreaseGini
         roll_belt                   847.16481
         pitch_belt                  475.73462
         yaw_belt                    626.30922
         total_accel_belt            167.73014
         gyros_belt_x                 64.39051
         gyros_belt_y                 79.06652
         gyros_belt_z                224.15580
         accel_belt_x                 81.52818
         accel_belt_y                 86.94999
         accel_belt_z                267.51839
         magnet_belt_x               173.73544
         magnet_belt_y               274.78199
         magnet_belt_z               290.53285
         roll_arm                    238.95758
         pitch_arm                   127.13482
         yaw_arm                     175.87481
         total_accel_arm              68.14704
         gyros_arm_x                  94.48493
         gyros_arm_y                  95.73867
         gyros_arm_z                  44.80364
         accel_arm_x                 173.47008
         accel_arm_y                 108.79793
         accel_arm_z                  93.52822
         magnet_arm_x                174.79547
         magnet_arm_y                162.11858
         magnet_arm_z                129.55268
         roll_dumbbell               301.15935
         pitch_dumbbell              126.22889
         yaw_dumbbell                183.51838
         total_accel_dumbbell        187.67506
         gyros_dumbbell_x             90.27847
         gyros_dumbbell_y            171.94444
         gyros_dumbbell_z             62.65224
         accel_dumbbell_x            170.99000
         accel_dumbbell_y            290.27433
         accel_dumbbell_z            236.13039
         magnet_dumbbell_x           328.78585
         magnet_dumbbell_y           467.37643
         magnet_dumbbell_z           534.06264
         roll_forearm                410.66255
         pitch_forearm               549.01214
         yaw_forearm                 119.68493
         total_accel_forearm          77.58351
         gyros_forearm_x              53.12716
         gyros_forearm_y              90.04705
         gyros_forearm_z              61.04178
         accel_forearm_x             229.30825
         accel_forearm_y             100.03163
         accel_forearm_z             163.73636
         magnet_forearm_x            154.81933
         magnet_forearm_y            155.07915
         magnet_forearm_z            197.49084
         
Next, the model results is evaluated through the confusion Matrix.

        install.packages(“e1071”)
        confusionMatrix(predict(model,newdata=validation_clean[,-ncol(validation_clean)]),validation_clean$classe)
        Confusion Matrix and Statistics
               Reference
          Prediction    A    B    C    D    E
                   A 1673    2    0    0    0
                   B    1 1137    6    0    0
                   C    0    0 1020    8    0
                   D    0    0    0  955    3
                   E    0    0    0    1 1079

          Overall Statistics
                                      
                       Accuracy : 0.9964          
                           95% CI : (0.9946, 0.9978)
            No Information Rate : 0.2845          
            P-Value [Acc > NIR] : < 2.2e-16       
                                          
                      Kappa : 0.9955          
          Mcnemar's Test P-Value : NA              

          Statistics by Class:
                                  Class: A Class: B Class: C Class: D Class: E
           Sensitivity            0.9994   0.9982   0.9942   0.9907   0.9972
           Specificity            0.9995   0.9985   0.9984   0.9994   0.9998
           Pos Pred Value         0.9988   0.9939   0.9922   0.9969   0.9991
           Neg Pred Value         0.9998   0.9996   0.9988   0.9982   0.9994
           Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
           Detection Rate         0.2843   0.1932   0.1733   0.1623   0.1833
           Detection Prevalence   0.2846   0.1944   0.1747   0.1628   0.1835
           Balanced Accuracy      0.9995   0.9984   0.9963   0.9950   0.9985
      
The accurancy for the validating data set

          Accur<-c(as.numeric(predict(model,newdata=validation_clean[,-ncol(validation_clean)])==validation_clean$classe))
          Accur<-sum(Accur)*100/nrow(validation_clean)
          Accur
         [1] 99.64316
  
Model Accuracy as tested over Validation set = 99.64316%

Prediction with the Testing Dataset

         predictions <- predict(model,newdata=TSTRaw[-1,])
         predictions
          2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
          A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
           Levels: A B C D E
  # correlation between predictors and outcome
To evaluate correlation between predictors and outcome ,random forest model is better than linear regression model

        cor <- abs(sapply(colnames(TRN_clean[, -ncol(TRN)]), function(x) cor(as.numeric(TRN_clean[, x]), as.numeric(TRN_clean$classe), method = "spearman")))
    
Create Random Forest Model We try to fit a random forest model and check the model performance on the validation set.

        RanForFit <- train(classe ~ ., method = "rf", data = TRN_clean, importance = T, trControl = trainControl(method = "cv", number = 4))
 
          RanForFit
          Random Forest 

          13737 samples
          52 predictor
          5 classes: 'A', 'B', 'C', 'D', 'E' 

          No pre-processing
          Resampling: Cross-Validated (4 fold) 
          Summary of sample sizes: 10303, 10302, 10304, 10302 
          Resampling results across tuning parameters:

          mtry  Accuracy   Kappa    
          2    0.9901723  0.9875677
         27    0.9906091  0.9881205
         52    0.9842032  0.9800152

Accuracy was used to select the optimal model using  the largest value.
The final value used for the model was mtry = 27.
 
        impVar<- varImp(RanForFit)$importance
        varImpPlot(RanForFit$finalModel, sort = TRUE, type = 1, pch = 20, col = 3, cex = 1, main = "Predictors' Importance ")
        
# Conclusion
Acccording to exploratory data analysis, fitting of appropriate model with high accuracy for prediction of sampla outcomes requires to remove missing value and near zero data therefore creating validation will be accurate with random forest model with cross validation .
