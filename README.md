Customer Churn Analysis & Risk Prediction

An end-to-end Customer Churn Analysis and Risk Prediction project combining MySQL, Python, Machine Learning, Excel, and Power BI to analyze customer behavior, identify churn patterns, predict customers at risk of churning, and visualize the results through interactive dashboards.

📌 Project Overview

The project is divided into two major stages:

Churn Analysis – Analyze historical customer data to understand churn patterns across demographics, contracts, payment methods, tenure, services, states, and churn reasons.
Churn Risk Prediction – Train a Random Forest Classification model on customers whose churn outcome is known (Stayed or Churned) and use the trained model to predict which newly joined customers are likely to churn.

The final predictions are exported to Predictions.csv and visualized in Power BI through a dedicated prediction/risk dashboard.

🛠️ Tech Stack
MySQL – Data storage, cleaning, transformation, SQL analysis, and views
Python – Data preprocessing and machine learning
Pandas / NumPy – Data manipulation
Scikit-learn – Random Forest, Label Encoding, train/test split, and evaluation
Joblib – Model serialization support
Power BI – Interactive dashboards, data modeling, KPI analysis, and churn-risk visualization
Excel – Intermediate data transfer between MySQL, Power BI, and Google Colab
Google Colab – Machine learning execution environment
🔄 End-to-End Workflow
Customer_Data.csv
       |
       v
     MySQL
       |
       |-- Data exploration
       |-- NULL / blank-value handling
       |-- Data transformation
       |-- prod_Churn table
       |-- vw_ChurnData view
       |-- vw_JoinData view
       |
       +----------------------+
       |                      |
       v                      v
   Power BI              Excel Workbook
 Churn Analysis         churn_predict.xlsx
                              |
                              v
                        Google Colab
                              |
                              |-- Preprocessing
                              |-- Label Encoding
                              |-- Train/Test Split
                              |-- Random Forest
                              |-- Model Evaluation
                              |
                              v
                     Prediction on Joiners
                              |
                              v
                       Predictions.csv
                              |
                              v
                           Power BI
                              |
                              v
                    Churn Risk Dashboard
1. 📂 Data Source

The original dataset is:

Customer_Data.csv

The CSV contains customer-level information such as:

Customer ID
Gender
Age
Marital status
State
Number of referrals
Tenure
Contract
Payment method
Monthly charge
Total charges
Total refunds
Total revenue
Internet and other service information
Customer status
Churn category
Churn reason

The CSV is imported into a MySQL table named:

customer_data
2. 🗄️ MySQL Data Processing

MySQL is used for data storage, exploration, cleaning, transformation, and creation of analytical views.

Database Initialization
CREATE DATABASE churn_db;
USE churn_db;
Exploratory Data Analysis
Gender Distribution
SELECT Gender, Count(Gender) as TotalCount,
Count(Gender) * 100.00 / 
(Select Count(*) from customer_data) as Percentage
FROM customer_data
GROUP BY Gender;

This determines the number and percentage of customers in each gender category.

Contract Distribution
SELECT Contract, Count(Contract) as TotalCount,
Count(Contract) * 100.00 / 
(Select Count(*) from customer_data) as Percentage
FROM customer_data
GROUP BY Contract;

This helps understand how customers are distributed across different contract types.

Customer Status and Revenue
SELECT Customer_Status, 
       Count(Customer_Status) as TotalCount,
       Sum(Total_Revenue) as TotalRev,
       Sum(Total_Revenue) / 
       (Select sum(Total_Revenue) from customer_data) * 100 
       as RevPercentage
FROM customer_data
GROUP BY Customer_Status;

This analyzes customer status along with its contribution to total revenue.

State Distribution
SELECT State, Count(State) as TotalCount,
Count(State) * 100.00 / 
(Select Count(*) from customer_data) as Percentage
FROM customer_data
GROUP BY State
ORDER BY Percentage DESC;
3. 🧹 Missing-Value Handling

The project first checks for NULL values across the dataset.

Blank strings and spaces are converted into SQL NULL using:

NULLIF(TRIM(column), '')

For example:

