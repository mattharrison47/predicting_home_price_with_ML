Introduction

The dataset provided to us consisted of a training dataset and testing dataset. The training data set includes 71 columns, 70 features and 1 target variable, and 1385 rows. The testing data set includes 70 columns of the same features, missing the target variable, and 1459 rows. The features in the data include both categorical and numerical attributes. First, I read in the data into a pandas dataframe by using pd.read_csv, and pulling initial observations from .info(), .head(), and shape commands. I generated a pandas profiling report as well to gain insight into the various features and their relation to each other. Next, before starting on the numerical and categorical features, I wanted to get an understanding of the target variable, ‘SalePrice’. Based on the overview of the data, we have information on features of houses and we are trying to predict the price at which they sold, in what looks to be dollar amounts. I pulled a distribution plot and boxplot of the target variable using the Seaborn package, seen below in Figure 1. It looks like there is a main distribution between 100,000-300,000 and some outliers at the higher end (400,000+). However, I chose to leave the target variable as is because I did not want to affect how the features correspond to sale price. Perhaps there are some specific features at the higher ‘SalePrice’ end that push them to a higher price. For this same reason, I did not remove outliers. I felt that a comprehensive data cleaning process would help deal with the outliers as well. Additionally, the target variable is continuous, so we will need to clean the data and use machine learning models with the goal of regression in mind.
   
Figure 1 – Seaborn distplot and boxplot on target variable ‘SalePrice’.

II. Numerical Data

Next, to start the data engineering process, I removed SalePrice from the training dataset. I then determined which columns were numerical attributes (based on type 'int16', 'int32', 'int64', 'float16', 'float32', 'float64') and categorical attributes (if type was anything else). By my count there were 30 numerical features and 40 categorical features. I then stacked the training and test data into one dataframe so that I made sure to engineer identical changes on both. The first thing I did was to check the numerical features for columns with missing data. I then filled the missing values with either a 0 or the median value of the column. I did a few different types of changes to the numerical data so that it would be better read by the machine learning models. The first change I did was to normalize the distributions of numerical features that I felt needed to be kept in numerical form because there was a wider range of values that would better inform the models. I did this using a Box-Cox transform from the stats package. For example, in Figure 2 you can see the changes in a distribution plot made to the ‘1stFlrSF’ feature. 

    
Figure 2. ‘1stFlrSF’ feature before (left) and after Box-Cox Transformation (right). 

Next, some numerical features were essentially turned into categorical features by process of binning. Take for example the category ‘2ndFlrSF’. As you can see in Figure 3, rows have this as a ‘0’ value, while a number of other rows have a value greater than 0 across a distribution. To keep some variation in the data, this was binned into 3 categories or ranges – (0, 0-700, 700+). This same process was used to bin a number of numerical features where distribution varied.

   
Figure 3. ‘2ndFlrSF’ distribution before binning (left) and after binning (right) using pandas.cut command.

Any features with greater than 95% of data at a single value from the pandas profiling report were deleted from the data set to avoid overfitting. There were also some features that remained unchanged based on their current distribution, which already somewhat resembled a normal distribution. The last change I made was to turn some numerical features into type ‘category’. I did this with some features that corresponded to years, because year would be more indicative of value versus a regular numerical distribution. All changes to numerical features can be seen in Appendix Table 1. (Appendix included at end of the report). 

III. Categorical Features

