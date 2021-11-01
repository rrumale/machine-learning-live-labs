# Using REST Services with Oracle Machine Learning

## Introduction

In this section we will leverage the Machine Learning models in Autonomous Database using REST APIs. In a situation where the bulk of the data is in Cloud database, we can easily create multiple models and compare them. In addition to that, using REST APIs we can use the most suitable model and score specific data for on the spot prediction or classification.

In this workshop, you have a dataset representing 15k customers of an insurance company. Each customer has around 30 attributes, and our goal is to train our database to predict customers life-time value (LTV), or to classify them in predefined classes based on this predicted value.

In marketing, [life-time value (LTV)](https://en.wikipedia.org/wiki/Customer_lifetime_value) is a prognostication of the net profit contributed to the whole future relationship with a customer.

Estimated Lab Time: 1 hour

### Objectives
*	Load data into Autonomous Database
*	Prepare the customer insurance date
*	Use AutoML to create a Machine Learning model.
*	Use Postman to score the machine learning model using REST APIs  
*	Use Curl to score the machine-learning model using REST APIs in command line.

### Prerequisites
* Oracle Cloud Infrastructure (OCI) account
* Autonomous Database deployed in Oracle Cloud
* Postman or Curl available on your working station



## Task 1: Display Insurance Customer Data in OML Notebook

* In the Autonomous Database instance details page. Click on the Service Console button.
![ADB-instance-home](images/prerequisites-screenshot-22.jpg)

* A new page with the service console is opened. In the Overview section we see the details of this specific instance. We can go to the Development section in the left side.
![ADB-service-console](images/prerequisites-screenshot-23.jpg)

* In the Development section, click on Oracle Machine Learning Notebooks.
![ADB-service-console](images/prerequisites-screenshot-24.jpg)

* Connect to OML services in Autonomous Database

Access the OML Services link and connect with the credentials that we created earlier. In our case the creadentials are:

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
DROP TABLE Customer_insurance_train_clasification;

DROP TABLE Customer_insurance_test_clasification;
</copy>
 ````
![drop-model-tables](images/automl-screenshot-3.jpg)

If the tables don't exist, the script will return an error. We will create the tables in the next steps

 * Create the training table for our Auto ML

````
<copy>
%script
 create table Customer_insurance_train_clasification as
select CUST_ID,"LAST","FIRST","STATE","REGION","SEX","PROFESSION","BUY_INSURANCE","AGE","HAS_CHILDREN","SALARY","N_OF_DEPENDENTS","CAR_OWNERSHIP","HOUSE_OWNERSHIP","TIME_AS_CUSTOMER","MARITAL_STATUS","CREDIT_BALANCE","BANK_FUNDS","CHECKING_AMOUNT","MONEY_MONTLY_OVERDRAWN","T_AMOUNT_AUTOM_PAYMENTS","MONTHLY_CHECKS_WRITTEN","MORTGAGE_AMOUNT","N_TRANS_ATM","N_MORTGAGES","N_TRANS_TELLER","CREDIT_CARD_LIMITS","N_TRANS_KIOSK","N_TRANS_WEB_BANK","LTV_BIN"
from customer_insurance
SAMPLE (60) SEED (1)
where cust_id not in ('CU12350','CU12331', 'CU12286')
</copy>
````
 ![create-training-table](images/automl-screenshot-4.jpg)

 Notice that we skip the ``LTV`` column so it would not influence the results. We keep the ``LTV_BIN`` column to be the target for learning.
 Our goal is to learn how customers are classified in the 4 groups, that are the ``LTV_BIN`` groups. For this particular workshop we leave outside 3 specific customers so we can use them to demo the test of our models.


* Create the test table for our Auto ML

````
<copy>%script
 create table Customer_insurance_test_clasification as
select CUST_ID,"LAST","FIRST","STATE","REGION","SEX","PROFESSION","BUY_INSURANCE","AGE","HAS_CHILDREN","SALARY","N_OF_DEPENDENTS","CAR_OWNERSHIP","HOUSE_OWNERSHIP","TIME_AS_CUSTOMER","MARITAL_STATUS","CREDIT_BALANCE","BANK_FUNDS","CHECKING_AMOUNT","MONEY_MONTLY_OVERDRAWN","T_AMOUNT_AUTOM_PAYMENTS","MONTHLY_CHECKS_WRITTEN","MORTGAGE_AMOUNT","N_TRANS_ATM","N_MORTGAGES","N_TRANS_TELLER","CREDIT_CARD_LIMITS","N_TRANS_KIOSK","N_TRANS_WEB_BANK"
from customer_insurance
minus
select CUST_ID,"LAST","FIRST","STATE","REGION","SEX","PROFESSION","BUY_INSURANCE","AGE","HAS_CHILDREN","SALARY","N_OF_DEPENDENTS","CAR_OWNERSHIP","HOUSE_OWNERSHIP","TIME_AS_CUSTOMER","MARITAL_STATUS","CREDIT_BALANCE","BANK_FUNDS","CHECKING_AMOUNT","MONEY_MONTLY_OVERDRAWN","T_AMOUNT_AUTOM_PAYMENTS","MONTHLY_CHECKS_WRITTEN","MORTGAGE_AMOUNT","N_TRANS_ATM","N_MORTGAGES","N_TRANS_TELLER","CREDIT_CARD_LIMITS","N_TRANS_KIOSK","N_TRANS_WEB_BANK" from Customer_insurance_train_clasification
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
    - Data Source: chose the **CUSTOMER\_INSURANCE\_TRAIN\_CLASIFICATION** table in the OMLUSER schema.
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

* Run the Auto ML for classification by clicking **```Start```** and **```Better Accuracy```**.

![Run-AutoML](images/automl-screenshot-10.jpg)

The AutoML Classification will run for several minutes showing which top 5 algorithms have a higher Balanced Accuracy. The running process takes around 10 minutes.

* And the result of the Auto ML

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


## Task 3: Deploy the model for REST access using OML Services

The next steps would be to take a model and deploy it for REST access.

* Go to the Main menu on the top left side near the Oracle Machine Learning icon.
![AutoML-menu](images/automl-screenshot-X06.jpg)

* Choose Models.

![Models Menu](images/automl-screenshot-15.jpg)

* We see the list of models created by the AutoML with their specific algorithm and target value.

![Models List](images/automl-screenshot-16.jpg)

* Click on the the  **SVMG** model based on the **Support Vector Machines Gaussian** algorithm and click Deploy. It is a powerful classification algorithm with very high F1 score

![Deploy Model](images/automl-screenshot-X16.jpg)


Enter the following details.


- Name: is prefilled with the Model name.
- URI: choose a specific URI, for example: **classsvmg**
- Version: **1**


Copy the Model URI in a accessible place because we are going to use it in the next sections of the workshop.

Click OK.

![Deploy Model](images/automl-screenshot-17.jpg)

The model will be deployed and a green banner will show the success of the deployment.

![Deploy Success](images/automl-screenshot-18.jpg)

* In the Deployment tab you can see the model and URI

![Models Deploy Details](images/automl-screenshot-19.jpg)

We can now use REST APIs to query the model, model scoring and scoring for specific data.

## Task 4:   Score the model using Postman

## Task 4.1:   Prepare REST calls


* Connect to the UI of your VM instance accessing the noVNC link. The URL to connect is on the home LiveLabs page.

````
<copy>http://<Instance IP\>:6080/vnc.html?password=MLlearnPTS#21_&resize=scale&quality=9&autoconnect=true</copy>
````


* Open a terminal and launch the Postman Application by running

````
<copy>./OML-Services/Postman/Postman</copy>
````

![Launch Postman](images/automl-screenshot-20.jpg)

Postman registration is not necessary for this workshop therefore you can choose Skip on the login page.

![Launch Postman](images/automl-screenshot-201.jpg)

You can close the scratchpad banner and hide the sidebar for a cleaner view.

![Launch Postman](images/automl-screenshot-21.jpg)


To access Oracle Machine Learning Services using the REST API, you must provide an access token. To authenticate and obtain an access token, use following POST command and the header to pass the user name and password for your Oracle Machine Learning Services account against the Oracle Machine Learning User Management Cloud Service REST endpoint /oauth2/v1/token.

The access token has a life span of one hour and can be refreshed before it expires.
- A token can be refreshed up to 8 hours.
- A token can be revoked, and a revoked token cannot be refreshed.
- Each token can be used many times.
- The token is tied to the user who authenticates using the database credential.

#### URL Structure

Use the following URL structure to access the REST endpoints:

````
https://adb.<region-prefix>.oraclecloud.com/<resource-path>
````

Where:
 - <region-prefix\>: The URL prefix based on region.
 - <resource-path\>: Relative path that defines the resource.

For example:
````
https://adb.us-ashburn-1.oraclecloud.com/omlusers
````

In this example:
 - adb.us-ashburn-1 is the URL prefix based on region. The region here is US.
 - omlusers is the resource path.


#### Generate Access Token

To generate the access token we will need the following details:

````
Operation: POST

URI endpoint:
<copy>https://adb.<region-prefix>.oraclecloud.com/omlusers/tenants/<tenancy-ocid>/databases/<dbname>/api/oauth2/v1/token </copy>

````

 - Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.
 - Replace **`<tenancy-ocid>`** with your the tenancy OCID as described in the Preparation task.
 - Replace **`<dbname>`** with your Autonomous Transaction Processing database name. You can review the Preparation Task for the Autonomous database name.



In the header Tab enter the details:

````
--header 'Content-Type: application/json'
--header 'Accept: application/json'
````

![Postman token headers](images/automl-screenshot-22.jpg)

In the Body tab, pick RAW format and enter the following:

````
<copy>
{
   "grant_type":"password",
   "username":"OMLUSER",
   "password":"Welcome12345"
}
</copy>
````
![Postman token body](images/automl-screenshot-23.jpg)

Click Send

The response in JSON format and it contains the access token:

![Postman token](images/automl-screenshot-24.jpg)

Choose the display format in RAW and copy the token starting from ``:"``  up until ``==``. In the above example the token is:

>{"accessToken":"**eyJhbGci....6zIw==**","expiresIn":3600,"tokenType":"Bearer"}

![Postman token copy](images/automl-screenshot-25.jpg)


## Task 4.2:  Use REST calls to predict customer classification

in this Task we can test our prediction for 3 distinct customers from the CUSTOMER\_INSURANCE\_TEST\_CLASIFICATION table:
 - ``CUST_ID = CU12350`` , ``LAST = FRAN``, ``FIRST = HOBBS``
 - ``CUST_ID = CU12331`` , ``LAST = AL`` , ``FIRST = FRANK``
 - ``CUST_ID = CU12286`` , ``LAST = ELLIOT`` , ``FIRST = PADGETT``


Open a new POSTMAN tab.

![New Postman Tab](images/automl-screenshot-26.jpg)

Enter the following details:

````
Operation: POST

URI endpoint:
<copy>https://adb.<region-prefix>.oraclecloud.com/omlmod/v1/deployment/<model_URI>/score </copy>

````
 - Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.

 - Replace **`<model_URI>`** with the model URI that we defined in Task 3: **``classsvmg``**

![Postman Model Score](images/automl-screenshot-27.jpg)

In the header Tab enter the details:

````
--header 'Content-Type: application/json'

````

![Postman Model Score Header](images/automl-screenshot-28.jpg)

In the Authorization tab pick **Bearer Token** and paste the token copied above in the field.

![Postman Model Score Access](images/automl-screenshot-29.jpg)

In the Body tab enter the Fran Hobs data in RAW format. Notice that we are not providing any of the ``LTV`` or ``LTV_BIN`` data, but provide fully the other important data

````
<copy>
{
   "inputRecords":[
      {
         "CUST_ID":"CU12350",
         "LAST":"FRAN",
         "FIRST":"HOBBS",
         "STATE":"NV",
         "REGION":"Southwest",
         "SEX":"F",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"No",
         "AGE":21,
         "HAS_CHILDREN":0,
         "SALARY":66180,
         "N_OF_DEPENDENTS":5,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":0,
         "TIME_AS_CUSTOMER":5,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":500,
         "CHECKING_AMOUNT":185,
         "MONEY_MONTLY_OVERDRAWN":53.21,
         "T_AMOUNT_AUTOM_PAYMENTS":197,
         "MONTHLY_CHECKS_WRITTEN":4,
         "MORTGAGE_AMOUNT":0,
         "N_TRANS_ATM":3,
         "N_MORTGAGES":0,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":2500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":0
      }
   ]
}
</copy>
````

![Postman Model Score Fran Hobs](images/automl-screenshot-30.jpg)

Click Send.

![Postman Model Score Fran Hobs Result](images/automl-screenshot-31.jpg)

Notice the result in JSON format shows the probability for this customer to be in each group:
+ For HIGH is ``4.243616931926422E-6``  therefore it is  ``0.000004243616931926422`` which is 0.0004243616931926422% a very small probability.
+ For **LOW** is ``0.9981701403896932``  which is 99% very close to 100%.
+ For MEDIUM is ``1.3669851638941142E-5``  which is 0.00013% a very small probability.
+ For VERY HIGH is ``0.0018119461417358974``  which is 0.18% a very small probability.


We can test now our second candidate:  AL FRANK. Copy the following Json data in the Body tab and click Send.

````
<copy>
{
   "inputRecords":[
      {
         "CUST_ID":"CU12331",
         "LAST":"AL",
         "FIRST":"FRANK",
         "STATE":"WA",
         "REGION":"West",
         "SEX":"M",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"No",
         "AGE":21,
         "HAS_CHILDREN":0,
         "SALARY":75415,
         "N_OF_DEPENDENTS":3,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":0,
         "TIME_AS_CUSTOMER":3,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":250,
         "CHECKING_AMOUNT":25,
         "MONEY_MONTLY_OVERDRAWN":53.06,
         "T_AMOUNT_AUTOM_PAYMENTS":257,
         "MONTHLY_CHECKS_WRITTEN":0,
         "MORTGAGE_AMOUNT":0,
         "N_TRANS_ATM":0,
         "N_MORTGAGES":0,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":1500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":0
      }
   ]
}
</copy>
````

![Postman Model Score Al Frank Result](images/automl-screenshot-32.jpg)


Notice the result in JSON format shows the probability for this customer to be in each group:
- For HIGH is ``0.0022210521197792597`` which is 0.22% a very small probability.
- For LOW is ``9.765441243777847E-9``  which is 0.00000009% a very small probability.
- For **MEDIUM** is ``0.9977784176843738``  which is 99.7% very close to 100%.
- For VERY HIGH is ``5.20430405663427E-7``  which is 0.0000052% a very small probability.


We can test now our third candidate: ELLIOT PADGETT. Copy the following Json data in the Body tab and click Send.

````
<copy>
{
   "inputRecords":[
      {
         "CUST_ID":"CU12286",
         "LAST":"ELLIOT",
         "FIRST":"PADGETT",
         "STATE":"NV",
         "REGION":"Southwest",
         "SEX":"M",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"Yes",
         "AGE":23,
         "HAS_CHILDREN":0,
         "SALARY":69724,
         "N_OF_DEPENDENTS":3,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":1,
         "TIME_AS_CUSTOMER":3,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":500,
         "CHECKING_AMOUNT":81,
         "MONEY_MONTLY_OVERDRAWN":52.96,
         "T_AMOUNT_AUTOM_PAYMENTS":164,
         "MONTHLY_CHECKS_WRITTEN":4,
         "MORTGAGE_AMOUNT":250,
         "N_TRANS_ATM":2,
         "N_MORTGAGES":1,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":1500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":250
      }
   ]
}
</copy>
````

![Postman Model Score ELLIOT PADGETT Result](images/automl-screenshot-33.jpg)


Notice the result in JSON format shows the probability for this customer to be in each group:
- For **HIGH** is ``0.9999980387263148`` which is 99.9% very close to 100%.
- For LOW is ``1.8967182019052724E-6``  which is 0.00001% a very small probability.
- For MEDIUM is ``6.451060389893198E-8``  which is 0.00000064% a very small probability.
- For VERY HIGH is ``4.4879376221183636E-11``  which is 0.0000000004% a very small probability.



## Task 5:   Score the model using CURL

## Task 5.1:   Prepare REST calls

To access Oracle Machine Learning Services using the REST API, you must provide an access token. To authenticate and obtain an access token, use cURL with the -d option to pass the user name and password for your Oracle Machine Learning Services account against the Oracle Machine Learning User Management Cloud Service REST endpoint /oauth2/v1/token.

* In the VM with CURL, export the environment

````
<copy>
export omlserver=https://adb.<region-prefix>.oraclecloud.com
export tenant=<tenancy-ocid>
export database=<dbname>
export username=OMLUSER
export password=Welcome12345
</copy>
````

- Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.
 - Replace **`<tenancy-ocid>`** with your the tenancy OCID as described in the Preparation Task.
 - Replace **`<dbname>`** with your Autonomous Transaction Processing database name. You can review the Preparation Task for the Autonomous database name.

* Run this statement to get the token to be used later  (replace what is needed)

````
$<copy>curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{"grant_type":"password", "username":"'${username}'", "password":"'${password}'"}' "${omlserver}/omlusers/tenants/${tenant}/databases/${database}/api/oauth2/v1/token"</copy>

{"accessToken":"eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJPTUxVU0VSIiwidGVuYW50X25hbWUiOiJPQ0lEMS5URU5BTkNZLk9DMS4uQUFBQUFBQUFGSjM3TVlUWDIyT1FVT1JDWk5MRlVINzdDRDQ1SU5UN1RUN0ZPMjdUVUVKU0ZRQllCWlJRIiwiZGF0YWJhc2VfbmFtZSI6IkFUUDA0Iiwicm9sZXMiOiJbe1wicm9sZVwiOlwiT01MX0RFVkVMT1BFUlwiLFwiY29tbW9uXCI6ZmFsc2V9XSIsImlzcyI6IkIxOTBDQUJBMEY3ODYxQUFFMDUzNkYxODAwMEE1M0JDIiwiZXhwIjoxNjI2NzkwNTMyLCJpYXQiOjE2MjY3ODY5MzJ9.V61pqfjXRWe4v-nqaxSPGvdCLWDvrUszLOCc_GWdKNJWXaNAsxR_b_BgwbrSBY2rJJJ0XchlDP9khFX1vhBVgHxUJfHGW9sdYPyu5KgGozRENldjte57E2XeupUqNkQot7APTu0mmpMufF_HOSW__I65TpXxPrB9Rv3EHkT9gaOhFQTj_xByAXTqZI7inSxxa5p6AOszoEuylF7wikO1WAT_GcJaCmUqLevsoc8QNNQFCUo3g_918wgiJWYqtf5qw6ZuxNi6HOjUCR8Pi722PP6H7Q1E5WwIIl9qSnMPQTeYcMO34wD58MngkJ9N0D51BK5QS6K0Da4QLPrLmDFACQ!NL/okRBiTH9JfS2eeuG+mRlNUOwD4Qxq6/VGDYIBuQrYN4E+8en/OmEjKEdduFcJZe+747aXXrVfA61zJ38AjIvWOCdS7WnoJ156Ohx541/a28+vpBbwXhkCxogyDXphpqE63oKP75hCKgKPDZWWhPKhJaWeMcFy2xpRq1bt0Vz4zthhv7XHANx2TZDs1oj684PiPSAXX1seJSy4TFgyV9OrOgCThkZe5rPs7LIlR46bKCuYb4mXs47i8crqu71Jv2bit7dgtMetrwlgVywz9PZSl3WPrHEzxeqH9iF82DEIa6tlH/EUy0B9OC6Fc5LB4WeeQfUwiumoXMr0iEdACA==","expiresIn":3600,"tokenType":"Bearer"}
````
Copy the token starting from ``:"``  up until ``==``. In the above example the token is:
`` eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJPTUxVU0VSIiwidGVuYW50X25hbWUiOiJPQ0lEMS5URU5BTkNZLk9DMS4uQUFBQUFBQUFGSjM3TVlUWDIyT1FVT1JDWk5MRlVINzdDRDQ1SU5UN1RUN0ZPMjdUVUVKU0ZRQllCWlJRIiwiZGF0YWJhc2VfbmFtZSI6IkFUUDA0Iiwicm9sZXMiOiJbe1wicm9sZVwiOlwiT01MX0RFVkVMT1BFUlwiLFwiY29tbW9uXCI6ZmFsc2V9XSIsImlzcyI6IkIxOTBDQUJBMEY3ODYxQUFFMDUzNkYxODAwMEE1M0JDIiwiZXhwIjoxNjI2NzkwNTMyLCJpYXQiOjE2MjY3ODY5MzJ9.V61pqfjXRWe4v-nqaxSPGvdCLWDvrUszLOCc_GWdKNJWXaNAsxR_b_BgwbrSBY2rJJJ0XchlDP9khFX1vhBVgHxUJfHGW9sdYPyu5KgGozRENldjte57E2XeupUqNkQot7APTu0mmpMufF_HOSW__I65TpXxPrB9Rv3EHkT9gaOhFQTj_xByAXTqZI7inSxxa5p6AOszoEuylF7wikO1WAT_GcJaCmUqLevsoc8QNNQFCUo3g_918wgiJWYqtf5qw6ZuxNi6HOjUCR8Pi722PP6H7Q1E5WwIIl9qSnMPQTeYcMO34wD58MngkJ9N0D51BK5QS6K0Da4QLPrLmDFACQ!NL/okRBiTH9JfS2eeuG+mRlNUOwD4Qxq6/VGDYIBuQrYN4E+8en/OmEjKEdduFcJZe+747aXXrVfA61zJ38AjIvWOCdS7WnoJ156Ohx541/a28+vpBbwXhkCxogyDXphpqE63oKP75hCKgKPDZWWhPKhJaWeMcFy2xpRq1bt0Vz4zthhv7XHANx2TZDs1oj684PiPSAXX1seJSy4TFgyV9OrOgCThkZe5rPs7LIlR46bKCuYb4mXs47i8crqu71Jv2bit7dgtMetrwlgVywz9PZSl3WPrHEzxeqH9iF82DEIa6tlH/EUy0B9OC6Fc5LB4WeeQfUwiumoXMr0iEdACA==``

Don't forget to add a single quote at the beginning and at the end.

 ````
$<copy> export token='eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJPTUxVU0VSIiwidGVuYW50X25hbWUiOiJPQ0lEMS5URU5BTkNZLk9DMS4uQUFBQUFBQUFGSjM3TVlUWDIyT1FVT1JDWk5MRlVINzdDRDQ1SU5UN1RUN0ZPMjdUVUVKU0ZRQllCWlJRIiwiZGF0YWJhc2VfbmFtZSI6IkFUUDA0Iiwicm9sZXMiOiJbe1wicm9sZVwiOlwiT01MX0RFVkVMT1BFUlwiLFwiY29tbW9uXCI6ZmFsc2V9XSIsImlzcyI6IkIxOTBDQUJBMEY3ODYxQUFFMDUzNkYxODAwMEE1M0JDIiwiZXhwIjoxNjI2NzkwNTMyLCJpYXQiOjE2MjY3ODY5MzJ9.V61pqfjXRWe4v-nqaxSPGvdCLWDvrUszLOCc_GWdKNJWXaNAsxR_b_BgwbrSBY2rJJJ0XchlDP9khFX1vhBVgHxUJfHGW9sdYPyu5KgGozRENldjte57E2XeupUqNkQot7APTu0mmpMufF_HOSW__I65TpXxPrB9Rv3EHkT9gaOhFQTj_xByAXTqZI7inSxxa5p6AOszoEuylF7wikO1WAT_GcJaCmUqLevsoc8QNNQFCUo3g_918wgiJWYqtf5qw6ZuxNi6HOjUCR8Pi722PP6H7Q1E5WwIIl9qSnMPQTeYcMO34wD58MngkJ9N0D51BK5QS6K0Da4QLPrLmDFACQ!NL/okRBiTH9JfS2eeuG+mRlNUOwD4Qxq6/VGDYIBuQrYN4E+8en/OmEjKEdduFcJZe+747aXXrVfA61zJ38AjIvWOCdS7WnoJ156Ohx541/a28+vpBbwXhkCxogyDXphpqE63oKP75hCKgKPDZWWhPKhJaWeMcFy2xpRq1bt0Vz4zthhv7XHANx2TZDs1oj684PiPSAXX1seJSy4TFgyV9OrOgCThkZe5rPs7LIlR46bKCuYb4mXs47i8crqu71Jv2bit7dgtMetrwlgVywz9PZSl3WPrHEzxeqH9iF82DEIa6tlH/EUy0B9OC6Fc5LB4WeeQfUwiumoXMr0iEdACA=='</copy>
 ````
## Task 5.2:  Use REST calls to predict customer classification

* We will check the predicted classification for 3 distinct customers

 ````
<copy>curl -X POST "${omlserver}/omlmod/v1/deployment/classsvmg/score" \
--header "Authorization: Bearer ${token}" \
--header 'Content-Type: application/json' \
-d '{
   "inputRecords":[
      {
         "CUST_ID":"CU12350",
         "LAST":"FRAN",
         "FIRST":"HOBBS",
         "STATE":"NV",
         "REGION":"Southwest",
         "SEX":"F",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"No",
         "AGE":21,
         "HAS_CHILDREN":0,
         "SALARY":66180,
         "N_OF_DEPENDENTS":5,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":0,
         "TIME_AS_CUSTOMER":5,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":500,
         "CHECKING_AMOUNT":185,
         "MONEY_MONTLY_OVERDRAWN":53.21,
         "T_AMOUNT_AUTOM_PAYMENTS":197,
         "MONTHLY_CHECKS_WRITTEN":4,
         "MORTGAGE_AMOUNT":0,
         "N_TRANS_ATM":3,
         "N_MORTGAGES":0,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":2500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":0
      }
   ]
}'
</copy>
 ````

  The results for the customer with ``CUST_ID = CU12350`` , ``LAST = FRAN``, ``FIRST = HOBBS`` are similar to:
 ``{"scoringResults":[{"classifications":[{"label":"HIGH","probability":4.3624052479934053E-7},{"label":"LOW","probability":0.9999549438897685},{"label":"MEDIUM","probability":3.252624363364432E-5},{"label":"VERY HIGH","probability":1.2093626073239925E-5}]}]}``

 Notice the probability for this customer to be in each group:
