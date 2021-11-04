# Make predictions for Formula 1 season using Oracle Machine Learning

## Introduction

## Objectives

## prerequisites

* The `csv` files;
* Autonomous Database Machine Learning User (in our case `RBRUSER / Welcome12345`)

## Data Loading

1. Upload the files in a Object Storage Bucket
![Data Loading](./images/data_loading_01.jpg)

2. Open Autonomous Database Actions
![Data Loading](./images/data_loading_02.jpg)

Connect with the RBRUSER user and password.
In the Database Actions Launchpad, in the Data Tools section, chose Data Load.
![Data Loading](./images/data_loading_03.jpg)

Chose Load Data and Cloud Storage.
![Data Loading](./images/data_loading_04.jpg)

3. In the Data Load menu, chose a pre-define cloud storage, or define a new cloud storage to the Object Storage Bucket where the files are uploaded.
![Data Loading](./images/data_loading_05.jpg)

Drag each file one by one in the middle.
![Data Loading](./images/data_loading_06.jpg)

Click on the Run button to upload all the files.
![Data Loading](./images/data_loading_07.jpg)


## Data Preparation

1. Connect the the Autonomous Database Machine Learning home with `RBRUSER` and `Welcome12345`;
![Data Preparation](./images/data_preparation_01.jpg)

2. Go to Notebooks
![Data Preparation](./images/data_preparation_02.jpg)

