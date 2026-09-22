# Real-World Messy E-Commerce Dataset

A production-grade, intentionally messy e-commerce dataset for building an **end-to-end Data Analytics portfolio project**.

## 📊 Dataset Overview

| Metric | Value |
|--------|-------|
| **Total Records** | 868,050 |
| **Total Tables** | 5 |
| **File Size** | ~128 MB (CSV) |
| **Time Period** | Jan 2023 - Dec 2025 |
| **Currency** | INR (Indian Rupees) |
| **Geography** | India |
| **Data Quality Issues** | ~210,000+ |
| **Reproducibility** | Seed=42 (deterministic) |

---

## 📁 File Structure

```
data/
├── raw/
│   ├── customers.csv       (50,800 records)
│   ├── products.csv        (10,200 records)
│   ├── sellers.csv         (2,050 records)
│   ├── orders.csv          (503,000 records) ⭐
│   └── payments.csv        (302,000 records)
├── cleaned/                (your cleaned data goes here)
└── quality_logs/           (data quality reports)
```

---

## 📋 Table Descriptions

### 1. **customers** (50,800 rows)
Customer master data with demographics and segmentation.
- Key columns: customer_id, name, age, city, segment
- Issues: ~300 duplicates, invalid ages, missing phone numbers

### 2. **products** (10,200 rows)
Product catalog with pricing and inventory.
- Key columns: product_id, category, brand, list_price, stock
- Issues: Invalid prices, missing brands, duplicates

### 3. **sellers** (2,050 rows)
Seller/vendor information.
- Key columns: seller_id, name, city, rating, status
- Issues: Invalid ratings (outside 1-5), missing values, duplicates

### 4. **orders** (503,000 rows) ⭐ **MAIN TABLE**
Main transactional table - one row per order.
- Key columns: order_id, customer_id, product_id, quantity, total_amount
- Issues:
  - ~10,000 duplicate order_ids (2%)
  - ~15,000 orphan records (invalid foreign keys)
  - ~50,000 missing values (dates, ratings, pincodes)
  - ~30,000 invalid values (negative prices, invalid ratings)
  - Logical errors (delivery before order)
  - Inconsistent formats (status, payment method)

### 5. **payments** (302,000 rows)
Payment transaction records.
- Key columns: payment_id, order_id, payment_method, amount, status
- Issues: ~2,000 duplicate attempts, orphan records, missing dates, invalid amounts

---

## 🔍 Data Quality Issues Included

### ✅ Missing Values (~50,000 records)
- order_date: 3%
- shipping_pincode: 3%
- customer_rating: 5%
- payment_date: 3%
- phone (customers): 3%
- seller_rating: 5%

### ✅ Duplicates (~5,550 records)
- Exact duplicate orders: 3,000
- Exact duplicate customers: 300
- Exact duplicate payments: 2,000
- Near-duplicate customers: 500
- Duplicate product records: 200
- Duplicate seller records: 50

### ✅ Invalid Data Types (~30,000 records)
- Negative ages (-5, -1, 0)
- Unrealistic ages (250, 300)
- Negative prices (-100)
- Extremely high prices (999999)
- Negative quantities (-5, -10)
- Unrealistic quantities (9999)
- Negative discounts (-50)
- Discounts exceeding 100% (150, 200)
- Invalid ratings (0, 6, 7, 10)
- Negative transaction amounts

### ✅ Inconsistent Formats (~100,000+ records)
- Gender: 'Male', 'male', 'M', 'FEMALE', 'F', 'Female', 'Unknown'
- Order Status: 'Delivered', 'delivered', 'DELIVERED', 'Shipped', 'shipped', 'Cancelled', 'canceled'
- Payment Method: 'UPI', 'upi', 'Upi', 'Credit Card', 'credit_card', 'COD', 'Cash on Delivery'
- Cities: 'Delhi', 'delhi', 'NEW DELHI', 'new delhi'

### ✅ Logical Errors (~10,000 records)
- Delivery date before order date (~2%)

### ✅ Orphan Records (~15,000 records)
- orders.customer_id referencing non-existent customers
- orders.product_id referencing non-existent products
- payments.order_id referencing non-existent orders

---

## 🚀 Quick Start

### Step 1: Load Data into PostgreSQL

