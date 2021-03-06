# Water Pump Classification

## A. Overview 
\
**Use Case** \
Classify the water pumps at various locations in Tanzania as one of 3 classes:\
\- Functional\
\- Non-functional\
\- Functional but needs repair\
\
**Objective** \
The main objective is to learn how to tackle a Machine Learning Classification problem with a large number of categorical features. In addition, I'm looking to improve my modeling skills by using sklearn pipelines with custom transformers, as well as using Optuna for hyperparameter tuning. I have also implemented a novel technique for detecting class-based outliers.\
\
This project is based on a practice competition hosted by *DrivenData*, a crowdsourcing platform that helps tackle social issues with the help of the Data Science community. Part of this project was implemented in a team in the Machine Learning course I had taken at Queen's University. Following is the link to the competition: [Click here](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/submissions/)\
\
**Business Problem** \
About 25 million people in Tanzania do not have access to clean drinking water. This has led to a widespread increase in waterborne diseases all across the country and a serious health crisis. As the government struggles to find solutions, our project aims to use predictive analytics to identify the Functional, Non-Functional and Functional-but-needs-repair waterpoints to help them optimally allocate water resources and maintain water pumps.\
\
**Contributions** \
Besides part of the data exploration, all the code displayed in this repository has been developed only by me. The original team project involved additional implementation:\
\- Ting Lan: Data Exploration\
\- Kevin Wang, Ajay Chaudhary: Model Experimentation\
The additional modeling code has not been included in the repository, but the results are displayed in this report.

<br/>

## B. Data
\
The data is provided by DrivenData in 3 files: Train_X.csv, Train_Y.csv, Test_X.csv.\
\
\- Training data sample size: 59,400\
\- Classes: 3\
\- Total features: 39 (geography, construction type, water source and quality, etc.)\
\- Categorical features: 28\
\
**Key Observations**\
\
\- 9 features are redundant (similar to other features)\
\- 7 have missing values\
\- Class are imbalanced\
\
<img src="images/pump_class.png?raw=true"/>
\
\
Further examination of the class-conditional distribution of features helps us identify the important ones.\
\
\- The proportion of non-functional pumps decreases almost linearly over time based on the construction year\
\
<img src="images/const_year1.PNG?raw=true"/>

\
\- Overall, the 3 classes seem to be similarly distributed across the country. However, the *functional but needs repair* pumps seem to be located slightly more North-West based on the median latitude and longitude shown in the right-most graph.\
\
<img src="images/lat_long1.JPG?raw=true"/>

\
\- If we observe specific regions, the distribution varies significantly. In Mtwara and Lindi non-functional waterpoints account for more than 60% of the total whereas in Iringa and Arusha they are less than 30%.\
\
<img src="images/region1.PNG?raw=true"/>

\
\- The proportion of non-functional pumps decreases with increase in height\
\
<img src="images/gps_height1.PNG?raw=true"/>

\
\- Waterpoints with a source in lakes or dams are far more likely to be non-functional than those getting water from rivers and springs.\
\
<img src="images/water_source1.PNG?raw=true"/>

<br/>

## C. Implementation
\
**Data Cleaning**\
\- Drop redundant columns that are similar to other features\
\- Change 0 construction year to NaN as missing values are handled internally by boosting trees\
\- Standardize spellings in the installer feature so that there are no redundant categories\
\
**Feature Engineering**\
\- Create *month_recorded* as a feature from date_recorded\
\- Extract days since start of time from date_recorded\
\- Due to the high cardinality of several categorical features, we group the ones that have a low frequency and uninteresting class distribution into a single *Rare* category. These were identified using the PowerBI dashboard, and the grouping is done in code.\
\- The categorical features are then encoded using both One-Hot-Encoding and LightGBM's default encoder.\
\
**Class-based Outlier Detection**\
\
Boosting trees are mostly robust to outliers. However, class-based outliers can cause them to overfit. It refers to data points whose large proportion of neighbors in the feature space belong to a different class.
\
<img src="images/class_outlier1.png?raw=true"/>

