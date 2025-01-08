# LA Collisions Project

## Introduction
Residents of Los Angeles City and County live with traffic collisions as a day to day concern. Similarly, collisions are a major concern for governmental agencies, who must make policy decisions that aim to prevent as many vehicle collisions as possible, and first responders, who must allocate limited resources in a way that prioritizes the accidents that are most severe. Traffic congestion and the frequency of accidents in Los Angeles complicates first response efforts. Police, firefighters, and ambulances commonly have to navigate gridlocked traffic to respond to multiple collisions occuring at the same time across the city. Consequently, an analysis of the existing data on traffic collisions in Los Angeles is warranted, to help improve the precision and efficiency of these efforts. 

## Dataset
+ **Sources**: Data acquired from Kaggle, titled "Los Angeles Traffic Collision Data" (https://www.kaggle.com/datasets/cityofLA/los-angeles-traffic-collision-data/data?select=traffic-collision-data-from-2010-to-present.csv). I used this source from LA City to translate the MO Codes from the police report into relevant information about the collision: https://data.lacity.org/api/views/d5tf-ez2w/files/8957b3b1-771a-4686-8f19-281d23a11f1b?download=true&filename=MO_CODES_Numerical_20180627.pdf

+ **Structure**: This data shows information like the police report number, date reported, date occurred, time occurred, area information, victim demographic information, MO codes from the police report, zip codes, and geospatial coordinates. The MO Codes describe specific circumstances of collisions as reported in police reports (such as a pedestrian being involved).

## Tools Used ##
All analysis for this project was done in Python with the help of the following libraries:
+ Pandas
+ Numpy
+ Scikit-learn
+ Folium
+ Matplotlib
+ Seaborn

## Data Collection and Preprocessing ##

The data from the source was not in very useful format, so required quite a bit of preprocessing before analysis could begin. 

First, the MO Codes were formatting as strings, like "3036 3004 3026 3101 4003". Each of these codes means that a certain circumstance was present in the accident. For example "3036" means that the accident occurred at an intersection. The first data cleaning step was to translate these codes into useful binary variables, defined in columns in the data frame. For example, if a pedestrian was involved in the collision it would be coded as "3036" or "3501", based on the MO Codes information. So if either of those values were present in the MO Codes string, the new column "Pedestrian_involved" would take a value of 1, otherwise it would be coded as a 0. I coded as many independent variables as seemed relevant in predicting the severity of the outcome of the collision. The outcomes "No_injury", "Severe_injury", "Fatality", and "Other_injury" were also coded as new columns using the MO Codes data, and those variables will be considered the dependent variables in this analysis. 

The next data cleaning steps were to drop unnecessary columns from the dataset, and format data and time correctly. Times were given as strings in the original data. For example, "1341" would indicate a time of 1:41 PM. I created a function to convert all these strings into correctly formatted times. Later, I also coded the times into a categorical variable, with categories of 'Late Night', 'Early Morning', 'Morning', 'Afternoon', 'Evening', 'Night' corresponding to 4 hour blocks of time starting from midnight to 4 AM ('Late Night') and so on. 

Finally, Latitude and Longitude were listed as strings in a tuple-esque format, as such: "(34.0255, -118.3002)". For the purposes of creating a folium map of fatal accidents later in the analysis, I extracted the latitude and longitude values into separate columns in the data frame. 

At this point the data was now in a useful format for conducting analysis. 

## Exploratory Data Analysis
### Time Series Trends

The first thing I wanted to visualize were the time series trends of each category of collision. Starting with fatal accidents, there's a clear uptrend over time: 

![download](https://github.com/user-attachments/assets/e51b83be-fbfe-40da-ad68-db16a497121b)

Similarly, the trend for collisions resulting in severe injuries is also upsloping: 

![download](https://github.com/user-attachments/assets/cbeb5c6b-4a39-4021-8cb9-5472849af2f4)

Conversely, the trends for total collisions and collisions categorized as "other injuries" peak around 2017-2019, and sharply decline after that: 

![download](https://github.com/user-attachments/assets/0a2140d3-62e0-4d41-8a42-8038fd36815a)
![download](https://github.com/user-attachments/assets/7724f8b1-cad5-43b9-9d9f-63d15eaf4052)

This discrepancy begs the question why collisions resulting in severe and fatal injuries have been rising, while overall collisions have been falling. Explaining that definitively is beyond the scope of this analysis, but some possible explanations could be changes in reporting practices, technological improvements resulting in fewer minor accidents, while perhaps people paying less attention or using phones while driving could in part explain the increase in severe accidents. The divergence in these trends warrants additional investigation. 

I also looked at time series of collisions involving pedestrians and cyclists. 
![download](https://github.com/user-attachments/assets/4e8545b1-cbbd-453e-b1a4-753e02997cf9)
![download](https://github.com/user-attachments/assets/6190f05d-69fb-458d-bf6e-be24a84d4fd5)

Collisions involving cyclists peaked in 2013 and have been on the decline since then, while collisions involving pedestrians peaked in 2019 and have declined since that time. The time series involving collisions with pedestrian involvement looks quite similar to the overall collisions time series, while for collisions involving cyclists, the declining trend started earlier. This could be due to greater awareness on the part of drivers to cyclists or to policies that encouraged cycling by creating new bike lanes or making bike lanes more segregated from normal traffic. According to LAIST, the city had 245 miles of bicycle infrastructure in 2005, which grew to over 562 miles in 2015. This association needs more supporting evidence, but it appears to suggest a return on investment to the city in bicycle infrastructure. Collisions involving bicycles decreased from about 2500 in 2013 to around 500 in 2023. 
+ Source: https://laist.com/news/kpcc-archive/watch-a-decade-of-growth-in-la-s-bike-infrastructu

### Sex and Ethnicity Statistics
Bar graphs of number of collisions by sex and ethnicity show some interesting correlations. Clearly, more collisions involved men than women - a finding that aligns with prior data suggesting men cause about 60% of accidents in LA county while women cause approximately 40%. Men are also consistently killed at 2-3 times the rate that women are in motor vehicle accidents in the US as a whole. 
![download](https://github.com/user-attachments/assets/ced9d284-30b8-4835-bfc0-7a59c8dc43fb)

+ Source: https://www.enjuris.com/blog/ca/men-versus-women/#:~:text=According%20to%20traffic%20collision%20data,40%20percent%20of%20car%20accidents.

In terms of ethnicity, Hispanics are most involved in vehicular collisions of all ethnic groups in LA city, followed by whites and then Blacks/Other.

![download](https://github.com/user-attachments/assets/26045441-f123-4989-966a-01a6672f31e8)

### Analysis of Accidents by Time of Day

Among all accidents, the most frequent time of day is the afternoon (from 12 PM to 4 PM) followed closely by evening (4PM to 8PM). 

![download](https://github.com/user-attachments/assets/7caca640-6217-4476-9719-3485033e9526)

However, collisions involving severe injuries are more skewed to occurring in the evening, while fatal collisions occur most often in the evening, night, and late night (altogether, from 4PM to 4AM). 
![download](https://github.com/user-attachments/assets/8e2a2056-7cf7-4149-acd2-d2454a419ecc)

![download](https://github.com/user-attachments/assets/6f5d0aee-42c3-4fb5-af9c-411833b6baaa)

Interestingly, although the afternoon ranks first as the time of day in which the most accidents occur, it ranks second to last in terms of fatal accidents. A cursory look at these bar plots suggests that total collisions is most correlated with when the most people tend to be driving (from 8am to 8pm), but that the severity of accidents seems more correlated to daylight/nighttime. 

### Folium Map of Fatal Accidents
To visualize the geographic distribution of fatal accidents, I created an interactive folium map using the newly created latitude and longitude columns in the dataframe. I only used fatal accidents because I didn't want the map to be too cluttered. 

![image](https://github.com/user-attachments/assets/c8cba9f1-ae4f-4879-8cb7-199f753dd601)

Note that the geographic distribution looks heavily concentrated in certain areas because of the odd geographic boundaries of the City of Los Angeles (which our data is limited to). The map below (source: ZeeMaps) shows the boundaries of the City of Los Angeles: 

![image](https://github.com/user-attachments/assets/c12f287a-dcdc-4235-9ff5-6f30b9fa5ebd)


## Machine Learning To Predict Severe/Fatal Accidents

Now that the exploratory analysis is complete, we can try to use machine learning to create predictive models. The ML algorithms I train in this project are as follows: 

+ Logistic Regression
+ XG Boost
+ Random Forest
+ Naive Bayes

These algorithms were selected because of their suitability to binary classification problems. In this data set we have 3274 fatal accidents out of over 600,000 recorded accidents, which would make the target variable too lopsided for effective predictions. Instead, I included accidents involving severe injuries with fatal accidents as representing positive cases (in other words, the cases we're interested in accurately predicting), while everything else (minor injuries and no injuries) were classified as negative cases for training purposes. If we can create a machine learning model that does decently well in predicting which accidents will be classified as positive cases, it could be useful to police and medical first responders in prioritizing response. 

Before creating machine learning algorithms, it's important to understand the algorithm's purpose. For practical purposes, accurately predicting accidents that result in severe injury or death should be the priority. That means that we want as few false negatives as possible. It's not too big a concern to incorrectly predict a negative case as positive, but it would be highly detrimental to predict positive cases as negative, since this may affect response times to accidents that truly do require urgency. The main goal therefore is to capture as many true positive cases as possible, while reducing the number of false positive predictions is a secondary goal. That primary goal will be important for fine-tuning the models to skew toward accurately capturing true positive cases, aiming for a minimum of 75% recall for positive cases. 

### Logistic Regression

Logistic Regression is a supervised learning classification algorithm and is commonly used for binary classification problems. It calculates probabilities for the target variable based on the input features. If the calculated probability for a test case is above 0.5, it is predicted as a positive case (=1), otherwise it is classified as a negative case (=0). Logistic Regression requires numerical variables to be scaled and categorical variables to be one-hot encoded, so I first created a preprocessor to perform both of those tasks. Then I trained the logistic regression using a pipeline and scikitlearn's various libraries. I lowered the prediction threshold from 0.5 to 0.44 to meet the objective of 75% recall rate. The model's scoring and confusion matrix for the test data is shown.

![image](https://github.com/user-attachments/assets/9beebf44-a571-4035-b099-65fbfe443d13)

Of the true positive cases in the testing data, the logistic regression classified 2915 correctly as positive, and 929 incorrectly as negative. Of the true negative cases, the model accurately classified 73751 as negative and 45870 as positive. Overall the model achieves an accuracy of 62%, with Log-Loss score of 0.5773 and ROC-AUC score of 0.7613. The ROC-AUC score is most appropriate for evaluation of a logistic regression, and it indicates that a randomly chosen positive test case would have a higher probability score than a randomly chosen negative test case 76.13% of the time. For reference, a perfect model would have an ROC-AUC score of 1 and a model that works as well as random guessing would have a score of 0.5. 

### XG Boost

The XG Boost algorithm is based on the concept of decision trees. It is an ensemble machine learning method that uses sequential decision trees to iteratively improve on each other, each tree itself learning from the errors made by its predecessor tree. XG Boosting has become a very popular machine learning algorithm across a wide variety of problems because of its broad effectiveness. 

When I initially ran the XGBoost algorithm, it didn't work very well due to the large imbalance in the training and testing data (many more 1's than 0's). To correct this, I balanced the training data to contain 50% positive cases and 50% negative cases, but testing the model was still done on the original, imbalanced test set. Additionally, I used a GridSearchCV for hyperparameter tuning, and then starting with the GridSearch values as a baseline, I played with the parameters (n_estimators, max_depth, and learning rate) myself to produce the best testing results I could. After these changes, the XGBoost model proved to be a big improvement on the logistic regression: 

![image](https://github.com/user-attachments/assets/c52e02f0-1d66-4636-a34b-9439e51a4c5b)

Since predicting true positive cases is the top priority, XGBoost is a better candidate for a final model than logistic regression, since it correctly classified 3456 positive test cases, and only incorrectly classified 388 positive cases as negative. That means the model predicts 90% of true positives correctly. The number of false positives is still substantial, but also a marked improvement over the logistic regression (38580 errors vs 45870). More true negatives are correctly classified by the XG Boost (81041). 

### Naive Bayes

The Naive Bayes algorithm is a generative learning algorithm, which means that it is optimized differently to the two discriminative models used so far. While discriminative models aim to optimize the probability of the target variable given the input features, generative models aim to maximize the probability of the union of the target variable and input features (in other words, the joint probability of x and y). In more simplistic terms, discriminative models learn decision boundaries to classify cases into positives and negatives, while generative models do so by learning probability distributions. 

The advantage of generative learning algorithms is the ability to create new data consistent with what the model was trained on (for example, image generation). However, they also require a number of assumptions to be met. In the case of Naive Bayes, assumptions like feature independence, normally distributed numerical variables, equal feature importance, and multinomial distribution of discrete features may be unrealistic for the kind of data used in this project. Nonetheless, I trained a Naive Bayes algorithm on the same data with a few alterations. First, I had to transform the data from a sparse matrix to dense matrix to make it compatible with the Naive Bayes model. After having some difficultly getting coherent results, I added a dimensionality reduction through Principle Component Analysis (PCA) to the model which improved its performance. 

![image](https://github.com/user-attachments/assets/315e9142-adb9-4645-870d-efcc8a2cb287)

As expected, the Naive Bayes does the worst job classifying cases of the three models developed so far, likely because the underlying assumptions that make the algorithm work are most out of step with the structure of the data. The Naive Bayes model still does an ok job on its own merits, with 63% accuracy and 69.32% ROC-AUC score, but it does a poor job classifying true positive cases, which is the top priority for models in this project. Compared to the XG boost which correctly classifies 90% of true positives as positives, the Naive Bayes only classifies 66% of true positives correctly when using a 50% decision threshold. 

### Random Forest

The Random Forest algorithm is similar to the XG Boost in that it is an ensemble of decision trees, but unlike XGBoost in which trees run iterately (one after the next), in a Random Forest the decision trees run in parallel, and a majority decision among all the trees in the "forest" determines the predicted value of the target variable. Each tree in a random forest is trained on a subset of the training data. The performance of the Random Forest Model (after fine tuning some paramenters) is shown below: 

![image](https://github.com/user-attachments/assets/9aa96922-0456-4864-a2d9-acb574f6eaa1)

Perhaps unsurprisingly, the Random Forest produces results that score similarly to the XGBoost (both models involving an ensemble of decision trees, just structured differently). The accuracy and ROC-AUC scores of 69% and 87.64% are similar to the XGBoost values, but the Log-Loss score of 0.5936 is an improvement over the XGBoost value of 0.6573. With a probability threshold of 0.5, the Random Forest model correctly predicts 82,148 negatives and 3431 positives, while incorrectly predicting 413 true positives as negatives and 37473 true negatives as positives. 


### Averaging Multiple Models Together

I have tried many ways of averaging the predictions of the different models to obtain something that works better than any model individually, but with only modest success. I think the main issue is that that since the XGBoost and Random Forest models work quite well, and produce predictions that are quite similar to each other, while the logistic regression and naive bayes work less well, averaging the XGB and RF doesn't yield much improvement, and including logistic regression/Naive Bayes weakens the overall predictions. One combination strategy that does seem to work quite well is actually **subtracting** the Naive Bayes probability value, and lowering the prediction threshold. *Why* this produces an improvement I will need to investigate further, but below is shown the output from one such attempt: 

![image](https://github.com/user-attachments/assets/46bb68b5-0dd0-466c-9d4d-f0d48efe1b2e)

Accuracy increases to 72%, ROC-AUC increases to 87.89%, and log-loss improves to 0.4797 in this combined model. This model is created by giving 45% weight to XGB and Random Forest probabilities, respectively, 10% weight to the logistic regression probability, and subtracting 10% of the Naive Bayes probability, and setting the threshold equal to 0.48. 

## Next Steps and Ideas for Further Improvement

There are a few other possible ways to improve the model that I would like to investigate further. These include: 
+ Try multiclass classification rather than binary classification. It might work better to predict the categories "no injury", "other injury", "severe injury", and "fatality", and then combine these predictions together to form binary categories.
+ Layer multiple machine learning models together. What I've done so far is transform one set of predicted inputs into predicted outputs. However, I could take these predicted outputs, and train a new model on those outputs to produce another set of outputs that might better predict the true target values. This process, called "stacking", to create a "Meta Model" is described here: https://medium.com/@brijesh_soni/stacking-to-improve-model-performance-a-comprehensive-guide-on-ensemble-learning-in-python-9ed53c93ce28
+ Encode missing values as their own category, rather than assigning them values of mean (for numerical variables) or mode (for categorical variables). The fact that a variable is missing from the data may have predictive power. In this case it likely means an omission from a police report by a responding officer. That a piece of information is omitted could very conceivably be related to the conditions of the collision and therefore offer predictive power in the models I've trained.

## Conclusion
The analysis of the collisions data has so far yielded useful information and offers predictive power to police and first responders. Using the best model developed in this project, first responders would be able to correctly classify 90% of accidents resulting in severe injury or death just based on information that can quickly be communicated from the scene of an accident. Nearly 100% of the time that the model predicts a non-serious accident, that is what first responders would find at the scene. 90% of the time that the model predicts a severe or fatal crash, first responders would find that the accident actually resulted in no injury or only minor injury, but this error is a small price to pay for correctly identifying serious cases. The model can be thought of as an indicator of response urgency. Accidents predicted as severe or fatal probably are not, but about 10% of the time they will be. Accidents predicted as not severe or fatal will almost certainly be correctly classified. 

The exploratory data analysis, including time series trends and time of day analysis of all accidents, severe accidents, and fatal accidents also provides useful information to policy makers. Theories attempting to explain these patterns are speculative, but are still important to establish baselines for further analysis. For example, we can see a clear correlation between darkness and accident severity, but not between darkness and accident quantity. We also see increasing time series trends of fatal and severe accidents, but a decreasing trend of overall accidents. A potential explanation for this apparent discrepancy is that better technology in cars makes accidents more rare, but that the more distractions in the car (such as ubiquity of cell phones) or overconfidence in the car's corrective technology may result in more severe accidents when they do occur. With the caveat that these explanations are only reasonable theories and not established fact, we should continue collecting and studying collision data to look for answers to prevent unnecessary damage, injury, and death. 