3. Import and open [**`F1_OML_SERVICES`**](https://objectstorage.eu-frankfurt-1.oraclecloud.com/p/bf9m9JiVydMdRhomPo3mq3_ehLsFA3Qoy1gJlmStQZR1EA6q5QeylEY9ihLB99Md/n/oraclepartnersas/b/RBR/o/F1_OML_SERVICES.json) notebook.
![Data Preparation](./images/data_preparation_03.jpg)

4. You can run all paragraphs or each paragraph in turn.

* Create Driver Data View for all the drivers running in 2021
![Data Preparation](./images/data_preparation_04.jpg)

* Create Circuit Data View with details about all the circuits
![Data Preparation](./images/data_preparation_05.jpg)

* Create Extended Dataset table
![Data Preparation](./images/data_preparation_06.jpg)

Run Data Insights on the `F1_dataset_extended` table.

## Data Insights

1. Open Autonomous Database Actions
![Data Insights](./images/data_loading_02.jpg)

2. Connect with the RBRUSER user and password.
In the Database Ations Launchpad, in the Data Tools section, chose Data Insights.
![Data Insights](./images/data_insights_01.jpg)

3. In the Data Insights page chose the following:

* Schema: RBRUSER
* Analytic View/ Table: F1_dataset_extended
* Column: prognostication
![Data Insights](./images/data_insights_02.jpg)

Click Search.
The process might take a while and it starts populating with graphical insights right away.
![Data Insights](./images/data_insights_03.jpg)


One insight that we can see is over QUALI_POS = 01 what is the distribution of constructors:

![Data Insights](./images/data_insights_04.jpg)

## Running AutoML

We are going to create multiple ML models for 3 scenarios.

1. Predict the average lap time per race. we are going to use regression to predict what is going to be the average lap time for a next race looking at past data and driver performance that we have in the `F1_dataset_extended` table.

In the AutoML menu click on the Create Experiment.
In the experiment settings chose:
* Name: F1-lap_time_regression
* Data Source: RBRUSER.F1_DATASET_EXTENDED
* Predict: avg_lap_time
* Case ID: raceId
* Prediction Type: regression
![AutoML](./images/auto_ml_01.jpg)


In the Additiona Settings chose
Database Service Level: HIGH
![AutoML](./images/auto_ml_02.jpg)

In the Features section pick the following columns to be used:
* avg_stop_duration
* circuit
* circuitId
* circuit_laps
* constructor
* constructorId
* constructor_reliability
* country
* driver
* driverId
* driver_confidence
* driver_home
* gp_name
* laps
* points
* position
* stops
* stop_laps
* year
![AutoML](./images/auto_ml_03.jpg)

Click Start for Better Accuracy.
![AutoML](./images/auto_ml_04.jpg)

When the experiment is completed we have a list of model whit their scoring. Chose the one with the highest scoring and click rename to rename the model to `F1_AVG_LAPTIME`.
![AutoML](./images/auto_ml_05.jpg)

You can click on Deploy also to be able to access the model using REST APIs.
![AutoML](./images/auto_ml_06.jpg)

Enter the following:
* Name: F1_AVG_LAPTIME
* URI: laptime
* Version: 1
![AutoML](./images/auto_ml_07.jpg)



2. Predict how many pitstops will a driver have in the race. We are going to use classification to predict the probability of 1, 2,3 or even 4 pit stops for a specific driver looking at past data and driver performance that we have in the `F1_dataset_extended` table.

In the AutoML menu click on the Create Experiment.
In the experiment settings chose:
* Name: F1_stops_classification
* Data Source: RBRUSER.F1_DATASET_EXTENDED
* Predict: stops
* Case ID: raceId
* Prediction Type: CLASSIFICATION
![AutoML](./images/auto_ml_08.jpg)


In the Additiona Settings chose
Database Service Level: HIGH
Model Metric: F1
Weight Option: Weighted

![AutoML](./images/auto_ml_09.jpg)

In the Features section pick the following columns to be used:
* avg_stop_duration
* circuit
* circuitId
* circuit_laps
* constructor
* constructorId
* constructor_nationality
* constructor_dnf
* constructor_home
* constructor_reliability
* country
* driver
* driver_nationality
* driverId
* driver_confidence
* driver_home
* gp_name
* laps
* points
* position
* year
* racetime
* Fastestlaptime
* Fastestlapspeed
* Fastestlap
* points
* psition


![AutoML](./images/auto_ml_10.jpg)


Click Start for Better Accuracy.
![AutoML](./images/auto_ml_11.jpg)

When the experiment is completed we have a list of model whit their scoring. Chose the one with the highest scoring and click rename to rename the model to `F1_stops_rf`.
![AutoML](./images/auto_ml_12.jpg)

You can click on Deploy also to be able to access the model using REST APIs.
![AutoML](./images/auto_ml_13.jpg)

Enter the following:
* Name: F1_stops_rf
* URI: stops
* Version: 1
![AutoML](./images/auto_ml_14.jpg)


3. Predict the Qualifying position of a driver in a race. We are going to use classification to predict the probability of `quali_pos` to be 1, 2,3 up to 20 for a specific driver looking at past data and driver performance that we have in the `F1_dataset_extended` table.
Quali is especially important and can be a deciding factor for the standings at the end of the race. There are tracks where overtaking is difficult and most of the drivers keep the position from quali session.


In the AutoML menu click on the Create Experiment.
In the experiment settings chose:
* Name: f1_quali_prediction
* Data Source: RBRUSER.F1_DATASET_EXTENDED
* Predict: QUALI_POS
* Case ID: raceId
* Prediction Type: CLASSIFICATION

![AutoML](./images/auto_ml_15.jpg)


In the Additiona Settings chose
Database Service Level: HIGH
Model Metric: F1
Weight Option: Weighted

![AutoML](./images/auto_ml_09.jpg)

In the Features section pick the following columns to be used:
* Year
* DOB
* constructor
* constructorId
* constructor_nationality
* constructor_running_dnf
* constructor_reliability
* constructor_home
* constructor_dnf
* driver
* driver_nationality
* age_at_gp_in_days
* driver_confidence
* driver_running_dnf
* driver_dnf
* driverId
* driver_confidence
* driver_home
* laps
* circuit
* circuitId
* circuit_laps
* country
* gp_name
* laps


![AutoML](./images/auto_ml_16.jpg)


Click Start for Better Accuracy.
![AutoML](./images/auto_ml_17.jpg)

When the experiment is completed we have a list of model whit their scoring. Chose the one with the highest scoring and click rename to rename the model to `f1_quali_rf`.
![AutoML](./images/auto_ml_18.jpg)


3. Predict the final Position of a driver in a race. We are going to use classification to predict the probability of `position` to be 1, 2,3 up to 20 for a specific driver looking at past data and driver performance that we have in the `F1_dataset_extended` table.


In the AutoML menu click on the Create Experiment.
In the experiment settings chose:
* Name: f1_position_prediction
* Data Source: RBRUSER.F1_DATASET_EXTENDED
* Predict: POSITION
* Case ID: raceId
* Prediction Type: CLASSIFICATION

![AutoML](./images/auto_ml_19.jpg)


In the Additiona Settings chose
Database Service Level: HIGH
Model Metric: F1
Weight Option: Weighted

![AutoML](./images/auto_ml_09.jpg)


In the Features section pick the following columns to be used:
* Year
* DOB
* quali_pos
* constructor
* constructorId
* constructor_nationality
* constructor_running_dnf
* constructor_reliability
* constructor_home
* constructor_dnf
* driver
* driver_nationality
* age_at_gp_in_days
* driver_confidence
* driver_running_dnf
* driver_dnf
* driverId
* driver_confidence
* driver_home
* laps
* circuit
* circuitId
* circuit_laps
* country
* gp_name
* laps
* stops
* avg_lap_time
* stop_laps

![AutoML](./images/auto_ml_20.jpg)


Click Start for Better Accuracy.
![AutoML](./images/auto_ml_21.jpg)

When the experiment is completed we have a list of model whit their scoring. Chose the one with the highest scoring and click rename to rename the model to `F1_position_rf`.
![AutoML](./images/auto_ml_22.jpg)

We have the models ready, we will use them to make predictions.

## Data Prediction

### Race standings

Returning to our notebook: **`F1_OML_SERVICES`**  and scroll to the bottom to run new paragraphs.

1. List the races in 2021 to see which race in the future we can pick:

````
%sql
<copy> select "date","name" from races where "year"=2021 </copy>
````
![Data Prediction](./images/data_prediction_01.jpg)


2. Run the following query to predict the standings for one of the races that you choose:

````
%sql
<copy> select b.driver, b.constructor,
        PREDICTION(	F1_position_rf USING b.*) Position_PREDICTION,
        PREDICTION_PROBABILITY(	F1_position_rf USING b.*) PRED_PROBABILITY,
        b.QUALI_POS, b.QUALI_POS_probility
FROM (
select a.*,
        PREDICTION(F1_AVG_LAPTIME USING a.*) avg_lap_time,
        PREDICTION(F1_stops_rf USING a.*) stops,
        PREDICTION(f1_quali_rf USING a.*) QUALI_POS,
        PREDICTION_PROBABILITY(f1_quali_rf USING a.*) QUALI_POS_probility
from(
select dd."year",
        dd.driver,
        dd.driverid,
        dd.dob,
        dd.driver_nationality,
        dd.driver_confidence,
        dd.driver_running_dnf,
        dd.constructor,
        dd.constructor_nationality,
        dd.constructor_running_dnf,
        dd.constructor_reliability,
        dd.constructor_dnf
       ,((select max("date") from races where "name"='${GP_NAME}'  and "year"=2021) - to_date(dob,'dd/mm/yyyy')) age_at_gp_in_days,
       cd.name circuit,
       cd.circuitId circuitId,
       cd."year" circuite_year_info,
 --       cd.avg_lap_time,
        cd.laps laps,
 --       cd.stops,
        cd.stop_laps,
        cd.avg_stop_duration
from F1_driverdata_vw dd,
    (select *
        from f1_circuit_data_view where racename='${GP_NAME}'
        and "year" = (select max("year") from f1_circuit_data_view where racename='${GP_NAME}'  )) cd
where  dd.driverId = cd.driverId (+)
and    dd.driver in (select distinct d."forename"||' '||d."surname" Driver
            from
                races r,
                driver_standings  ds,
                drivers d
            where  d."driverId" = ds."driverId"
            and ds."raceId"=r."raceId"
            and r."year" =2021)

and dd."year"=2021
and ( dd."date" = (select max("date") from races where "name"='${GP_NAME}'  and "year"=2021) or
        dd."date" = (select max("date") from F1_driverdata_vw where "year"=2021) )
)a
order by 3,4 desc
) b
order by 3, 4 desc, 5 asc </copy>
````

When you click on Run notice the field with the labe **GP_NAME** at the bottom of the paragraph. Insert ther any Grand Prix name from the above query.


![Data Prediction](./images/data_prediction_02.jpg)

Notice that the 2 contenders for the championship have the highest probabiliy to get goth Pole Position and Race Win.



### Number of Pit Stops

Prerequisites:
* CURL
* DB_Name
* Tenancy OCID
* OML_server
* Username and password ( in this case `RBEUSER/ Welcome12345`)



Open a session in terminal or command line on the machine where CURL is installed and run:

````
<copy>
export omlserver=https://adb.eu-frankfurt-1.oraclecloud.com
export tenant=ocid1.tenancy.oc1..aaaaaaaafj37mytx22oquorcznlfuh77cd45int7tt7fo27tuejsfqbybzrq
export database=OMLDB01
export username=RBRUSER
export password=Welcome12345
</copy>
````

To get the token run:

````
<copy>
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{"grant_type":"password", "username":"'${username}'", "password":"'${password}'"}' "${omlserver}/omlusers/tenants/${tenant}/databases/${database}/api/oauth2/v1/token"
</copy>
````

Save the token or export it in command line:
````
export token='eyJhbGciOiJSUzI1NiJ9.eyJzd...VA=='
````

Let's see the probability for Max Verstapen to have a 2 stop race in Italy at Monza.

````
<copy>
curl -X POST "${omlserver}/omlmod/v1/deployment/stops/score" \
--header "Authorization: Bearer ${token}" \
--header 'Content-Type: application/json' \
-d '{"inputRecords":[{ "year":2021,
   "driver":"Max Verstappen",
    "driver_nationality":"Dut",
    "constructor":"Red Bull",
   "constructor_nationality":"Aus",
   "constructor_reliability":0.8364197530864197530864197530864197530864,
   "circuit":"Autodromo Nazionale di Monza",
   "gp_name":"Italian Grand Prix",
   "avg_lap_time":1499800,
   "laps":53,
   "country":"Italy",
   "avg_stop_duration":774472
 }]}'