I then looked at making changes to the categorical features. Among the 40 categorical features, there were 23 columns with missing data. I filled in these missing spaces with ‘NA’. Then, looking at the pandas profiling report, I deleted any columns with greater than 90% of values corresponding to one value or missing data to avoid overfitting and bias. Some columns showed a good distribution of features (no less than 5% in any one category), so I left them unchanged. However, it seemed that a lot of categorical attributes did not have an even distribution. Rather, they had a couple categories that had a small percentage of the distribution, and a couple that had most to the distribution of values. This would cause machine learning models to show bias to the small few in the lower percentage categories. To solve this issue, I used binning to lump together categories with smaller values. This should help with generalization and variation when modeling. For example, in Figure 3 you can see the changes made to the ‘HeatingQC’ column. The categories ‘Gd’, ‘Fa’, and ‘Po’ were binned together into one category to give a better distribution of categories. Binning was implemented by building functions on conditional statements for different categories, and observations on the distributions of categories pulled from the pandas profiling report. Lastly, the column ‘FireplaceQu’ was deleted because it was missing 47% of data. All changes to the categorical features are summarized in Appendix Table 2. 

   
Figure 3. ‘HeatingQC’ before (left) and after binning (right) using 




IV. Final Engineering Tasks and Summary

After all changes made to the numerical and categorical data, the remaining dataset consisted of 47 columns or features, 6 of which remained numerical, while the rest were now categorical. The next step in the data engineering process was to make the data was readable for regression based machine learning models. To do this, all categories would need to be numerical in nature. First, the numerical features that were made into categorical features via binning were encoded with label encoding from the sklearn preprocessing package ‘LabelEncoder’. This will transform their ‘type’ to numerical in nature (for example type int or float). This was performed because this categorical data is hierarchical – generally, more features should most likely mean higher home sale price, the target variable. All other categorical data was one-hot encoded to transform them into separate columns with numerical values. This was done with the pandas ‘get_dummies” package. After encoding, there were now 98 columns. To avoid overfitting again, any columns with greater than 98% with the value ‘0’ were dropped from the dataset. This removed 9 columns, all originally from the ’Neighborhood’ column that had been one-hot encoded. The entire dataset was then standardized with the sklearn.preprocessing package “StandardScaler” so it could be uniformly read by each model and all numerical values are on the same scale. Finally, the training and test sets were again split from each other into separate dataframes, and the target variable was added back to the training set.

V. Modeling and Conclusions

For the machine learning models used, the same evaluation process was followed for multiple models. The training and target columns were split into an 80% training set and 20% validation set using sklearn ‘train_test_split’. The model was first fit on the 80% set. Then the model was used to predict the values for the target variable on the 20% validation set and the root mean squared error was calculated versus the actual target variable values. Root mean squared error was used because it would give a sense of how much of a “dollar” amount I was off by. The same random state (42) was used for each training and test set for reproducibility. Then separately, after this initial 80/20 test split, cross validation was then performed for each model using 10 folds. The 10 folds were split among the entire training data set. The mean, lower, and upper 95% confidence interval was calculated for each model from the sklearn.model_selection package ‘cross_val_score’.

The first model used was linear regression from sklearn.linear_model “LinearRegression”. Initially no changes were made to this model, and performance was mediocre. Next, ridge regression and lasso regression for linear regression were used to see if they could help reduce collinearity that might be found in the training set. The performance was marginally improved. 

The next model used was Decision Tree Random Forests with the sklearn.ensemble package “RandomForest Regressor”. Initially no changes were made to the default parameters of the model. This model showed improved RMSE on the 80/20 split and a higher upper and lower confidence interval. Gridsearch for the model showed to be too intensive a process for my computer to perform (hardware issue) so the number of estimators was increased to 500 and the performance increased slightly again. 

Next, Support Vector Machine was used from the sklearn.svm package “SVR”. This had much worse root mean squared error, almost twice the value of the root mean squared error for linear regression. A gridsearch from the package ‘GridSearchCV’ was performed on the SVR model find the best values for the parameters ‘C’, ‘gamma’, and ‘kernel’. The best values found in the gridsearch were ‘C’=1000, ‘gamma’ = 0.01, and ‘kernel’=rbf. These changes were implemented, and performance improved. However, overall performance was still much worse than other models. 

