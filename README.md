# Integration inventory data from Google Cloud SQL database to BigQuery using Google Cloud Data Fusion

![whole process](https://user-images.githubusercontent.com/98153604/151255830-ae47f9d9-9435-458d-bcbc-194b22549a47.png)

For business reason, database and data warehouse are both useful for a company to save, check, and study its inventory state, but they have different function. Here I use inventory data to create a database in google cloud SQL for MySQL server, and then create a ETL data pipeline for data from MySQL data base on cloud (cloud SQL) to BigQuery.
Every product stock balance can be caculate and update in Bigquery automatically when Inventory database update.

Here, I send Inventory data from MySQL database to BigQuery without coding for the pipeline design, I use leverage Google’s Cloud Data Fusion to accomplish this goal. 

The whole configurations may involve the following steps:

1. Set up Google cloud SQL for MySQL Server database, and create a database
2. load data to Cloud SQL from Google cloud storage
3. Create and run a Cloud Data Fusion ETL job
4. View the results in BigQuery

## 1. Set up Google cloud SQL for MySQL Server database, and create a database

(1) Create a Cloud SQL instance

   In the Google Cloud Console, Select Navigation menu > SQL (in the Databases section).
   
   Click Create instance.

   Click Choose MySQL.
   
   ![SQL instance](https://user-images.githubusercontent.com/98153604/151021358-f3d241cd-6adf-43c3-afcb-04815fbbc2f6.JPG)

 (2) create database in cloud SQL
 
 Find the Connect to this instance box on the page and click on Open Cloud Shell: gcloud sql connect rentals --user=root --quiet, 
    Press ENTER.
    
 run SQL code to create database inventoryDB:
    
      CREATE DATABASE IF NOT EXISTS inventoryDB;
      USE inventoryDB;

      DROP TABLE IF EXISTS inventory_begining;
      DROP TABLE IF EXISTS inventory_calendar;
      DROP TABLE IF EXISTS product;
      DROP TABLE IF EXISTS units_in;
      DROP TABLE IF EXISTS units_out;


      CREATE TABLE inventory_begining (
        ProductAlternateKey varchar(25) NOT NULL,
        DateKey int,
        UnitCost float,
        UnitsBalance int);

      create table inventory_calendar (
      DateKey int,
      FullDateAlternateKey date,
      DayNumberOfWeek int,
      EnglishDayNameOfWeek varchar(10),
      DayNumberOfMonth int,	
      DayNumberOfYear int,
      WeekNumberOfYear int,
      EnglishMonthName varchar(10),
      MonthNumberOfYear int,
      CalendarQuarter int,	
      CalendarYear int,	
      CalendarSemester int,	
      FiscalQuarter int,	
      FiscalYear int,
      FiscalSemester int);

      create table product(
      ProductAlternateKey varchar(25),
      WeightUnitMeasureCode varchar(10) default 'LB',
      SizeUnitMeasureCode varchar(10) default 'CM',
      EnglishProductName text(50),
      StandardCost float, 
      Color text(15),
      SafetyStockLevel int,
      ReorderPoint int,
      ListPrice float,
      Size text(50),
      SizeRange text(50),
      Weight float,
      DaysToManufacture int,
      DealerPrice float,
      ModelName text(50),
      EnglishDescription text(400));


      create table units_in (
      DateKey int,
      ProductAlternateKey varchar(25),
      UnitCost float,
      UnitsIn int);

      create table units_out (
      DateKey int,
      ProductAlternateKey varchar(25),
      UnitCost float,
      UnitsOut int);


      create view inventory_list as select product.*, inventory_calendar.*
      from product
      cross join inventory_calendar;


      alter table product 
      add primary key (ProductAlternateKey);

      alter table inventory_calendar
      add primary key (DateKey);


      ALTER TABLE inventory_begining
      ADD FOREIGN KEY (ProductAlternateKey) REFERENCES product (ProductAlternateKey);

      ALTER TABLE inventory_begining
      ADD FOREIGN KEY (DateKey) REFERENCES inventory_calendar(DateKey);

      ALTER TABLE units_in
      ADD FOREIGN KEY (ProductAlternateKey) REFERENCES product (ProductAlternateKey);

      ALTER TABLE units_in
      ADD FOREIGN KEY (DateKey) REFERENCES inventory_calendar (DateKey);

      ALTER TABLE units_out
      ADD FOREIGN KEY (ProductAlternateKey) REFERENCES product (ProductAlternateKey);

      ALTER TABLE units_out
      ADD FOREIGN KEY (DateKey) REFERENCES inventory_calendar (DateKey);
      
   I connect Google cloud SQL with local MySQL workbenth, and get the ERD for database inventoryDB:
   ![database ERD](https://user-images.githubusercontent.com/98153604/151031746-65b11670-3469-4e83-b0ac-88533752019f.JPG)\
   
  ## 2. load data to Cloud SQL from Google cloud storage
      
  (1).first copy data to google cloud storage bucket
      
  ![google cloud storage](https://user-images.githubusercontent.com/98153604/151032350-2734020d-f0bc-419c-b113-abf995dcccab.JPG)
      
  (2). import data from google cloud storage bucket to cloud SQL database
      
  ![import data](https://user-images.githubusercontent.com/98153604/151032628-18564a8a-be34-4850-afdb-3ed6e40cf8a0.JPG)
  
  ## 3. Create and run a Cloud Data Fusion ETL job

  
  (1). Enable data fusion API
  
  ![image](https://user-images.githubusercontent.com/98153604/151033425-9ce30e77-aa77-40e1-8efe-4b789ef1a025.png)
  
  (2). Create a Cloud Data Fusion instance
  
      Select Basic for the Edition type.
      Under Authorization section, click Grant Permission.
      
  ![datafusion3 instance](https://user-images.githubusercontent.com/98153604/151036674-688ed51e-1a4c-45c1-8e7f-6643e3fc62ba.JPG)
  
  (3). Download the Driver to connect Cloud SQL for MySQL with Data Fusion
  
  ![hub JDBC Drivers and plugins](https://user-images.githubusercontent.com/98153604/151037046-9a968fb6-ab29-41bb-9814-e5c019782807.JPG)
  ![hub JDBC Drivers and plugins2](https://user-images.githubusercontent.com/98153604/151037095-8e76172a-b223-4e89-914f-ca5163d374fa.JPG)
  
  (4). Config IAM to allowd Data Fusion connect Cloud SQL
  
  ![IAM](https://user-images.githubusercontent.com/98153604/151038104-b007d6a7-d4fc-4011-bcbe-678d2463e5c9.jpg)
  
  (5). Create a ETL work flow
  
 the ETL pipeline like this:
 ![pipeline2](https://user-images.githubusercontent.com/98153604/151041579-b73823cd-73b8-4e97-a99a-b7bc9758d426.JPG)

  click studio to generate a ETL pipeline:
  ![datafusion2](https://user-images.githubusercontent.com/98153604/151038325-4d15f4f0-3517-4687-b62d-50f334fde158.JPG)

  first click cloud SQL MySQL
     
 ![studio2](https://user-images.githubusercontent.com/98153604/151038700-0e9f3e91-1cc9-45a4-b825-014e406ed714.JPG)
 
 Config like:
 ![cloud SQL connnect](https://user-images.githubusercontent.com/98153604/151038842-0a9a9efe-002b-47ce-b3b5-4602850c9036.JPG)
 
 click Joinner, and do table left join
 joiner:
 ![joiner](https://user-images.githubusercontent.com/98153604/151039375-0355eeb2-e009-4de2-a08c-7885f04a8a26.JPG)
 
 joiner1:
 ![joiner1](https://user-images.githubusercontent.com/98153604/151039485-51585624-ec8d-462a-a14e-b359b030a7f0.JPG)
 
 joiner2:
  ![joiner2](https://user-images.githubusercontent.com/98153604/151039540-2b4ff573-cc51-4f45-a2b6-aa5732e6423d.JPG)

The joined table have to do some data cleaning, here I use wangler to clean the data flow from mutiple left join steps:

![wrangler](https://user-images.githubusercontent.com/98153604/151040032-a3432e1a-b672-473f-a158-f14dc26a165c.JPG)

Here, I need to go to cloud bigquery to create a dataset and table, and set table schema for later data load.

Bigquery:
![bigquery](https://user-images.githubusercontent.com/98153604/151040186-d2072f99-7dfd-4cd8-b7a9-673e6e969d93.JPG)

Bigquery execute: data saved in bigquery can directly using SQL syntax in Bigquery to do one step further cleaning, and data calculation to create a new table

        SELECT *,
        FIRST_VALUE(UnitsBalance) OVER (partition by productalternatekey ORDER BY datekey ) 
         + sum(unitsin - UnitsOut) OVER (partition by productalternatekey ORDER BY datekey )
         AS current_balance
         FROM 	`t-osprey-337221.inventory.inventory_in_out2`
         ORDER BY ProductAlternateKey, datekey;

![bigquery execute2](https://user-images.githubusercontent.com/98153604/151041427-f7c8ec1a-cb97-40c9-957e-535f8f413f0f.JPG)

After these steps, click Schedule for schedule the job:
![schedule](https://user-images.githubusercontent.com/98153604/151041896-428e0fcb-f7cd-4575-a9fe-de315d4bbe71.JPG)

Save the pipeline and deploy, and click Run, then the ETL job will start run on cloud:
![pipeline](https://user-images.githubusercontent.com/98153604/151042130-4f9f0994-dafb-45fa-beba-8a427197bcd7.JPG)
![datafusion1](https://user-images.githubusercontent.com/98153604/151044061-e53ee358-0006-4cb7-a7a0-2c07111c20ab.JPG)


After succeeded, go to Bigquery, to check the saved cleaning data:
![bigquery](https://user-images.githubusercontent.com/98153604/151043787-6aff057d-61f5-4f24-9b90-db723aed404b.JPG)

Here, I succussfully create a ETL pipeline, the bigqurey inventory product stock banlance can update automaticlly when database update.

## 4. View the results in Tableau

connect BigQuery with Tableau for data visulization

Here I plot all the products stock balance every day
![inventory stock balance Tableau](https://user-images.githubusercontent.com/98153604/151047881-e61d54d8-6217-4a1d-a38a-12de00c08d73.JPG)


























  


     



  
  



  


    



    