</copy>
````

The result is:

````
{
   "scoringResults":[
      {
         "classifications":[
            {
               "label":"1",
               "probability":0.3873006380943179
            },
            {
               "label":"2",
               "probability":0.38444413318446446
            },
            {
               "label":"3",
               "probability":0.15739577933577417
            },
            {
               "label":"4",
               "probability":0.04819505228786759
            },
            {
               "label":"5",
               "probability":0.018634069528431323
            },
            {
               "label":"6",
               "probability":0.004030327569144564
            }
         ]
      }
   ]
}
````

So the both 1 stop and 2 stop strategies are verry possible.

### Average Lap Time


Let's see the average lap time that Max Verstapen should have in Italy at Monza.

````
<copy>
curl -X POST "${omlserver}/omlmod/v1/deployment/laptime/score" \
--header "Authorization: Bearer ${token}" \
--header 'Content-Type: application/json' \
-d '{"inputRecords":[{ "year":2021,
   "driver":"Max Verstappen",
    "driver_nationality":"Dut",
    "constructor":"Red Bull",
   "constructor_nationality":"Aus",
   "constructor_reliability":0.8364197530864197530864197530864197530864,
   "circuit":"Autodromo Nazionale di Monza",
   "gp_name":"Italian Grand Prix",
   "position":1,
    "laps":53,
   "country":"Italy",
   "avg_stop_duration":22000
 }]}'
 }]}'
</copy>
````

The result is:

````
{
   "scoringResults":[
      {
         "regression":85573.97681099334
      }
   ]
}
````
the average is about 85573 milliseconds which converts to `1:25.573` as an average race pace.