I have implemented a function that first calls the sklearn.neighbors package to determine the k nearest neighbors for each data point. I then calculate what proportion of neighbors have a different class label. If this is greater than a certain threshold (in our case 85%), the given data point is flagged as an outlier and removed from the training data. This resulted in a significant increase in test accuracy.\
\
**Modeling and Evaluation**\
\
Due to the general success of tree-based models on structured datasets with categorical features, we experimented exclusively with Random Forest, LightGBM and CatBoost. LightGBM and CatBoost produced the best results, and were particularly useful because of their inbuilt capability of handling missing values and categorical features. The code in the repository only contains the LightGBM implementation, as the other two models were implemented by my teammates as mentioned in Contributions.\
\
The models results were evaluated by DrivenData using classification accuracy. We also used 10-fold cross-validation to tune and evaluate the models before submission. Tuning was done with the help of Optuna's MedianPruner. A further Grid Search was also carried out based on the best results from Optuna.

<br/>

## D. Results & Business Impact
\
**Results**

|     Model     | CV Accuracy | Test Accuracy |
| ------------- | ----------- | ------------- |
| Random Forest |   0.7942    |    0.7875     |
|   CatBoost    |   0.8195    |    0.8146     |
|   LightGBM    |   0.8213    |    0.8170     |

LightGBM demonstrated best performance on test data based on submission results on DrivenData. We further examine its Classification Report and Confusion Matrix.

<br/>

**Classification Report**\
\
<img src="images/accuracy_report1.PNG?raw=true"/>

**Confusion Matrix**\
\
<img src="images/CM.png?raw=true"/>

While the model performs well overall, it struggles with the minority Functional-Needs-Repair class. This is mainly because of less data available for the class. While under-sampling and over-sampling techniques could help improve performance, it was not prioritized as our only objective for the competition is the overall accuracy and not the macro average.\
\
The Recall of the Non-Functional class is also not so high. This indicates high False Negatives for the class and from the confusion matrix, we can further validate that a significant proportion of Non-Functional pumps are being predicted as Functional.\
\
On the other hand, the Functional class has very high Recall. It has fewer False Negatives than False Positives, which means that very few Functional pumps are being predicted as Non-Functional.\
\
If we were to prioritize the business objective over the competition scores, we would focus on improving the Non-Functional Recall and Functional Precision. This is because predicting Non-Functional pumps as Functional is very costly in terms of health hazards. This is explained in the Business Impact section.\
\
**Business Impact**\
\
Although there is no scope for our model to be actually used by the Tanzanian Government, we have analyzed its business impact for our own learning.\
\
As of today, the government does not have enough resources to inspect every waterpoint in the country. With the help of our predictions, the Functional waterpoints can be identified and used to improve water allocation across communities. The Non-Functional waterpoints can be removed so that people do not have to consume unclean and harmful water. For the Functional-Needs-Repair waterpoints which constitute just 7% of the total, if correctly identified, maintenance and repair can easily be arranged.\
\
Improving water access with our predictions could have a massive impact financially and on people's health. It is esimtated that 43% of Tanzanians do not have access to safe drinking water. 23,900 children under the age of 5 also die every year due to water-borne diseases. 70% of the Tanzania Govt health budget (~700 million USD) is spent on diseases linked to lack of clean water and sanitation.\
\
Based on these figures and a few assumptions, we estimate that each Non-Functional or FNR waterpoint predicted as Functional would cost 6200 USD in terms of healthcare and medical infrastructure costs, as well as 0.7 lives of infants. Each Functional waterpoint predicted as NF or FNR would result in additional inspection and labor cost of 200 USD. These estimates help us map the costs to our confusion matrix.\
\
<img src="images/financials1.PNG?raw=true"/>