#### Prerequisites
```bash
# Install PostgreSQL
sudo apt-get install postgresql postgresql-contrib

# Start PostgreSQL service
sudo service postgresql start
```

#### Create Database and Load Data

```bash
# Connect to PostgreSQL
psql -U postgres

# In psql prompt:
CREATE DATABASE ecommerce_raw;
\c ecommerce_raw
\i schema.sql

# Load data (adjust paths)
\COPY customers FROM '/path/to/data/raw/customers.csv' WITH (FORMAT csv, HEADER true);
\COPY products FROM '/path/to/data/raw/products.csv' WITH (FORMAT csv, HEADER true);
\COPY sellers FROM '/path/to/data/raw/sellers.csv' WITH (FORMAT csv, HEADER true);
\COPY orders FROM '/path/to/data/raw/orders.csv' WITH (FORMAT csv, HEADER true);
\COPY payments FROM '/path/to/data/raw/payments.csv' WITH (FORMAT csv, HEADER true);
```

#### Verify Data Load

```sql
-- Check row counts
SELECT COUNT(*) as customer_count FROM customers;
SELECT COUNT(*) as product_count FROM products;
SELECT COUNT(*) as seller_count FROM sellers;
SELECT COUNT(*) as order_count FROM orders;
SELECT COUNT(*) as payment_count FROM payments;

-- View data quality issues
SELECT * FROM v_missing_values;
SELECT * FROM v_duplicate_orders;
SELECT * FROM v_invalid_values;
SELECT * FROM v_logical_errors;
SELECT * FROM v_orphan_records;
```

### Step 2: Data Exploration with Python

```python
import pandas as pd
import numpy as np

# Load data
customers = pd.read_csv('data/raw/customers.csv')
orders = pd.read_csv('data/raw/orders.csv')
products = pd.read_csv('data/raw/products.csv')
sellers = pd.read_csv('data/raw/sellers.csv')
payments = pd.read_csv('data/raw/payments.csv')

# Quick stats
print(f"Orders: {len(orders):,} records")
print(f"Duplicate order_ids: {len(orders) - orders['order_id'].nunique():,}")
print(f"\nMissing values:\n{orders.isnull().sum()}")
print(f"\nData types:\n{orders.dtypes}")

# Identify issues
print(f"\nNegative quantities: {(orders['quantity'] < 0).sum()}")
print(f"Invalid ratings: {((orders['customer_rating'] > 5) | (orders['customer_rating'] < 1)).sum()}")
```

### Step 3: Data Cleaning Roadmap

#### Phase 1: Understand the Data (EDA)
```python
# Notebook: 01_data_quality_assessment.ipynb
- Load all 5 tables
- Identify missing values
- Find duplicates
- Detect outliers
- Analyze data types
- Create quality report
```

#### Phase 2: Clean the Data
```python
# Notebook: 02_data_cleaning.ipynb
- Remove/handle duplicates
- Standardize text fields (gender, status, payment method)
- Fix invalid values (ages, prices, ratings)
- Impute missing values
- Fix logical errors (delivery dates)
- Remove orphan records
- Validate foreign keys
- Save cleaned data to data/cleaned/
```

#### Phase 3: Build Analytics
```python
# Notebook: 03_eda.ipynb
- Business metrics (revenue, order count, customer value)
- Customer segmentation analysis
- Product performance analysis
- Seller ratings analysis
- Time series analysis
- Create visualization-ready data
- Export to Power BI
```

---

## 💾 How to Use Each CSV File

### customers.csv
```python
df = pd.read_csv('data/raw/customers.csv')
# Clean: standardize gender, fix ages, remove duplicates
```

### products.csv
```python
df = pd.read_csv('data/raw/products.csv')
# Clean: fix invalid prices, standardize categories
```

### sellers.csv
```python
df = pd.read_csv('data/raw/sellers.csv')
# Clean: fix invalid ratings, remove duplicates
```

### orders.csv
```python
df = pd.read_csv('data/raw/orders.csv')
# Clean: remove duplicate order_ids, fix logical errors
# This is your main analytical table
```

### payments.csv
```python
df = pd.read_csv('data/raw/payments.csv')
# Clean: remove duplicate payments, validate foreign keys
```

---

## 📊 Portfolio Project Ideas

