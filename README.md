# Integration inventory data from Google Cloud SQL to BigQuery using Google Cloud Data Fusion

![Picture1](https://user-images.githubusercontent.com/98153604/151009547-dbd113f8-968a-4ac2-9112-b16a2d69cc19.png)

ETL data pipeline for data from MySQL data base on cloud (cloud SQL) to BigQuery

For business reason, database and data warehouse are both useful for a company to save, check, and study its inventory state.

For those I wish to send Inventory data from MySQL database to BigQuery without coding for the pipeline design, I use leverage Google’s Cloud Data Fusion to accomplish this goal. 
This pipeline can integration batch data from a Cloud MySQL Server database to a BigQuery table.



The whole configurations may involve the following steps:

Set up MSSQL Server database to enable replication
Create and run a Cloud Data Fusion Replication job
View the results in BigQuery