+ For HIGH is ``4.3624052479934053E-7``  therefore it is  ``0.00000043624052479934053`` which is 0.000043624052479934053% a very small probability.
+ For LOW is ``0.9999549438897685``  which is 99% very close to 100%.
+ For MEDIUM is ``3.252624363364432E-5``  which is 0.00032% a very small probability.
+ For VERY HIGH is ``1.2093626073239925E-5``  which is 0.00012% a very small probability.


````
<copy>
  curl -X POST "${omlserver}/omlmod/v1/deployment/classsvmg/score" \
--header "Authorization: Bearer ${token}" \
--header 'Content-Type: application/json' \
-d '{
   "inputRecords":[
      {
         "CUST_ID":"CU12331",
         "LAST":"AL",
         "FIRST":"FRANK",
         "STATE":"WA",
         "REGION":"West",
         "SEX":"M",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"No",
         "AGE":21,
         "HAS_CHILDREN":0,
         "SALARY":75415,
         "N_OF_DEPENDENTS":3,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":0,
         "TIME_AS_CUSTOMER":3,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":250,
         "CHECKING_AMOUNT":25,
         "MONEY_MONTLY_OVERDRAWN":53.06,
         "T_AMOUNT_AUTOM_PAYMENTS":257,
         "MONTHLY_CHECKS_WRITTEN":0,
         "MORTGAGE_AMOUNT":0,
         "N_TRANS_ATM":0,
         "N_MORTGAGES":0,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":1500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":0
      }
   ]
}'
</copy>
 ````

  The results for the customer ``CUST_ID = CU12331`` , ``LAST = AL`` , ``FIRST = FRANK`` are similar to:
 ``{"scoringResults":[{"classifications":[{"label":"HIGH","probability":0.03644200787288413},{"label":"LOW","probability":7.010119418858265E-7},{"label":"MEDIUM","probability":0.9635568775103579},{"label":"VERY HIGH","probability":4.136048160814146E-7}]}]}``

 Notice the probability for this customer to be in each group:
- For HIGH is ``0.03644200787288413`` which is 3.6% a very small probability.
- For LOW is ``7.010119418858265E-7``  which is 0.000007% a very small probability.
- For MEDIUM is ``0.9635568775103579``  which is 96.3% very close to 100%.
- For VERY HIGH is ``4.136048160814146E-7``  which is 0.0000041% a very small probability.

 ````
 <copy>
  curl -X POST "${omlserver}/omlmod/v1/deployment/classsvmg/score" \
--header "Authorization: Bearer ${token}" \
--header 'Content-Type: application/json' \
-d '{
   "inputRecords":[
      {
         "CUST_ID":"CU12286",
         "LAST":"ELLIOT",
         "FIRST":"PADGETT",
         "STATE":"NV",
         "REGION":"Southwest",
         "SEX":"M",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"Yes",
         "AGE":23,
         "HAS_CHILDREN":0,
         "SALARY":69724,
         "N_OF_DEPENDENTS":3,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":1,
         "TIME_AS_CUSTOMER":3,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":500,
         "CHECKING_AMOUNT":81,
         "MONEY_MONTLY_OVERDRAWN":52.96,
         "T_AMOUNT_AUTOM_PAYMENTS":164,
         "MONTHLY_CHECKS_WRITTEN":4,
         "MORTGAGE_AMOUNT":250,
         "N_TRANS_ATM":2,
         "N_MORTGAGES":1,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":1500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":250
      }
   ]
}'
</copy>
 ````

 The results for the customer ``CUST_ID = CU12286`` , ``LAST = ELLIOT`` , ``FIRST = PADGETT`` are similar to:
 {"scoringResults":[{"classifications":[{"label":"HIGH","probability":0.9998831800626283},{"label":"LOW","probability":2.0777460784672917E-6},{"label":"MEDIUM","probability":1.1454951360792583E-4},{"label":"VERY HIGH","probability":1.9267768518468563E-7}]}]}

  Notice the probability for this customer to be in each group:
- For HIGH is ``0.9998831800626283`` which is 99.9% very close to 100%.
- For LOW is ``2.0777460784672917E-6``  which is 0.00002% a very small probability.
- For MEDIUM is ``1.1454951360792583E-4``  which is 0.0011% a very small probability.
- For VERY HIGH is ``1.9267768518468563E-7``  which is 0.0000019% a very small probability.




## Task 5.3:  Verify the classification prediction

* Return to OML services in Autonomous Database
* Run the following SQL statement using the same ``CUST_IDs`` as in the REST call. You can replace the model name with the one used previously.

 ````
 <copy>%sql
  SELECT a.cust_id,
        a. Last,
        a.First,
        PREDICTION(SVMG USING a.*) PREDICTION,
        PREDICTION_PROBABILITY(SVMG USING a.*)
        PREDICTION_PROBABILITY,
        b.LTV_BIN
  FROM Customer_insurance_test_clasification a,
  Customer_insurance b
 where a.cust_id = b.cust_id
 and b.cust_id in ('CU12350','CU12331', 'CU12286')
 </copy>
 ````

![Classification Prediction](images/automl-screenshot-34.jpg)


Notice the predictions are the same as in the REST calls. In SQL statement it is returned the most probable group or class for the data provided. In out case the prediction is the same as the actual ``LTB_BIN`` column in ``CUSTOMER_INSURANCE`` initial table.


