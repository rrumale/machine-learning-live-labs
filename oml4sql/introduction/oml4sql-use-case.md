# Customer Insurance Use Case for introduction to OML4SQL Workshop

## Introduction

Before we start the lab exercises, let's examine the use case and understand what we are trying to achieve with machine learning.  

We are using customer insurance data for a business selling insurance.  The data contains over 15,000 records of customer information such as name, customer ID, age, marital status, income, and also important business information such as whether the customer bought the insurance, and whether the customer is considered to have business value with an attribute called life time value or LTV.  

The insurance business wants four business cases to get demographics about a set of customers historically known to have insurance, have bought one or the field  BUY_INSURANCE=Yes in his database, with the aim of getting to know your customers and your market better, and launch new campaigns in the marketing area.

### Business Objectives
 
* 1) Find the most atypical members of this group (outlier identification).
* 2) Discover the common demographic characteristics of the most typical customers with insurance. 
* 3) Compute how typical a given new/hypothetical customer is.
* 4) Identify rows that are most atypical in the input dataset. Consider each type of marital status to be separate, so the most anomalous rows per marital status group should be returned.

We can use historical records to make predictions with machine learning.  An employee of the business has been keeping records of the customers on a table with name cust_insur_ltv for many years. 


### OML4SQL Workshop Objectives

In this lab, you will:

* Examine the customer insurance historical data set.
* Review the buy insurance and LTV columns.
* Examine the new customer data set.

### Prerequisites

* Access to the Oracle database containing the customer insurance table.

## **STEP 1:** Review the Customer Insurance Data



## Acknowledgements
* **Author** - Valentin Leonard Tabacaru, Milton Wan, Adrian Castillo
* **Last Updated By/Date** -  Adrian Castillo, September 2021