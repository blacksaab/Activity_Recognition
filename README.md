# Analysis of Smartphone Based Activity Recognition 

The data for this analysis is provided courtesy of [Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. A Public Domain Dataset for Human Activity Recognition Using Smartphones. 21th European Symposium on Artificial Neural Networks, Computational Intelligence and Machine Learning, ESANN 2013. Bruges, Belgium 24-26 April 2013](https://archive.ics.uci.edu/dataset/240/human+activity+recognition+using+smartphones). 

## Introduction 

The motivation behind this project is to classify different human activities based on sensor data colelcted from a common smartphone. The activities performed include: 

1. Walking 
2. Walking Upstairs 
3. Walking Downstairs 
4. Sitting 
5. Standing 
6. Laying 

The data was collected from 30 volunteers and has been randomly partitioned into two sets. 


## Exploratory Data Analysis 

This data has $561$ features - which is both a blessing and a curse. The large number of distinct features makes it easy to classify the different activities, however it makes any model fit on all features difficult to interpret. 

The data for this set are calculated by taking the average of a $128$-wide rolling window with $50\%$ overlap. For example, the X-Axis Acceleration is calculated by averaging $2.56$ second long windows (which overlap each other by $50\%$). This means that the rows are average readings taken $1.28$ seconds apart from each other. 

The following plot shows the X-axis acceleration for all 6 activity types side-by-side. 

<!-- Insert image of activity plots -->
![X-axis acceleration for all activities](/images/Activities.png)

We see that each activity has a distinct pattern. The following plot shows a histogram for the X-axis acceleration for each activity type. 

![Distribution of X-axis acceleration for each activity](/images/Activities.png)

This plot further shows that for X-axis acceleration, each activity type has its own distinct distribution. Specifically, there is a striking difference between the moving activities (walking, walking upstairs, and walking downstairs), and the non-moving activities (sitting, standing, and laying). 

While it would be interesting to perform similar EDA on every variable, this is not pheasible because there are $560$ more features to look at. 


## Overview of Models 

I tested and optimized several model types. I fit each model using both the full set of features, as well as a UMAP-reduced dataset. 

### K-Neighbors Classifier 

Both KNN classifiers performed exceptionally. The classifier fit on all features (`Scale_KNN`) achieved a training accuracy of $0.98$ and a testing accuracy of $0.91$, and the classifier fit on UMAP-reduced-data (`UMAP_KNN`) achieved a training accuracy of $1.0$ and a testing accuracy of $0.9$. 

While the `UMAP_KNN` model performed better on the training data, it performed significantly worse on the test data. This indicates that `UMAP_KNN` overfits the data. 

### Decision Tree 

Both Decision Tree Classifiers performed very well. The decision tree fit on the full feature set (`CART`) achieved a training accuracy of $0.93$ and a testing accuracy of $0.86$, while the tree fit on the dimensionally reduced data achieved a training accuracy of $1.0$ and a testing accuracy of $0.89$. 

In this case, it appears that the use of UMAP significantly improves the performace of the decision tree model. However, this method still produced poorer results than the other methods. 

### Neural Net 

The Multi-Layer Perceptron fit on the full feature set (`MLP`) achieved a training accuracy of $1.0$ and a testing accuracy of $0.95$, while the Multi-Layer Perceptron fit on the dimensionally reduced data achieved a training accuracy of $1.0$ and a testing accuracy of $0.90$. 

In the case of the neural net, it appears that using UMAP let to overfitting and reduced the test performance of the model. 


## Model Selection 

In the interest of brevity I have only discussed accuracy thusfar. While all models performed similarly, the model which performed best overall (considering all classification metric; not just accuracy) is the MLP fit on the full feature set (`MLP`). In precision, recall, f1-score, and accuracy, `MLP` outperformed all other models when predicting the test data. 


## Discussion of Best Model 

### Classification Metrics

The output from the `classification_report()` function for `MLP` is the following: 

                  precision    recall  f1-score   support

               1       0.92      0.98      0.95       496
               2       0.93      0.91      0.92       471
               3       0.98      0.95      0.96       420
               4       0.95      0.89      0.92       491
               5       0.91      0.96      0.93       532
               6       1.00      0.99      1.00       537

        accuracy                           0.95      2947
       macro avg       0.95      0.95      0.95      2947
    weighted avg       0.95      0.95      0.95      2947

Precision is the proportion of correctly predicted observations out of the total number of predicted observations for a particular class. We see that `MLP` has the best accuracy when classifying activity 6 (laying down), and has the worst accuracy when classifying activities 1 and 5 (walking and standing respectively). This indicates that activities are most likely to be misclassified as walking or standing. 

Recall is the proportion of correctly classified observations out of the total number of predictions for a praticular class. `MLP` has relatively low recall when classifying activity 4 (sitting) and activity 2 (walking upstairs). This indicates that `MLP` misclassifies these classes most frequently. 

Overall, this model still performs very well. The accuracy, which is the total number of correct predictions out of the total number of predictions, is $95\%$. 

### SHAP

In order to show which features were the most important in making predictions, I calculated SHAP values. 

![shap bar plot](/images/shap_bar_plot.png)

This plot shows the marginal importance of each feature. I was hoping that only a few features would account for most of the prediction; however, we see that the vast majority of importance comes from the "$552$ other features". This means that a large number of features are used by the model when making predictions. 

Interestingly, the most important features are mostly "X" features (tGravityAcc-energy()-X, tBodyGyroJerk-entropy()-X, etc). 

![shap beeswarm plot](/images/shap_beeswarm_plot.png)


## Next Steps 

While I am able to predict the activity type to a high degree of accuracy, the results are less helpful than I would like because I made use of all $561$ features. The huge number of features cannot be quickly calculated, which limits the application of my work. The clear next step is to create a model which predicts activity type using as few features as possible without sacrificing too much accuracy. 