Gradient Boosted Trees were looked at with the ‘xgboost’ package ‘XGBRegressor’. The default parameters were used for this model and it showed about equal mean squared error to RandomForestRegressor, and improved values for the upper and lower 95% confidence interval. A gridsearch was performed for the model parameters for ‘n_estimators’, ‘gamma’, ‘learning_rate’, ‘subsample’, and ‘max_depth’. The best values were found to be ‘gamma’=0.8, ‘learning_rate’=0.1, ‘max_depth’=4, ‘n_estimators’=50, and ‘subsample’=0.7. The model was used with these parameters and the root mean squared error on the 80/20 split slightly improved, however, the confidence intervals were approximately unchanged. To show generally how the model performs, you can see the kde plot below in Figure 4. This figure plots the 20% predicted values versus the actual predicted values in the 80/20 validation split. You can see that it closely matches the distribution of ‘SalePrice’. Additionally, to see what features influence this model, I suggest looking at Appendix Table 1. It looks like ‘OverallQuality’ has a significantly high influence to the model, but there is enough variation to where it seems balance. 

 
Figure 4. XGB predicted values vs. actual sale price distribution plot in 20% validation set. 

Logistic Regression was used next with the sklearn.linear_model package “LogisticRegression”. This showed poor performance the train test split and cross validation. 

K-Nearest Neighbors was next performed with the sklearn.neighbors package “KNeighborsRegressor”. Before fitting the model, I needed to find the best value for the “K” or number of neighbors to use for clustering. To do this, root mean square error on the validation set (prediction vs. actual) for the train test 80/20 split was calculated for ‘K’ in a range of k=1 to k=30. The results of this calculation can be seen in Figure 5. Choosing ‘K’ at the “elbow” equated in ‘K’ = 5 for lowest error. This K value was used in the train test 80/20 split and the cross validation resulting in decent performance. 

 
Figure 5. K-Value (1 to 30) for KNN Regressor versus Root Mean Squared Error for target variable ‘SalePrice’ in validation set. 

A Decision Tree was used from the sklearn.tree package “DecisionTreeRegressor”. The default parameters were kept and the model showed good performance for the train/test split and cross-validation, albeit not the best. 

Model	80/20 Split Validation Set Root Mean Squared Error	Cross Validation 95% CI Mean Score	Cross Validation 95% CI Upper and Lower Bounds (L, U)
Linear Regression	41,064.45	0.8217	(0.7719, 0.8714)
Linear Regression (Ridge)	41,047.45	0.8218	(0.7721, 0.8715)
Linear Regression (Lasso)	41,061.45	0.8217	(0.7719, 0.8714)
Random Forest	32,814.48	0.8430	(0.7691, 0.9168)
Support Vector Machine	62,571.30	0.3756	(0.3183, 0.4330)
Gradient Boosted Trees	31,714.82	0.8586	(0.8098, 0.9074)
Logistic Regression	50,182.92	0.0101	(0.0016, 0.0019)
KNN	40,677.23	0.7450	(0.6901, 0.7999)
Decision Tree	39,875.85	0.8398	(0.7649, 0.9146)
Table 1. Summarized machine learning model performance. 
Performance for the 9 different machine learning models used can be seen in Table 1. Based on best overall performance and stability in the 10-fold cross validation, the best model was chosen to be Gradient Boosted Trees. This model shows a good balance of feature importance and distribution of predicted values closely follows actual target values. It would seem, logically, being able to predict a house sale price within about $30,000 is a pretty good model. Finally, for submission of the test set predictions, the ‘XGBRegressor’ model was then retrained on the entire training data set, and used to predict the target variable values. 



VI. Appendix 
 