## Task 6 Import the Decision Tree model

In this section of the workshop we will import the Decision Tree model used on the OML4PY < !--  link to OMPL4Py to be added --> workshop Task 3.

We are going to run the next steps:
 1. Extract the saved model to a file;
 2. Load the model in the Autonomous Database repository;
 3. Deploy the model;
 4. Score a customer using the Decision Tree model;


## Task 6.1: Extract the saved model to a file (Optional)

When running OML-Services standalone you can skip this step and got to Task 6.2. (You can run this task only if you went through the OML4Py workshop and run OML-Services workshop as an added step. )

* Connect to the VM and open a Terminal window.

  ![VM-Terminal](images/model-import-01.jpg)


* Go to the OML_Services folder.

````
$<copy> cd ~/OML-Services</copy>
````

  ![VM-Terminal](images/model-import-03.jpg)


* Run the `prepare_model_import.sql` script to write the exported DT_MODEL to a file on the operating system.
Please replace the **`<ServiceName>`** tag with the actual service name of the database instance. For example **`mlpdb1.sub09301404280.omlvcn.oraclevcn.com`**.


````
$ <copy> sqlplus sys/MLlearnPTS#21_@localhost:1521/<ServiceName> as sysdba @prepare_model_import.sql</copy>
````

The script runs the following steps.

  ![VM-Terminal](images/model-import-04.jpg)

  - Creates the **`dmuser`** OS folder
  - Creates the **`DMDIR`** database directory
  - Creates a procedure that writes a BLOB to a file
  - Writes the saved Decision Tree model to a file named **`DTModel.mod`**

  Now we have the model saved in a file, the next steps are to load the model in Autonomous Database, deploy it there and score a customer.