UPDATE customer_data
SET
    Gender = NULLIF(TRIM(Gender), ''),
    Married = NULLIF(TRIM(Married), ''),
    State = NULLIF(TRIM(State), ''),
    Value_Deal = NULLIF(TRIM(Value_Deal), ''),
    Phone_Service = NULLIF(TRIM(Phone_Service), ''),
    Multiple_Lines = NULLIF(TRIM(Multiple_Lines), ''),
    Internet_Service = NULLIF(TRIM(Internet_Service), ''),
    Internet_Type = NULLIF(TRIM(Internet_Type), ''),
    Online_Security = NULLIF(TRIM(Online_Security), ''),
    Online_Backup = NULLIF(TRIM(Online_Backup), ''),
    Device_Protection_Plan = NULLIF(TRIM(Device_Protection_Plan), ''),
    Premium_Support = NULLIF(TRIM(Premium_Support), ''),
    Streaming_TV = NULLIF(TRIM(Streaming_TV), ''),
    Streaming_Movies = NULLIF(TRIM(Streaming_Movies), ''),
    Streaming_Music = NULLIF(TRIM(Streaming_Music), ''),
    Unlimited_Data = NULLIF(TRIM(Unlimited_Data), ''),
    Contract = NULLIF(TRIM(Contract), ''),
    Paperless_Billing = NULLIF(TRIM(Paperless_Billing), ''),
    Payment_Method = NULLIF(TRIM(Payment_Method), ''),
    Customer_Status = NULLIF(TRIM(Customer_Status), ''),
    Churn_Category = NULLIF(TRIM(Churn_Category), ''),
    Churn_Reason = NULLIF(TRIM(Churn_Reason), '');

After converting blank values to NULL, business-friendly replacements are applied.

Examples:

Column	Missing Value Replacement
Value_Deal	None
Multiple_Lines	No
Online_Security	No
Online_Backup	No
Device_Protection_Plan	No
Premium_Support	No
Streaming_TV	No
Streaming_Movies	No
Streaming_Music	No
Unlimited_Data	No
Internet_Type	None
Churn_Category	Others
Churn_Reason	Others
4. 🗃️ Creating the prod_Churn Table

A cleaned production table called prod_Churn is created:

CREATE TABLE prod_Churn AS
SELECT 
    Customer_ID,
    Gender,
    Age,
    Married,
    State,
    Number_of_Referrals,
    Tenure_in_Months,
    IFNULL(Value_Deal, 'None') AS Value_Deal,
    Phone_Service,
    IFNULL(Multiple_Lines, 'No') AS Multiple_Lines,
    Internet_Service,
    IFNULL(Internet_Type, 'None') AS Internet_Type,
    IFNULL(Online_Security, 'No') AS Online_Security,
    IFNULL(Online_Backup, 'No') AS Online_Backup,
    IFNULL(Device_Protection_Plan, 'No') AS Device_Protection_Plan,
    IFNULL(Premium_Support, 'No') AS Premium_Support,
    IFNULL(Streaming_TV, 'No') AS Streaming_TV,
    IFNULL(Streaming_Movies, 'No') AS Streaming_Movies,
    IFNULL(Streaming_Music, 'No') AS Streaming_Music,
    IFNULL(Unlimited_Data, 'No') AS Unlimited_Data,
    Contract,
    Paperless_Billing,
    Payment_Method,
    Monthly_Charge,
    Total_Charges,
    Total_Refunds,
    Total_Extra_Data_Charges,
    Total_Long_Distance_Charges,
    Total_Revenue,
    Customer_Status,
    IFNULL(Churn_Category, 'Others') AS Churn_Category,
    IFNULL(Churn_Reason, 'Others') AS Churn_Reason
FROM customer_data;

This produces a cleaned and analysis-ready dataset.

5. 👁️ SQL Views

Two views are created to separate historical customers from new joiners.

Historical Churn Data
CREATE VIEW vw_ChurnData AS
SELECT *
FROM prod_Churn
WHERE Customer_Status IN ('Churned', 'Stayed');

vw_ChurnData contains customers whose actual outcomes are known.

These customers are used to train the machine-learning model.

New Joiner Data
CREATE VIEW vw_JoinData AS
SELECT *
FROM prod_Churn
WHERE Customer_Status = 'Joined';

vw_JoinData contains customers whose future churn outcome is not yet known.

These customers are used for prediction.

Why separate them?

The model learns from:

Stayed
Churned
   ↓
Historical Training Data

and predicts on:

Joined
   ↓
Prediction Data

This allows the model to identify potential churners among newly joined customers.

6. 📤 Data Export

The following datasets are exported:

prod_Churn.csv
vw_ChurnData.csv
vw_JoinData.csv

The churn and joiner datasets are then combined into an Excel workbook:

churn_predict.xlsx

The workbook contains two sheets:

vw_churndata
vw_joindata

The Excel file is uploaded to Google Colab for machine learning.

7. 🤖 Machine Learning
Model Used

The project uses:

Random Forest Classification

Implementation:

rf_model = RandomForestClassifier(
    n_estimators=100,
    random_state=42
)

Random Forest was selected because it can handle multiple customer attributes and provides feature-importance values that can help understand factors associated with churn.

8. ⚙️ Data Preprocessing

The following columns are removed because they are not used as model features:

Customer_ID
Churn_Category
Churn_Reason

Categorical columns are converted into numerical values using LabelEncoder.

The encoded columns include:

Gender
Married
State
Value_Deal
Phone_Service
Multiple_Lines
Internet_Service
Internet_Type
Online_Security
Online_Backup
Device_Protection_Plan
Premium_Support
Streaming_TV
Streaming_Movies
Streaming_Music
Unlimited_Data
Contract
Paperless_Billing
Payment_Method

The target variable is encoded as:

Stayed  → 0
Churned → 1
9. 🧪 Training and Testing

The historical dataset is divided into training and testing datasets:

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42
)

The model is trained:

rf_model.fit(X_train, y_train)

Predictions are generated:

y_pred = rf_model.predict(X_test)

The model is evaluated using:

Confusion Matrix
Classification Report
Feature Importance

The exact performance metrics are generated when the notebook is executed.

10. 📊 Feature Importance

Random Forest provides feature importance values:

importances = rf_model.feature_importances_

These values are sorted and visualized to determine which customer attributes contribute most to the model's predictions.

This provides additional insight into potential churn drivers.

11. 🔮 Churn Risk Prediction

After the model is trained, the same label encoders used during training are applied to the new joiner data.

The original customer information is preserved so that predictions can be associated with individual customers.

The model produces:

0 → Not predicted as churned
1 → Predicted as churned

A new column is added:

Customer_Status_Predicted

Only predicted churners are retained:

original_data = original_data[
    original_data['Customer_Status_Predicted'] == 1
]

The final output is:

Predictions.csv

This file contains customers identified by the model as potential churn risks.

Important: A predicted churner is not guaranteed to churn. It represents a customer identified by the model as having an elevated predicted risk.

12. 📊 Power BI Dashboard

Power BI is used to transform the processed data and machine-learning predictions into interactive business dashboards.

The project contains two major dashboard pages.

A. Churn Analysis — Summary

The Summary dashboard provides an overview of customer churn.

Key KPIs
Total Customers
New Joiners
Total Churn
Churn Rate
Analysis Areas

The dashboard analyzes churn by:

Gender
Age Group
State
Internet Type
Payment Method
Contract
Tenure Group
Churn Category
Individual Services
Filters

The dashboard includes interactive filters such as:

Monthly Charge Range
Married

The dashboard helps answer:

What is the overall churn rate?
How many customers have churned?
Which age groups have higher churn?
Which states have higher churn rates?
Which contracts are associated with higher churn?
Which payment methods have higher churn?
Which services are associated with churn?
What are the major churn categories?
B. Churn Analysis — Prediction

The Prediction dashboard focuses on customers identified by the Random Forest model as potential churners.

It provides:

Total predicted churners
Predicted churners by gender
Predicted churners by age group
Predicted churners by marital status
Predicted churners by tenure group
Predicted churners by payment method
Predicted churners by contract
Predicted churners by state
Customer-level predicted-risk table

The customer-level table contains information such as:

Customer_ID
Monthly_Charge
Total_Revenue
Total_Refunds
Number_of_Referrals

This allows the analysis to move from high-level churn trends to individual customers who may require retention efforts.

13. 🔗 Power BI Data Model

The Power BI data model contains the following major tables:

prod_Churn
prod_services
Churn_Predictions
mapping_age
mapping_tenuregrp
z_dummy
prod_Churn

Contains the cleaned customer-level data.

prod_services

Contains service-related customer information.

mapping_age

Maps individual ages into groups:

<20
20-35
35-50
>50
mapping_tenuregrp

Maps tenure into groups:

<6 Months
6-12 Months
12-18 Months
18-24 Months
>=24 Months
Churn_Predictions

Contains machine-learning predictions and customer information used in the Prediction dashboard.

z_dummy

Supporting table used in the Power BI data model.

14. 💼 Business Value

This project combines descriptive, diagnostic, and predictive analytics.

Descriptive Analytics

Power BI answers:

What happened?

For example, it shows churn rates by age, state, contract, payment method, and services.

Diagnostic Analytics

Customer attributes and churn categories help investigate:

Why might customers be leaving?

Predictive Analytics

The Random Forest model answers:

Which customers are at risk of churning?

Business Action

The predicted churn list can be used to:

Prioritize high-risk customers
Develop targeted retention campaigns
Review contract-related churn
Investigate service dissatisfaction
Focus retention efforts on high-risk segments
Monitor customer behavior
15. 📁 Recommended GitHub Structure
# 15. 📁 GitHub Repository Structure

The project is organized in the GitHub repository as follows:

```text
Customer-Churn-Analysis-and-Prediction-Dashboard/
│
├── Images/
│   └── Dashboard and Power BI screenshots
│
├── Raw Data/
│   └── Customer churn datasets and processed data files
│
├── DataCleaning.sql
│   └── MySQL database creation, data cleaning,
│       transformation, analysis, and view creation
│
├── RandomForestClassifier.py
│   └── Python preprocessing, Random Forest model training,
│       evaluation, feature importance, and churn prediction
│
└── README.md
    └── Project documentation and workflow
```

### Repository Components

**`Images/`**
Contains the screenshots and visual assets used to demonstrate the Power BI dashboards and project results.

**`Raw Data/`**
Contains the source and processed datasets used throughout the project, including the customer data and prediction-related files.

**`DataCleaning.sql`**
Contains the complete MySQL workflow, including database initialization, data exploration, missing-value handling, creation of the cleaned `prod_Churn` table, and creation of the `vw_ChurnData` and `vw_JoinData` views.

**`RandomForestClassifier.py`**
Contains the Python machine-learning workflow, including preprocessing, categorical encoding, train/test splitting, Random Forest training, model evaluation, feature-importance analysis, and prediction of churn risk for new joiners.

**`README.md`**
Provides an overview of the project, methodology, technologies used, workflow, dashboards, and instructions for reproducing the analysis.

> The repository structure shown above reflects the files and folders currently present in the GitHub repository.

16. 🚀 How to Run the Project
Step 1 — Import the CSV

Import:

Customer_Data.csv

into the MySQL table:

customer_data
Step 2 — Run the SQL Script

Run the SQL script to:

Create the database
Explore the data
Check missing values
Convert blank strings to NULL
Replace missing categorical values
Create prod_Churn
Create vw_ChurnData
Create vw_JoinData
Step 3 — Export Data

Export:

prod_Churn.csv
vw_ChurnData.csv
vw_JoinData.csv

Prepare:

churn_predict.xlsx

with:

vw_churndata
vw_joindata
Step 4 — Run Machine Learning

Open the notebook in Google Colab.

Update:

file_path = r"<file path>/churn_predict.xlsx"

Run the notebook to:

Load historical churn data
Preprocess the data
Encode categorical variables
Train Random Forest
Evaluate the model
Generate feature importance
Load new joiners
Apply the trained encoders
Predict churn risk
Generate Predictions.csv
Step 5 — Power BI

Import the processed datasets and:

Predictions.csv

into Power BI.

Create the required relationships and mapping tables.

The resulting Power BI report provides:

Historical Churn Analysis
          +
Predictive Churn Risk
          =
Customer Retention Insights
17. ⭐ Key Project Highlights
Built an end-to-end Customer Churn Analysis and Prediction pipeline.
Used MySQL for data cleaning, transformation, exploration, and SQL views.
Converted blank text values into SQL NULL.
Replaced missing categorical values with meaningful business categories such as None, No, and Others.
Created separate SQL views for historical customers and new joiners.
Used Random Forest Classification for churn prediction.
Applied consistent categorical encoding between training and prediction datasets.
Evaluated the model using a confusion matrix and classification report.
Used Random Forest feature importance for model interpretation.
Generated a customer-level Predictions.csv.
Built interactive Power BI dashboards for both historical churn analysis and predictive risk analysis.
Connected SQL + Machine Learning + Business Intelligence into one end-to-end project.
18. 🖼️ Dashboard Screenshots


## Churn Analysis - Summary


![Churn Analysis Summary](Images/Summary(Dashboard).png)


## Churn Analysis - Prediction


![Churn Prediction Dashboard](screenshots/Prediction_Dashboard.png)


## Power BI Data Model


![Power BI Data Model](screenshots/PowerBI_Data_Model.png)
19. ⚠️ Important Notes
The model is trained only on customers with known outcomes: Stayed and Churned.
Joined customers are treated as prediction candidates.
The future churn outcome of a joined customer is not used during prediction.
The same label encoders fitted on the training data are used for prediction data.
The exact model performance depends on the dataset and notebook execution.
Predictions.csv contains model predictions, not confirmed future churn outcomes.
A predicted churner should therefore be interpreted as a customer at elevated predicted risk, not a guaranteed churner.
20. 🏁 Conclusion

This project demonstrates a complete Data Analytics + Machine Learning + Business Intelligence workflow:

Raw Customer Data
       ↓
MySQL Data Cleaning
       ↓
SQL Analysis & Views
       ↓
Python Preprocessing
       ↓
Random Forest Classification
       ↓
Churn Risk Prediction
       ↓
Predictions.csv
       ↓
Power BI
       ↓
Actionable Customer Retention Insights

The combination of SQL, Python machine learning, and Power BI provides both a historical understanding of customer churn and a predictive layer for identifying customers who may require proactive retention efforts.