1stFlrSF	Box-Cox Transformation to normalize distribution
2ndFlrSF	Binned in 3 ranges (0, 0-700, 700-max)
GrLivArea	Box-Cox Transformation to normalize distribution
BsmtFullBath	Binned in 2 ranges (0, 1-max)
FullBath	Binned in 2 ranges (0-1, 2-max)
HalfBath	Binned in 2 ranges (0, 1-max)
BedroomAbvGr	Binned in 3 ranges (0-2, 3, 4+)
KitchenAbvGr	> 95% of rows belonged to same value so deleted due to bias
BsmtHalfBath	> 95% of rows belonged to same value to deleted due to bias
3SssnPorch	> 95% of rows belonged to same value to deleted due to bias
PoolArea	> 95% of rows belonged to same value to deleted due to bias
MiscVal	> 95% of rows belonged to same value to deleted due to bias
MasVnrArea	Binned in 2 ranges (0, 1-max)
ScreenPorch	Binned in 2 ranges (0, 1-max)
TotRmsAbvGrd	Binned in 6 ranges (1-4, 5, 6, 7, 8, 9-max)
Fireplaces	Binned in 2 ranges (0, 1-max)
GarageYrBlt 	Made into category 
OpenPorchSF	Binned in 2 ranges (0, 1-max)
BsmtFinSF1	Unchanged 
BsmtUnfSF	Unchanged
GarageCars	Binned in 4 ranges (0,1,2,3-max)
WoodDeckSF	Binned in 2 ranges (0, 1-max)
LotArea	Box-Cox Transformation to normalize distribution
OverallQual	Binned in 6 ranges (0-4,5,6,7,8,9-max)
OverallCond	Binned in 3 ranges (0-5, 6, 7-max)
YearBuilt	Binned in 4 ranges (0-1950, 1951-1980, 1981-2000, 2001-max)
YearRemodAdd	Binned in 2 ranges (0-1980, 1990-max)
YrSold	Made into category 
MoSold	Made into category
TotalBsmtSF	Unchanged
Appendix Table 1. Numerical feature changes

Heating	> 95% of rows belonged to same value so deleted due to bias
Electrical	> 95% of rows belonged to same value so deleted due to bias
Functional	> 95% of rows belonged to same value so deleted due to bias
FireplaceQu	Missing 47% of data so deleted
GarageQual	> 95% of rows belonged to same value so deleted due to bias
GarageCond	> 95% of rows belonged to same value so deleted due to bias
PavedDrive	Binned into 2 categories 
Street	> 95% of rows belonged to same value so deleted due to bias
Alley	94% missing data so deleted
Utilities	> 95% of rows belonged to same value so deleted due to bias
Condition1	> 90% of rows belonged to same value so deleted due to bias
Condition2	> 95% of rows belonged to same value so deleted due to bias
ExterCond	Binned into 2 categories 
BsmtCond	Binned into 2 categories
BsmtExposure	> 90% of rows belonged to same value so deleted due to bias
BsmtFinType1	Deleted due to high bias with other features
SaleType	Binned into 2 categories 
MiscFeature	> 95% of rows missing data so deleted 
HeatingQC	Binned into 3 categories
CentralAir	> 95% of rows belonged to same value so deleted due to bias
LandContour	> 90% of rows belonged to same value so deleted due to bias
Exterior2nd	High correlation with Exterior1st so deleted due to collinearity
KitchenQual	Binned into 2 categories
BsmtQual	Binned into 2 categories
Fence	Binned into 2 categories
MSZoning	Binned into 2 categories
GarageType	Binned into 2 categories
GarageFinish	Binned into 2 categories
LotShape	Binned into 2 categories
LotConfig	Binned into 3 categories 
Neighborhood	Unchanged
BldgType	Binned into 2 categories 
HouseStyle	Binned into 2 categories 
RoofStyle	Binned into 2 categories 
Exterior1st	Binned into 5 categories 
MasVnrType	Binned into 2 categories
ExterQual	Binned into 2 categories
Foundation	Binned into 3 categories
PoolQC	> 95% of rows belonged to same value so deleted due to bias
SaleCondition	Binned into 2 categories
Appendix Table 2. Categorical feature changes 
 
Appendix Figure 1. Feature importance (percentage) for Gradient Boosted Trees 
