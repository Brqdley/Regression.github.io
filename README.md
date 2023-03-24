# Eletrical Outages Regression Model Building
Link to Exploratory Data Analysis : https://brqdley.github.io

# Framing the Problem
My prediction problem is estimating the number of customers affected by an electrical outage(Quantitative) at the start of an outage.
I chose this variable because I think this would be useful by the electrical companies to gauge how much of an impact the outage will have,
so they know how much manpower to use.

At the time of predicting, the restoration dates, and the outage duration would not be available, 
as the values would not be available at the start of an outage. Therefore, I cannot use these as features for the regression model, and have to stick with features that are available at the start of an outage, such as environmental conditions and location, and information about the electrical company that is gathered from previous days.


*Metric: Root Mean Squared Error*\
I used this metric because since this is a regression model, accuracy would not work, as that is for classifiers. Moreover, I chose RMSE over other metrics because it penalizes large errors more severely than small errors, which makes it a good metric for detecting outliers. It also has the same unit of measurement as the dependent variable, making it easier to interpret. 

# Baseline Model

*Model*\
The model is a linear regression model with a preprocessor that uses a column transformer to preprocess the data. The preprocessor consists of three transformers, including a datetime transformer that converts the OUTAGE.START.DATETIME.DAY feature into an integer, a numerical transformer that imputes missing values using the mean and scales the POPULATION feature using standard scaling, and a categorical transformer that imputes missing values using the most frequent value and encodes the CAUSE.CATEGORY feature using one-hot encoding.

*Features*\
The model is trained on a subset of the features in the dataset, including CAUSE.CATEGORY, POPULATION, and OUTAGE.START.DATETIME.DAY.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CAUSE.CATEGORY This is a nominal feature.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POPULATION This is a quantitative feature.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OUTAGE.START.DATETIME.DAY This is a datetime feature which I converted into a quantitative integer feature.  

For this model the root mean squared error is 262170.52, which I would not consider very good. Although CUSTOMERS.AFFECTED is in the hundredths of thousands, this still is very high variability.


# Final Model
I added several new features to my model, including CLIMATE.CATEGORY, TOTAL.CUSTOMERS and COM.SALES. These features are relevant to the prediction task as they could have an impact on the number of customers affected during an outage. The CLIMATE.CATEGORY could have a trend for the extend of the outage. If there is an intentional attack, the extent of the outage would most likely be greater than severe weather. The TOTAL.CUSTOMERS could be correlated to many customers the outage could affect, if the number of customers that use this electricity is very small, the number of customers affected would most likely be small as well. COM.SALES could also be similar as it represents the commercial sales, which could also be correlated with the number of customer's affected. as a larger community would have more customers.

I modified my model to use a Decision Tree Regressor instead of a Linear Regression model. This is a better choice for the current data as it is more flexible and can capture non-linear relationships between features and the target variable.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CAUSE.CATEGORY: This is a nominal feature.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POPULATION: This is a quantitative feature.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OUTAGE.START.DATETIME.DAY: This is a datetime feature which I converted into a quantitative integer feature.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CLIMATE.CATEGORY: This is a nominal feature.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TOTAL.CUSTOMERS: This is a quantitative feature.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;COM.SALES: This is a quantitative feature.   
To select the best hyperparameters, I  used GridSearchCV using root mean squared error to quantify performance and used different values for max_depth,n_neighbors, and min_samples_split for two regressors, NNeighborsRegressor and DecisionTreeRegressor. The best combination, one with the lowest root mean squared error, was found to be a DecisionTreeRegressor with a max_depth=2 and min_samples_split=100.

Compared to the baseline model that only used three features (CAUSE.CATEGORY, POPULATION, and OUTAGE.START.DATETIME.DAY), the final model with seven features and a decision tree regressor has improved the performance. The root mean squared error decreased from the baseline score of 262,170.52 to a score of 178,114.17. This indicates that the additional features and the decision tree model have helped to better capture the patterns in the data and improve the accuracy of the predictions.

# Fairness Analysis
For this fairness analysis, I chose Group X as the outages with an ANOMALY.LEVEL greater than or equal to 0, and Group Y as the outages with an ANOMALY.LEVEL under 0. The evaluation metric is the root mean squared error of the predicted number of customers affected by an outage.

The null hypothesis is that there is no difference in the root mean squared error of the predictions between Group X and Group Y. The alternative hypothesis is that there is a significant difference in the root mean squared error of the predictions between Group X and Group Y.

I used a permutation test with 1000 permutations as the method to test the null hypothesis. The test statistic is the difference in root mean squared error between Group X and Group Y. The significance level is 0.05.

The p-value of the test is 0.88, which is greater than the significance level of 0.05. Therefore, we cannot reject the null hypothesis and can conclude that there is no evidence that there is a significant difference in the root mean squared error of the predictions between Group X and Group Y.

Based on this analysis, we can conclude that the model is most likely not worse in predicting the number of customers affected by an outage for outages with an ANOMALY.LEVEL greater than or equal to 0 compared to those with an ANOMALY.LEVEL under 0. 


