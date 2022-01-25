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
    



    


