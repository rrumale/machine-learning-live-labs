# Initialize Environment

## Introduction
This lab will show you how launch Jupyter Notebooks in your NoVNC instance.

*Estimated Lab Time*: 30 minutes

### Objectives
* Get familiar with the lab Instance
* Connecting to Autonomous Database instance
* Exploring dataset
*

### Prerequisites
This lab assumes you have:
* You have completed reservation process and able to login to Remote Desktop connection
* Oracle Cloud Infrastructure (OCI) account
* Autonomous Database deployed in Oracle Cloud

## Task 1: Check the Lab setup
1. The remote desktop instance has pre-configured all the software and environment needed for this workshop. This includes a 21c Databases & Jupyter Notebooks. All services would start automatically.  The lab guide is available in a browser session, this would allow you to access the lab guide and Jupyter Notebooks side by side.

    ![](./images/oml4py-novnc-guide.png " ")

## Task 2:


## Task 1: Display Insurance Customer Data in OML Notebook

* In the Autonomous Database instance details page. Click on the Service Console button.
![ADB-instance-home](images/prerequisites-screenshot-22.jpg)

* A new page with the service console is opened. In the Overview section we see the details of this specific instance. We can go to the Development section in the left side.
![ADB-service-console](images/prerequisites-screenshot-23.jpg)

* In the Development section, click on Oracle Machine Learning Notebooks.
![ADB-service-console](images/prerequisites-screenshot-24.jpg)

* Connect to OML services in Autonomous Database

Access the OML Services link and connect with the credentials that we created earlier. In our case the credentials are:

   - Username: **OMLUSER**
   - Password: **Welcome12345**


![ADB-OML-connect](images/prerequisites-screenshot-25.jpg)


* Open a Scratchpad

![Open-Scratchpad](images/automl-screenshot-1.jpg)


* The notebook server is starting. Once opened we can run a select on the **CUSTOMER_INSURANCE** table

  ````
  <copy>select * from customer_insurance;</copy>
  ````

 ![customer-insurance](images/automl-screenshot-2.jpg)

 Notice the columns ``LTV`` and ``LTV_BIN`` when you scroll to the right. These are our targets for the machine learning.

* Drop training and test tables if they exist

 ````
<copy> %script
DROP TABLE Customer_insurance_train_classification;

DROP TABLE Customer_insurance_test_classification;
</copy>
 ````
![drop-model-tables](images/automl-screenshot-3.jpg)

If the tables don't exist, the script will return an error. We will create the tables in the next steps

 * Create the training table for our Auto ML UI

````
<copy>
%script
 create table Customer_insurance_train_classification as
select CUST_ID,"LAST","FIRST","STATE","REGION","SEX","PROFESSION","BUY_INSURANCE","AGE","HAS_CHILDREN","SALARY","N_OF_DEPENDENTS","CAR_OWNERSHIP","HOUSE_OWNERSHIP","TIME_AS_CUSTOMER","MARITAL_STATUS","CREDIT_BALANCE","BANK_FUNDS","CHECKING_AMOUNT","MONEY_MONTLY_OVERDRAWN","T_AMOUNT_AUTOM_PAYMENTS","MONTHLY_CHECKS_WRITTEN","MORTGAGE_AMOUNT","N_TRANS_ATM","N_MORTGAGES","N_TRANS_TELLER","CREDIT_CARD_LIMITS","N_TRANS_KIOSK","N_TRANS_WEB_BANK","LTV_BIN"
from customer_insurance
SAMPLE (60) SEED (1)
where cust_id not in ('CU12350','CU12331', 'CU12286')
</copy>
````
 ![create-training-table](images/automl-screenshot-4.jpg)

 Notice that we skip the ``LTV`` column so it would not influence the results. We keep the ``LTV_BIN`` column to be the target for learning.
 Our goal is to learn how customers are classified in the 4 groups, that are the ``LTV_BIN`` groups. For this particular workshop we leave outside 3 specific customers so we can use them to demo the test of our models.


* Create the test table for our Auto ML UI

````
<copy>%script
 create table Customer_insurance_test_classification as
