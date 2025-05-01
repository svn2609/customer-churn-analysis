![alt text](nyu-logo.png)
# Reducing Customer Churn in Retail with Personalized Incentives

### Big Data (CSGY-6513-C) | Fall 2024
##### Team Members: 
* Abhishek Agrawal (aa9360)
* Shubham Naik (svn9724)
* Vaibhav Rouduri (vr2470)

<br>

# Index
1. [Exploratory Data Analysis](#exploratory-data-analysis)
2. [Feature Engineering](#feature-engineering)
3. [Machine Learning Models](#machine-learning-models)
4. [Churn Prediction or Probable Churns](#churn-prediction-or-probable-churns)
5. [Personalized Offers](#personalized-offers)
6. [Technical Overview](#technical-overview)
7. [Conclusion](#conclusion)

<br>

# Exploratory Data Analysis:
The dataset consists of four tables in separate .csv files, covering a two-year span of purchase transactions for 2500 households. Additionally, demographic information of households, product details, and coupon codes per product based on business policies are available.

Dataset:
* Transactions (transaction_data.csv) - Contains purchase transactions of households.
* Demographics (hh_demographic.csv) - Contains demographic information of households.
* Products (product.csv) - Contains product details.
* Coupon (coupon.csv) - Contains coupon information.

<br>

# Feature Engineering:
To understand customer behavior and predict churn, we engineered several features from the transactional data spanning over two years (approximately 2.5 million records).

The features incorporated into the model include:
* Frequency: The number of transactions within a specified timeframe (e.g., past 30/60/90 days).
* Monetary Value: The total expenditure within a specified timeframe.
* Average Basket Size: The average number of items per transaction.
* Discount Utilization: The average discount applied per transaction.
* Discount Count: The number of discounts used per transaction.

Churn Definition: Churn probability is determined based on the recency of the last purchase. A customer is considered `churned` if their churn probability exceeds `0.5`.

<br>

# Machine Learning Models: 

Experiments were conducted with three different machine learning models to predict customer churn using the specified features.

1. **Logistic Regression**: 
   - A statistical model that employs a logistic function to model a binary dependent variable. 
   - Recognized for its simplicity and effectiveness in binary classification problems such as churn prediction.
   - Served as a baseline model for performance comparison with more complex models.

2. **MLPClassifier (Multi-layer Perceptron Classifier)**:
   - An artificial neural network comprising multiple layers of nodes.
   - Capable of capturing complex patterns in data by learning non-linear relationships.
   - A neural network with one hidden layer was utilized, experimenting with various numbers of neurons to optimize performance.

3. **Random Forest Classifier**:
   - An ensemble learning method that constructs multiple decision trees during training and outputs the mode of the classes for classification.
   - Known for its robustness to overfitting and ability to handle large datasets with higher dimensionality.
   - The number of trees and the maximum depth of the trees were tuned to achieve optimal results.

The Random Forest Classifier emerged as the top-performing model, configured with **200 estimators** and a **random state of 42**. It was evaluated using **6-fold cross-validation**, achieving an impressive accuracy of **96.26%**. 
- The feature importance weights were determined as follows:
    - **Frequency**: 0.243
    - **Monetary Value**: 0.204
    - **Average Basket Size**: 0.196
    - **Discount Count**: 0.182
    - **Discount Utilization**: 0.175
    
- **Reference**: Detailed analysis available in [churn-model-generator.ipynb](churn-model-generator.ipynb).

![alt text](feature-weights.png)
<br>

# Churn Prediction or Probable Churns: 
Using the Random Forest Classifier trained model and a 52-week window of transactional data, the churn probability for each household was predicted. The following steps were taken to create a new dataset of probable churns:

- **Latest 52 Weeks Transaction Trends**:
  - The latest week number was determined from the transaction data.
  - Transaction trends for the latest 52 weeks were aggregated by household key, basket ID, store ID, day, and week number.
  - Aggregated metrics included total quantity, sales value, retail discount, coupon discount, and coupon match discount.

- **Dataset Creation**:
  - The predicted churn probability was added to the transactional data using the trained model i.e. `random_forest_model.pkl` hosted on AWS SageMaker.
  - A flag for personalized offer distribution (is_churn_coupon) was added to track households that responded to churn probability interventions.

- **Reference**: Detailed analysis available in [probable-churn.ipynb](probable-churn.ipynb).

This comprehensive approach aimed to retain customers by enhancing satisfaction and fostering deeper connections.

<br>

# Personalized Offers:

- **Objective**: Minimize customer churn by crafting personalized coupon offers tailored to unique shopping habits.
  
- **Identification of Target Households**:
  - Focus on households with a recency of six weeks or less.
  - Exclude those already engaged with churn-related incentives.

- **Data Analysis**:
  - Analyze transaction data from the past 31 weeks.
  - Identify purchasing trends and interactions with products and coupons.

- **Personalized Offer Creation**:
  - Curate offers featuring top products aligned with household preferences.
  - Refine offers using demographic information for enhanced relevance and appeal.

- **Strategic Delivery**:
  - Execute delivery through multiple channels to maximize customer engagement.
  - Monitor engagement and redemption rates to refine strategies.

- **Outcome**:
  - Reduce churn and foster deeper customer connections.
  - Enhance overall customer experience and satisfaction.

- **Reference**: Detailed strategy available in [personalized-offers.ipynb](personalized-offers.ipynb).

<br>

# Technical Overview:
* All pipelines are crafted and deployed using `AWS SageMaker`.
- **Model Training**:
  - The `churn-model-generator.ipynb` notebook was utilized to train the Random Forest Classifier.
  - The resulting model was stored as a `.pkl` file.

- **Churn Prediction**:
  - The `probable-churn.ipynb` notebook facilitated the creation of a pipeline to estimate churn probabilities for each household using the trained model.
  - This dataset of potential churns was then uploaded to an S3 bucket for access.

- **Tailored Offers**:
  - The `personalized-offers.ipynb` notebook was employed to design personalized offers, targeting specific households based on their preferences.
  - These offers were uploaded to an S3 bucket and can be distributed via impressions or emails according to business needs.

- All code is also mirrored and maintained in a Git repository [here](https://github.com/abhishekagrawal9360/churn-predictor).

<br>

Pipeline Execution Steps:
1. Upload the latest transaction, demographic, coupon, and product datasets to their respective S3 locations:
   - Transactions: `s3://avs-churn-predictor/raw/transactions`
   - Demographics: `s3://avs-churn-predictor/raw/demographics`
   - Coupons: `s3://avs-churn-predictor/raw/coupons`
   - Products: `s3://avs-churn-predictor/raw/products`
2. Run the `churn-model-generator` SageMaker notebook to train the Random Forest Classifier (**if not already trained**).
3. Run the `probable-churn` SageMaker notebook to generate the probable churn dataset. 
   - The dataset will be saved at `s3://avs-churn-predictor/probable-churn`.
4. Run the `personalized-offers` SageMaker notebook to generate the personalized offers dataset.
   - The dataset will be saved at `s3://avs-churn-predictor/personalized-offers`.

Note: The above steps are manual and can be easily automated using AWS resources, but for the sake of project and cost considerations, the steps are kept manual.

<br>

# Conclusion:
This project effectively established a robust system for predicting customer churn and distributing personalized offers, utilizing cutting-edge machine learning techniques and thorough data analysis. By pinpointing customers at high risk of leaving and providing them with tailored incentives, the system aimed to decrease churn rates and boost customer loyalty. This implementation highlighted the power of data-driven strategies in promoting business growth and enhancing customer retention.

Key Insights from the Dataset:
- Out of 2500 households, only 800 have registered demographic information.
- Over a 52-week period, the predicted and actual churns account for more than 15.3% of the total households.
- Current offers and coupons are linked to a limited range of products.

Based on the analysis, approximately 3.4% of the households identified as probable or already churned have received offers. The following recommendations are proposed:
- Enhance the dataset with additional demographic details for households.
- Expand the range of products associated with offers to increase customer engagement.
- Distribute offers through various channels to optimize customer engagement.

<br>

<img src="probable-churn.png" alt="Probable Churn" width="42%" height="350px" style="display: inline-block;">
<img src="offers-distribution.png" alt="Offers Distribution" width="48%" height="350px" style="display: inline-block;">
