Create and Manage Cloud Spanner Instances: Challenge Lab (GSP381)
This repository contains the solution and steps to complete the Google Cloud Skills Boost Challenge Lab GSP381, "Create and Manage Cloud Spanner Instances". All commands are executed using the Google Cloud Shell.

Lab Completion Date: 22/07/2025

Prerequisites
Access to Google Cloud Shell.

An active Google Cloud project with the necessary permissions for Cloud Spanner.

Step-by-Step Guide
Task 1: Create a Cloud Spanner instance
First, create the Cloud Spanner instance. Note that the region (--config) might be different for your lab session; please adjust the regional-us-central1 value accordingly.

gcloud spanner instances create banking-ops-instance \
  --config=regional-us-central1 \
  --description="Banking Operations Instance" \
  --nodes=1

Task 2: Create a database in your instance
Next, create the database within the newly created instance.

gcloud spanner databases create banking-ops-db \
  --instance=banking-ops-instance

Task 3: Create tables in your database
Define the database schema by creating the necessary tables using a DDL (Data Definition Language) update.

gcloud spanner databases ddl update banking-ops-db \
  --instance=banking-ops-instance \
  --ddl='
CREATE TABLE Portfolio (
  PortfolioId INT64 NOT NULL,
  Name STRING(MAX),
  ShortName STRING(MAX),
  PortfolioInfo STRING(MAX)
) PRIMARY KEY (PortfolioId);

CREATE TABLE Category (
  CategoryId INT64 NOT NULL,
  PortfolioId INT64 NOT NULL,
  CategoryName STRING(MAX),
  PortfolioInfo STRING(MAX)
) PRIMARY KEY (CategoryId);

CREATE TABLE Product (
  ProductId INT64 NOT NULL,
  CategoryId INT64 NOT NULL,
  PortfolioId INT64 NOT NULL,
  ProductName STRING(MAX),
  ProductAssetCode STRING(25),
  ProductClass STRING(25)
) PRIMARY KEY (ProductId);

CREATE TABLE Customer (
  CustomerId STRING(36) NOT NULL,
  Name STRING(MAX) NOT NULL,
  Location STRING(MAX) NOT NULL
) PRIMARY KEY (CustomerId);
'

Task 4: Insert and modify data in the database tables
Populate the Portfolio, Category, and Product tables with initial data using SQL INSERT statements.

# Insert into Portfolio table
gcloud spanner databases execute-sql banking-ops-db \
  --instance=banking-ops-instance \
  --sql="
INSERT INTO Portfolio (PortfolioId, Name, ShortName, PortfolioInfo) VALUES
(1, 'Banking', 'Bnkg', 'All Banking Business'),
(2, 'Asset Growth', 'AsstGrwth', 'All Asset Focused Products'),
(3, 'Insurance', 'Insurance', 'All Insurance Focused Products');
"

# Insert into Category table
gcloud spanner databases execute-sql banking-ops-db \
  --instance=banking-ops-instance \
  --sql="
INSERT INTO Category (CategoryId, PortfolioId, CategoryName, PortfolioInfo) VALUES
(1, 1, 'Cash', NULL),
(2, 2, 'Investments - Short Return', NULL),
(3, 2, 'Annuities', NULL),
(4, 3, 'Life Insurance', NULL);
"

# Insert into Product table
gcloud spanner databases execute-sql banking-ops-db \
  --instance=banking-ops-instance \
  --sql="
INSERT INTO Product (ProductId, CategoryId, PortfolioId, ProductName, ProductAssetCode, ProductClass) VALUES
(1, 1, 1, 'Checking Account', 'ChkAcct', 'Banking LOB'),
(2, 2, 2, 'Mutual Fund Consumer Goods', 'MFundCG', 'Investment LOB'),
(3, 3, 2, 'Annuity Early Retirement', 'AnnuFixed', 'Investment LOB'),
(4, 4, 3, 'Term Life Insurance', 'TermLife', 'Insurance LOB'),
(5, 1, 1, 'Savings Account', 'SavAcct', 'Banking LOB'),
(6, 1, 1, 'Personal Loan', 'PersLn', 'Banking LOB'),
(7, 1, 1, 'Auto Loan', 'AutLn', 'Banking LOB'),
(8, 4, 3, 'Permanent Life Insurance', 'PermLife', 'Insurance LOB'),
(9, 2, 2, 'US Savings Bonds', 'USSavBond', 'Investment LOB');
"

Task 5: Use a script to load data
Load customer data from a CSV file into the Customer table using a Python script.

Download the data file:

gsutil cp gs://cloud-training/OCBL375/Customer_List_500.csv .

Create the Python script:
Create a new file named load_customer.py (e.g., using nano load_customer.py) and add the following content.

import csv
from google.cloud import spanner

# --- Configuration ---
# Ensure these match your lab's instance and database IDs
INSTANCE_ID = "banking-ops-instance"
DATABASE_ID = "banking-ops-db"

spanner_client = spanner.Client()
instance = spanner_client.instance(INSTANCE_ID)
database = instance.database(DATABASE_ID)

def insert_data():
    with open('Customer_List_500.csv', 'r') as csvfile:
        reader = csv.reader(csvfile)
        rows = [tuple(row) for row in reader] 

    with database.batch() as batch:
        batch.insert(
            table='Customer',
            columns=('CustomerId', 'Name', 'Location'),
            values=rows
        )

if __name__ == "__main__":
    insert_data()
    print("✅ Data successfully inserted.")

Ctrl+O then Enter
Ctrl+X

Execute the script:
Run the script from the Cloud Shell to perform the bulk insert.

python3 load_customer.py

Task 6: Add a column to a table
Modify the schema of an existing table by adding a new column.

gcloud spanner databases ddl update banking-ops-db \
  --instance=banking-ops-instance \
  --ddl='ALTER TABLE Category ADD COLUMN MarketingBudget INT64;'

You can optionally verify the change with the following command:

gcloud spanner databases ddl describe banking-ops-db \
  --instance=banking-ops-instance

This completes all the required tasks for the lab.