To check the exported file run the command:

````
$<copy>ls -alt </copy>
````

  ![VM-Terminal](images/model-import-05.jpg)



## Task 6.2: Load the model in the Autonomous Database repository

*  In the Postman session opened at Task 4 run the following Get method to get the list of models deployed.

````
Operation: GET

URI endpoint:
<copy>https://adb.<region-prefix>.oraclecloud.com/omlmod/v1/models </copy>

````
 - Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.

 In the Authorization tab pick **Bearer Token** and paste the token copied from Task 4.

 ![Model Import](images/model-import-06.jpg)

The response is just the **SVMG** model we used in our previous tasks.

  ![Model Import](images/model-import-07.jpg)

In case you get the Expired Token Error, rerun the generate Token command explained in **Task 4.1**.

* Open a new tab in Postman and run the following POST command to load the model in Autonomous Database.

````
Operation: POST

URI endpoint:
<copy>https://adb.<region-prefix>.oraclecloud.com/omlmod/v1/models </copy>

````
 - Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.

 In the Authorization tab pick **Bearer Token** and the token is pre-filled.

 ![Model Import](images/model-import-08.jpg)

In the Headers tab enter the following:

````
Content-Type: multipart/form-data
boundary: Boundary
````

  ![Model Import](images/model-import-09.jpg)