### 1. **Data Quality Dashboard**
- Automated data quality checks
- Missing value heatmap
- Duplicate detection report
- Outlier analysis
- Export to Power BI

### 2. **Customer Analytics**
- Customer segmentation (RFM analysis)
- Churn prediction
- Lifetime value calculation
- Geographic analysis
- Cohort analysis

### 3. **Sales Analytics**
- Revenue trends and forecasting
- Product performance ranking
- Seller performance comparison
- Order fulfillment analysis
- Discount impact analysis

### 4. **ETL Pipeline**
- Build automated data cleaning scripts
- Create data validation checks
- Implement incremental data loads
- Schedule jobs with Airflow
- Monitor data quality metrics

### 5. **Power BI Dashboard**
- Executive dashboard (KPIs)
- Sales drill-down analysis
- Customer segments
- Product catalog analysis
- Seller performance scorecard

---

## 🔧 Technical Stack

This project works with:
- **Python**: Pandas, NumPy, Faker
- **SQL**: PostgreSQL, SQL queries
- **BI Tools**: Power BI, Tableau, Google Data Studio
- **Notebooks**: Jupyter, Google Colab
- **Version Control**: Git, GitHub

---

## 📚 Learning Outcomes

After completing this project, you'll master:

✅ Data cleaning and preprocessing
✅ Handling missing values and duplicates
✅ SQL queries and database design
✅ Exploratory Data Analysis (EDA)
✅ Data validation and quality checks
✅ Python data manipulation (Pandas)
✅ Business metrics and KPIs
✅ Dashboard creation (Power BI)
✅ Real-world data issues and solutions
✅ ETL/ELT best practices

---

## 📖 Documentation

- **DATA_DICTIONARY.md** - Detailed column definitions, data types, and quality issues
- **schema.sql** - PostgreSQL table definitions and data quality views
- **generate_data.py** - Python script that generated this dataset (reproducible)

---

## ⚠️ Important Notes

### Data Characteristics
- **Realistic**: Based on actual e-commerce patterns
- **Messy**: Contains real-world data quality issues
- **Large**: 500K+ orders for meaningful analysis
- **Reproducible**: Fixed seed (SEED=42) for consistency
- **Self-Contained**: No external dependencies needed

### Not Included (By Design)
- ❌ Personal Identifiable Information (PII) - Synthetic names/emails
- ❌ Perfectly clean data - This is intentionally messy
- ❌ Real business data - Fully generated for portfolio use

---

## 🎯 Expected Cleaning Effort

| Phase | Tasks | Time |
|-------|-------|------|
| **Data Loading** | Import CSV, create tables | 10 min |
| **Exploration** | EDA, profiling, quality report | 2-3 hours |
| **Cleaning** | Fix issues, standardize formats | 4-6 hours |
| **Validation** | Verify cleaned data, checks | 1-2 hours |
| **Analysis** | KPIs, segmentation, trends | 3-5 hours |
| **Dashboard** | Power BI visualizations | 2-4 hours |
| **Documentation** | Project writeup, GitHub | 1-2 hours |

**Total Time: 13-23 hours** (realistic for a fresher)

---

## 🤝 Contributing

This dataset is designed for learning. Feel free to:
- Report any issues or bugs
- Suggest additional data quality scenarios
- Create improved cleaning scripts
- Share your portfolio project

---

## 📄 License

This dataset is provided for **educational purposes** and portfolio building. You're free to use, modify, and redistribute it.

---

## 📧 Contact & Support

For questions or issues with this dataset:
1. Check the DATA_DICTIONARY.md for detailed information
2. Review the schema.sql for table structure
3. Consult the PostgreSQL data quality views
4. Check your data cleaning code logic

---

## 🎉 Next Steps

1. ✅ Load data into PostgreSQL
2. ✅ Run data quality queries
3. ✅ Create EDA notebook
4. ✅ Build cleaning pipeline
5. ✅ Generate business metrics
6. ✅ Design Power BI dashboard
7. ✅ Document your process
8. ✅ Push to GitHub
9. ✅ Add to portfolio website
10. ✅ Share in interviews!

---

**Generated**: September 22, 2026
**Seed**: 42 (reproducible)
**Version**: 1.0
**Total Records**: 868,050
**Data Size**: ~128 MB (CSV)

Good luck with your data analytics journey! 🚀
