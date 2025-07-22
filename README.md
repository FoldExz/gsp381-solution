# Create and Manage Cloud Spanner Instances

**Lab:** GSP381 – Challenge Lab  
**Completed on:** July 22, 2025 at 22:48 (Asia/Jakarta)  
**Environment:** Google Cloud Shell

## Overview

This lab walks you through creating and managing Cloud Spanner instances and databases. You will:
1. Create a Spanner instance.
2. Create a Spanner database.
3. Define database schema (DDL).
4. Insert sample data.
5. Load customer data from a CSV using a Python script.
6. Alter the database schema.

## Prerequisites

- Active Google Cloud project.
- Google Cloud SDK (`gcloud`) installed.
- Python 3 environment.

## Tasks

### Task 1: Create Spanner Instance

```bash
gcloud spanner instances create banking-ops-instance   --config=REGION_CONFIG   --description="Banking Operations Instance"   --nodes=1
```
> Replace `REGION_CONFIG` with a valid regional config (e.g., `regional-us-central1`).

### Task 2: Create Spanner Database

```bash
gcloud spanner databases create banking-ops-db   --instance=banking-ops-instance
```

### Task 3: Define Schema (DDL)

```bash
gcloud spanner databases ddl update banking-ops-db   --instance=banking-ops-instance   --ddl='
CREATE TABLE Portfolio (
  PortfolioId   INT64 NOT NULL,
  Name          STRING(MAX),
  ShortName     STRING(MAX),
  PortfolioInfo STRING(MAX)
) PRIMARY KEY (PortfolioId);

CREATE TABLE Category (
  CategoryId    INT64 NOT NULL,
  PortfolioId   INT64 NOT NULL,
  CategoryName  STRING(MAX),
  PortfolioInfo STRING(MAX)
) PRIMARY KEY (CategoryId);

CREATE TABLE Product (
  ProductId        INT64 NOT NULL,
  CategoryId       INT64 NOT NULL,
  PortfolioId      INT64 NOT NULL,
  ProductName      STRING(MAX),
  ProductAssetCode STRING(25),
  ProductClass     STRING(25)
) PRIMARY KEY (ProductId);

CREATE TABLE Customer (
  CustomerId STRING(36) NOT NULL,
  Name       STRING(MAX) NOT NULL,
  Location   STRING(MAX) NOT NULL
) PRIMARY KEY (CustomerId);
'
```

### Task 4: Insert Sample Data

```bash
gcloud spanner databases execute-sql banking-ops-db   --instance=banking-ops-instance   --sql="
INSERT INTO Portfolio (PortfolioId, Name, ShortName, PortfolioInfo) VALUES
  (1, 'Banking', 'Bnkg', 'All Banking Business'),
  (2, 'Asset Growth', 'AsstGrwth', 'All Asset Focused Products'),
  (3, 'Insurance', 'Insurance', 'All Insurance Focused Products');
"

gcloud spanner databases execute-sql banking-ops-db   --instance=banking-ops-instance   --sql="
INSERT INTO Category (CategoryId, PortfolioId, CategoryName, PortfolioInfo) VALUES
  (1, 1, 'Cash', NULL),
  (2, 2, 'Investments - Short Return', NULL),
  (3, 2, 'Annuities', NULL),
  (4, 3, 'Life Insurance', NULL);
"

gcloud spanner databases execute-sql banking-ops-db   --instance=banking-ops-instance   --sql="
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
```

### Task 5: Load Customer Data via Python

1. Download the CSV:
   ```bash
   gsutil cp gs://cloud-training/OCBL375/Customer_List_500.csv .
   head Customer_List_500.csv
   ```

2. Create `load_customer.py`:
   ```python
   import csv
   from google.cloud import spanner

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
   ```

3. Run the script:
   ```bash
   python3 load_customer.py
   ```

### Task 6: Alter Schema

```bash
gcloud spanner databases ddl update banking-ops-db   --instance=banking-ops-instance   --ddl='ALTER TABLE Category ADD COLUMN MarketingBudget INT64;'
```

**Verification (optional):**
```bash
gcloud spanner databases ddl describe banking-ops-db   --instance=banking-ops-instance
```

---

*This README provides a concise, step-by-step guide to complete the Cloud Spanner lab.*  