select CUST_ID,"LAST","FIRST","STATE","REGION","SEX","PROFESSION","BUY_INSURANCE","AGE","HAS_CHILDREN","SALARY","N_OF_DEPENDENTS","CAR_OWNERSHIP","HOUSE_OWNERSHIP","TIME_AS_CUSTOMER","MARITAL_STATUS","CREDIT_BALANCE","BANK_FUNDS","CHECKING_AMOUNT","MONEY_MONTLY_OVERDRAWN","T_AMOUNT_AUTOM_PAYMENTS","MONTHLY_CHECKS_WRITTEN","MORTGAGE_AMOUNT","N_TRANS_ATM","N_MORTGAGES","N_TRANS_TELLER","CREDIT_CARD_LIMITS","N_TRANS_KIOSK","N_TRANS_WEB_BANK"
from customer_insurance
minus
select CUST_ID,"LAST","FIRST","STATE","REGION","SEX","PROFESSION","BUY_INSURANCE","AGE","HAS_CHILDREN","SALARY","N_OF_DEPENDENTS","CAR_OWNERSHIP","HOUSE_OWNERSHIP","TIME_AS_CUSTOMER","MARITAL_STATUS","CREDIT_BALANCE","BANK_FUNDS","CHECKING_AMOUNT","MONEY_MONTLY_OVERDRAWN","T_AMOUNT_AUTOM_PAYMENTS","MONTHLY_CHECKS_WRITTEN","MORTGAGE_AMOUNT","N_TRANS_ATM","N_MORTGAGES","N_TRANS_TELLER","CREDIT_CARD_LIMITS","N_TRANS_KIOSK","N_TRANS_WEB_BANK" from Customer_insurance_train_classification
</copy>
 ````
 ![create-test-table](images/automl-screenshot-5.jpg)

Notice that in the testing table we will not use any of the leading ``LTV`` or ``LTV_BIN`` columns. These column might be misleading in the process. We will still use them in our verification process.



## Task 2: Use AutoML and Autonomous Database

* Go to the Main menu on the top left side near the Oracle Machine Learning icon.
![AutoML-menu](images/automl-screenshot-X06.jpg)


* Choose AutoML.
![AutoML-menu](images/automl-screenshot-6.jpg)


* Click Create in the AutoML Experiments page

![AutoML-create-experiment](images/automl-screenshot-7.jpg)

* In the Create Experiment page choose the following details:


    - Name: **AutoML Classification**
    - Data Source: chose the **CUSTOMER\_INSURANCE\_TRAIN\_classification** table in the OMLUSER schema.
    - Predict: **LTV_BIN**
    - Prediction Type: **CLASSIFICATION**
    - Case ID: **CUST_ID**


![AutoML-create-experiment](images/automl-screenshot-8.jpg)

* To make some customizations you can expand the Additional Settings menu

![AutoML-additional-settings](images/automl-screenshot-9.jpg)

Notice that we can choose the Database Service Level to High, and select by which metric should we compare the algorithms, and which predefined algorithms to include or exclude from this experiment.

 - Choose the following options for your experiment:

    - Database Service Level: **High**
    - Model Metric: **F1**
    - Weight Option: **Weighted**



![AutoML-additional-settings](images/automl-screenshot-9a.jpg)

The F1 score is the harmonic mean of the precision and recall; where the precision is the number of true positive results divided by the number of all positive results, including those not identified correctly, and the recall is the number of true positive results divided by the number of all samples that should have been identified as positive. Precision is also known as positive predictive value, and recall is also known as sensitivity in diagnostic binary classification.

**F1-score = 2 × (precision × recall)/(precision + recall)**

![Precision and Recall](images/Model_metric.jpg)

* Run the Auto ML UI for classification by clicking **```Start```** and **```Better Accuracy```**.

![Run-AutoML](images/automl-screenshot-10.jpg)

The AutoML Classification will run for several minutes showing which top 5 algorithms have a higher Balanced Accuracy. The running process takes around 10 minutes.

* And the result of the Auto ML UI

![Classification Experiment Result](images/automl-screenshot-11.jpg)

Each model described here is based on an algorithm and ran against our training data. We can click on the model name and see its details.

![Chose a model](images/automl-screenshot-12.jpg)

The model detail window opens and the first tab is Prediction Impacts.

![Model Prediction Impacts](images/automl-screenshot-13.jpg)

Notice how the other table columns impact differently our model and which column has a higher weight in it. We can click on the Confusion Matrix tab.

![Model Confusion Matrix](images/automl-screenshot-14.jpg)

There we can see for each class: **LOW**, **MEDIUM**, **HIGH**, **VERY HIGH** how may customers are actually in that class and how many are predicted to be in that class.

* We can rename the Support Vector Machine model so it would be easier to recognize in the next sections. For this we can select the model and click on the Rename button.

![Model Rename ](images/automl-screenshot-X14.jpg)

* Enter a new Model Name and click OK. In our case we are going to use **SVMG**.

![Model Rename ](images/automl-screenshot-X114.jpg)

* The model is now renamed in the results page also.

![Model Rename ](images/automl-screenshot-X1114.jpg)

You may now proceed to the next lab


## Acknowledgements
* **Author** - Rajeev Rumale, Valentin Leonard Tabacaru, Milton Wan
* **Last Updated By/Date** -  Rajeev Rumale, November 2021
