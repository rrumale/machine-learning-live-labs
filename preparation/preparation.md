# Oracle Machine Learning compute node preparation

## Introduction

Oracle Machine Learning workshop requires a compute node with Oracle Database 21c and OML tools. This instance can be created from a custom image.

### Objectives

In this lab, you will:
* Create the compute node from the custom image.
* Verify installed database and tools.

### Prerequisites

* Access to OCI OraclePartnerSas tenancy.

## Task 1: Create Compute Instance

1. On OCI OraclePartnerSas tenancy, select **Germany Central (Frankfurt)** region.

2. Navigate to Compute > **Custom Images**.

3. Select Compartment oraclepartnersas (root)/Team/VTABACARU/**OMLshowcase**.

4. Next to **OML-WS-IMG** custom image click **⋮** > **Create Instance**.

    - Name: give it a short name, e.g. *oml-vm-xy*.
    - Create in compartment: choose your compartment.
    - Under Network, Select existing virtual cloud network: Virtual cloud network in OMLshowcase, OMLVCN.
    - Under Subnet, Select existing subnet: use the Public Subnet.
    - Public IP Address: it is necessary to Assign a public IPv4 address.

> **Note** : If you want to use your own VCN, change the compartment and select your VCN. You need to have the following ports open in your VCN Public Subnet Security List:
> - TCP 6080 for NoVNC
> - TCP 8888 for Jupyter
> - TCP 8787 for RStudio

5. After the instance is created, copy the Public IP address.


## Task 2: Set the environment and run OML4Py

1. In your browser, go to [**Public-IP**]:6080/index.html?resize=remote

2. Click Connect. Password is **MLlearnPTS#21_**

3. Open a Terminal. Run these commands one by one:

    ````
    fixHost

    . setEnv
    ````

4. This Terminal window must stay open until you finish and close Jupyter Notebook. To close, press **Ctrl-C** and **y**.

5. These are the only commands you have to execute, now the environment is ready to start the hands-on labs.

6. For OML4Py lab all code is ready to run in the 2 Jupyter notebooks that are available: OML4Py-part-1.ipynb and OML4Py-part-2.ipynb. You don't need to copy/paste from the lab guide the code for OML4Py lab.


## Task 3: Only if you restart the compute instance (ONLY IF NECESSARY)

1. Open a Terminal as **oracle** user.

2. Set the environment for Oracle database.

    ````
    . oraenv
    ORACLE_SID = [oracle] ? mlcdb
    The Oracle base has been set to /u01/app/oracle
    ````

3. Start the listener.

    ````
    lsnrctl start
    ````

4. Start the database.

    ````
    sqlplus / as sysdba

    startup

    exit
    ````

5. Go to the Python project folder in Terminal.

    ````
    cd ~/projects/oml4py/
    ````

6. Activate Python environment. Install any Python library in this environment. Ignore the warning message about Pip old version.

    ````
    . orclvenv/bin/activate

    pip install [library]
    ````

7. Start Jupyter Notebook.

    ````
    jupyter notebook --ip=0.0.0.0​
    ````

8. Copy the token value from the output. Jupyter Notebook is automatically launched in a browser window.

    ````
    http://0.0.0.0​:8888/?token=deee4a5109aa637deb61784774b61876a650b87970f2c499
    ````

9. If you want to connect to Jupyter Notebook from your browser, use the Public IP address: 

    ````
    http://[**Public-IP**]:8888/?token=deee4a5109aa637deb61784774b61876a650b87970f2c499
    ````

10. This Terminal window must stay open until you finish and close Jupyter Notebook. To close, press **Ctrl-C** and **y**.

    ````
    Shutdown this notebook server (y/[n])? y
    ````

11. To deactivate the Python environment, run:

    ````
    deactivate
    ````


## Task 4: Connect to RStudio

1. On the remote desktop, launch in a browser window and navigate to:

    ````
    localhost​:8787
    ````

2. If you want to connect to RStudio from your browser, use the Public IP address: 

    ````
    [**Public-IP**]:8787
    ````


## Acknowledgements
* **Author** - Valentin Leonard Tabacaru
* **Last Updated By/Date** -  Valentin Leonard Tabacaru, July 2021
