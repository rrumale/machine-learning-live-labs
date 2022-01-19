# Creating models using OML Notebooks and OML AutoML UI

In this section we will connect to OML Notebooks and split the data in train and test chunks. We will use the train data to run the OML AutoML UI and create a good model to make our prediction. We will deploy the model for use with OML Services REST endpoints in the next sections.



Estimated Time: 15 minutes

### Objectives

* Create an OML notebook
* Split the data into Train and Test Data
* Use OML AutoML UI to create the models
* See the models created and Deploy one


### Prerequisites
* Autonomous Database created
* Data loaded in the database
* OML user created

##                                           

## Task 1: Connect to OML Notebooks and display Insurance Customer Data

* In the Autonomous Database instance details page. Click on the Service Console button.
  ![ADB-instance-home](images/prerequisites-screenshot-22.jpg)

* A new page with the service console is opened. In the Overview section we see the details of this specific instance. We can go to the Development section in the left side.
  ![ADB-service-console](images/prerequisites-screenshot-23.jpg)


* Click on Oracle Machine Learning User Interface.
  ![ADB-service-console](images/prerequisites-screenshot-24.jpg)

* Login to OML Machine Learning User Interface in Autonomous Database

  Access the Oracle Machine Learning Notebooks link and connect with the credentials that we created earlier. In our case the credentials are:

   - Username: **OMLUSER**
   - Password: **Welcome12345**

   ![ADB-OML-connect](images/prerequisites-screenshot-25.jpg)

* Open a Scratchpad

  ![Open-Scratchpad](images/automl-screenshot-1.jpg)


* The notebook server is starting. Once opened we can run a select on the ``CUSTOMER_INSURANCE`` table
    ````
    <copy> select * from customer_insurance;  </copy>
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
    SAMPLE (85) SEED (1)
    where cust_id not in ('CU12350','CU12331', 'CU12286')
    </copy>
    ````
    ![create-training-table](images/automl-screenshot-4.jpg)

    Notice that we skip the ``LTV`` column so it would not influence the results. We keep the ``LTV_BIN`` column to be the target for our supervised learning classification model.
    Our goal is to build a model that can predict which LTV category each customer likely belongs to. For this particular workshop we exclude 3 specific customers so we will score 3 different models using their data.


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


## Task 2: Use OML AutoML UI from Oracle Autonomous Database

* Go to the Main menu on the top left side near the Oracle Machine Learning icon.
![AutoML-menu](images/automl-screenshot-X06.jpg)


* Choose AutoML.
![AutoML-menu](images/automl-screenshot-6.jpg)


* Click Create in the AutoML Experiments page
![AutoML-create-experiment](images/automl-screenshot-7.jpg)

* In the Create Experiment page choose the following details:

    - Name: **AutoML Classification**
    - Data Source: chose the **CUSTOMER\_INSURANCE\_TRAIN\_CLASSIFICATION** table in the OMLUSER schema.
    - Predict: **LTV_BIN**
    - Prediction Type: **CLASSIFICATION**
    - Case ID: **CUST_ID**

    ![AutoML-create-experiment](images/automl-screenshot-8.jpg)

* To make some customizations you can expand the Additional Settings menu

    ![AutoML-additional-settings](images/automl-screenshot-9.jpg)

    Notice that we can set the Database Service Level to High, and select by which metric we should compare the models, and which predefined algorithms to include or exclude from this experiment.

      - Choose the following options for your experiment:

        - Database Service Level: **High**
        - Model Metric: **F1**
        - Weight Option: **Weighted**

    ![AutoML-additional-settings](images/automl-screenshot-9a.jpg)

    The F1 score is the harmonic mean of the precision and recall; where the precision is the number of true positive results divided by the number of all positive results, including those not identified correctly, and the recall is the number of true positive results divided by the number of all samples that should have been identified as positive. Precision is also known as positive predictive value, and recall is also known as sensitivity in diagnostic binary classification.

    **F1-score = 2 × (precision × recall)/(precision + recall)**

    ![Precision and Recall](images/Model_metric.jpg)

* Run OML Auto ML experiment by clicking **```Start```** and **```Better Accuracy```**.

  ![Run-AutoML](images/automl-screenshot-10.jpg)

  The AutoML Classification will run for several minutes showing which top 5 algorithms have a Better Accuracy. The running process takes around 10 minutes.

* And the result of the experiment

  ![Classification Experiment Result](images/automl-screenshot-11.jpg)

  Each model described here is based on one of the automatically selected algorithms. We can click on the model name and see its details.

  ![Chose a model](images/automl-screenshot-12.jpg)

  The model detail window opens and the first tab is Prediction Impacts.

  ![Model Prediction Impacts](images/automl-screenshot-13.jpg)

  Notice how the other predictor columns impact our model differently and which columns have a higher weight. We can click on the Confusion Matrix tab.

  ![Model Confusion Matrix](images/automl-screenshot-14.jpg)

  There we can see for each class: **LOW**, **MEDIUM**, **HIGH**, **VERY HIGH** what percentage of customers correctly predicted for each combination of actual and predicted values.


* We can rename the Support Vector Machine model so it would be easier to recognize in the next sections. For this we can select the model and click on the Rename button.

  ![Model Rename ](images/automl-screenshot-X14.jpg)

* Enter a new Model Name and click OK. In our case we are going to use **SVMG**.

  ![Model Rename ](images/automl-screenshot-X114.jpg)

* The model is now renamed in the Leader Board.

  ![Model Rename ](images/automl-screenshot-X1114.jpg)


## Task 3: Deploy the model for REST API access using OML Services

The next steps are to take a model and deploy it for OML Services for access via Rest endpoints.

* Go to the Main menu on the top left side near the Oracle Machine Learning icon.
  ![AutoML-menu](images/automl-screenshot-X06.jpg)

* Choose Models.
  ![Models Menu](images/automl-screenshot-15.jpg)

* We see the list of models created by the AutoML UI with their specific algorithm and target value.
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


## Task 4: Verify the classification prediction

  * Return to the OML Notebooks Scratchpad we created earlier. Click on the menu and chose Notebooks.
  ![Classification Prediction](images/automl-screenshot-21.jpg)


  * Click on the Scratchpad notebook.
    ![Classification Prediction](images/automl-screenshot-22.jpg)


  * Run the following SQL statement using the ``CUST_IDs`` we picked in the train test split. You can replace the model name with the one used previously.

     ````
     <copy>%sql
      SELECT a.cust_id,
            a. Last,
            a.First,
            PREDICTION(SVMG USING a.*) PREDICTION,
            PREDICTION_PROBABILITY(SVMG USING a.*)
            PREDICTION_PROBABILITY,
            b.LTV_BIN
      FROM Customer_insurance_test_classification a,
      Customer_insurance b
     where a.cust_id = b.cust_id
     and b.cust_id in ('CU12350','CU12331', 'CU12286')
     </copy>
     ````

  ![Classification Prediction](images/automl-screenshot-34.jpg)

 The SQL statement returns the most probable group or class for the data provided in the column PREDICTION with it’s corresponding prediction probabilty. In out case the prediction is the same as the actual ``LTB_BIN`` column in ``CUSTOMER_INSURANCE`` initial table.

## Acknowledgements
* **Authors** -  Andrei Manoliu, Milton Wan
* **Contributors** - Rajeev Rumale
* **Last Updated By/Date** -  Andrei Manoliu, December 2021

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
