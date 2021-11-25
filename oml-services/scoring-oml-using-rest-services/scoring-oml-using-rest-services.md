# Scoring OML using Rest Services

In this lab we will learn about accessing ML model using REST services.  We will use postman and curl to test the REST services, the required software is pre-installed on the virtual machine assigned.


Estimated Time: 1 hour

### Objectives
*
*
*

### Prerequisites
*

## Scoring OML using Rest Services
## Task 1: Access the model using REST APIs using POSTMAN

## Task 1.1: Prepare the REST calls


* Connect to the UI of your VM instance accessing the noVNC link. The URL to connect is on the home LiveLabs page.
    ````
    <copy>http://<Instance IP\>:6080/vnc.html?password=MLlearnPTS#21_&resize=scale&quality=9&autoconnect=true</copy>
    ````


* Open a terminal and launch the Postman Application by running
    ````
    <copy>./OML-Services/Postman/Postman</copy>
    ````

  Postman registration is not necessary for this workshop therefore you can choose Skip on the login page.

  ![Launch Postman](images/automl-screenshot-20.jpg)

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


## Task 1.2:  Use REST calls to predict customer classification

  In this Task we can test our prediction for 3 distinct customers from the CUSTOMER\_INSURANCE\_TEST\_CLASIFICATION table:
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



## Task 2: Access the model using REST APIs using CURL

## Task 2.1: Prepare the REST calls

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
## Task 2.2: Use REST calls to predict customer classification

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




## Task 2.3: Verify the classification prediction

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

## Acknowledgements
* **Authors** -  Andrei Manoliu, Milton Wan
* **Contributors** - Rajeev Rumale
* **Last Updated By/Date** -  Andrei Manoliu, October 2021

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