In the Body tab pick form-data and enter `modelData` in the key field, hover over the right side of the field and pick File in the dropdown.

  ![Model Import](images/model-import-10.jpg)

Click Select File; go to `/home/oracle/OML-Services/` folder and select the **`DTModel.mod`** file created earlier.

  ![Model Import](images/model-import-11.jpg)

Add the next keys in the Body tab and click Send.

````
 modelName=DTModel
 modelType=OML
 version=1.0
````
  ![Model Import](images/model-import-12.jpg)

The response is that the model is created.

 ![Model Import](images/model-import-13.jpg)

Copy the **`modelId`** displayed in the JSON response.


## Task 6.3: Deploy the model

* Open a new tab in Postman and run the following POST command to deploy the model in Autonomous Database.

````
Operation: POST

URI endpoint:
<copy>https://adb.<region-prefix>.oraclecloud.com/omlmod/v1/deployment </copy>

````
  - Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.

In the Authorization tab pick **Bearer Token** and the token is pre-filled.

  ![Model Deploy](images/model-import-14.jpg)  

In the Headers tab enter the following:

````
Content-Type: application/json
````

  ![Model Deploy](images/model-import-15.jpg)

 In the Body tab enter the following JSON:

````
 {
   "uri":"dtmodel",
   "modelId":"50ffc903-aaf6-4a94-a228-4cb7ab6b3c07"
}
````

