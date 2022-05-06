# Prepare Setup

## Introduction
This lab will show you how to download the Oracle Resource Manager (ORM) stack zip file required to set up the resource needed for this workshop. We would use it in the next lab to create a compute instance to run the lab tasks.

*Estimated Lab Time:* 15 minutes

### Objectives
-   Download ORM stack need to create Compute Instance(Virtual Machine) for this workshop.

### Prerequisites
This lab assumes you have:
- An Oracle Free Tier or Paid Cloud account
- Have sufficient quota for in your tenancy to create VM and VCN.

## Task 1: Download Oracle Resource Manager (ORM) stack zip file
1.  Click on the link below to download the Resource Manager zip file you need to build your environment: [oml-services-mkplc-freetier.zip](https://objectstorage.us-ashburn-1.oraclecloud.com/p/OZ4PWWp2n3l58f_vpD8oi6wJNwkGV0hseq4F6IsRKEANngipzmpll9tKRSBVQxYu/n/oraclepartnersas/b/omlvm-mkplc-freetier/o/oml-services-mkplc-freetier.zip)

2.  Save the **omlvm-mkplc-freetier.zip** file in your machine and note the folder location.

The stack contains a terraform script that would provision a VCN, Compute instance and an Autonomous database to use in this hands on lab.

## Task 2: Setup Compute   
Using the details from the above task, proceed to the lab *Environment Setup* to setup your workshop environment using Oracle Resource Manager (ORM) and one of the following options:
  - Create Stack:  *Compute only* with an existing VCN where the security list is updated in the resource stack.
  - You must select the availability domain and set the name and admin password for the Autonomous Database.  


You may now [proceed to the next lab](#next).

## Acknowledgements

* **Author** - Rajeev Rumale
* **Contributors** -  
* **Last Updated By/Date** - Rajeev Rumale, April 2022
