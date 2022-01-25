# Integration inventory data from Google Cloud SQL database to BigQuery using Google Cloud Data Fusion

![Picture1](https://user-images.githubusercontent.com/98153604/151009547-dbd113f8-968a-4ac2-9112-b16a2d69cc19.png)

For business reason, database and data warehouse are both useful for a company to save, check, and study its inventory state, but they have different function. Here I use inventory data to create a database in google cloud SQL for MySQL server, and then create a ETL data pipeline for data from MySQL data base on cloud (cloud SQL) to BigQuery.

For those I wish to send Inventory data from MySQL database to BigQuery without coding for the pipeline design, I use leverage Google’s Cloud Data Fusion to accomplish this goal. 

The whole configurations may involve the following steps:

1. Set up Google cloud SQL for MySQL Server database
2. load data to Cloud SQL from Google cloud storage
3. Create and run a Cloud Data Fusion Replication job
4. View the results in BigQuery

## 1. Set up Google cloud SQL for MySQL Server database