Where `50ffc903-aaf6-4a94-a228-4cb7ab6b3c07` is the model Id copied from the previous step. Replace it with the returned model Id from your command in Task 6.2.

The URI **`dtmodel`** we are going to use in the Task 6.4 when scoring the data against our model.

  ![Model Deploy](images/model-import-16.jpg)

And the result is:

 ![Model Deploy](images/model-import-17.jpg)

The next step is to score a customer.


## Task 6.4: Score a customer using the Decision Tree model

In this step we are going to score Fran Hobbs against our Decision Tree imported model.

Enter the following details:

````
Operation: POST

URI endpoint:
<copy>https://adb.<region-prefix>.oraclecloud.com/omlmod/v1/deployment/<model_URI>/score </copy>

````
 - Replace **`<region-prefix>`** with your region. In our case: _eu-frankfurt-1_.

 - Replace **`<model_URI>`** with the model URI that we defined in previous task: **`dtmodel`**

In the Authorization tab pick **Bearer Token** and the token is pre-filled.

 ![Model Score](images/model-import-18.jpg)  


In the header Tab enter the details:

````
--header 'Content-Type: application/json'

````

  ![Model Score](images/model-import-19.jpg)  


In the Body tab enter the Fran Hobs data in RAW format. Notice that we are not providing any of the ``LTV`` or ``LTV_BIN`` data, but provide fully the other important data

````
<copy>
{
   "inputRecords":[
      {
         "CUST_ID":"CU12350",
         "LAST":"FRAN",
         "FIRST":"HOBBS",
         "STATE":"NV",
         "REGION":"Southwest",
         "SEX":"F",
         "PROFESSION":"Programmer/Developer",
         "BUY_INSURANCE":"No",
         "AGE":21,
         "HAS_CHILDREN":0,
         "SALARY":66180,
         "N_OF_DEPENDENTS":5,
         "CAR_OWNERSHIP":1,
         "HOUSE_OWNERSHIP":0,
         "TIME_AS_CUSTOMER":5,
         "MARITAL_STATUS":"SINGLE",
         "CREDIT_BALANCE":0,
         "BANK_FUNDS":500,
         "CHECKING_AMOUNT":185,
         "MONEY_MONTLY_OVERDRAWN":53.21,
         "T_AMOUNT_AUTOM_PAYMENTS":197,
         "MONTHLY_CHECKS_WRITTEN":4,
         "MORTGAGE_AMOUNT":0,
         "N_TRANS_ATM":3,
         "N_MORTGAGES":0,
         "N_TRANS_TELLER":2,
         "CREDIT_CARD_LIMITS":2500,
         "N_TRANS_KIOSK":4,
         "N_TRANS_WEB_BANK":0
      }
   ]
}
</copy>
````

Click Send

  ![Model Score](images/model-import-20.jpg)  

Notice the response for this scoring.

  ![Model Score](images/model-import-21.jpg)  

In this case the percentages are different but it still has the highest probability to be in the **LOW** category as it was in our Support Vector Machine Gaussian model.

In this exercise we managed took a model exported in the Python workshop and imported, deployed and scored it in Autonomous Database using the REST API interface.



## Acknowledgements
* **Authors** -  Andrei Manoliu, Milton Wan
* **Contiributors** - Rajeev Rumale
* **Last Updated By/Date** -  Andrei Manoliu, November 2021

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
