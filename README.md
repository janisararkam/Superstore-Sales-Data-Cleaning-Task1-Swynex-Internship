# Superstore-Sales-Data-Cleaning-Task1-Swynex-Internship
## Data Cleaning & Preparation

The Superstore Sales dataset was downloaded from **Kaggle** and loaded into **Python using Jupyter Notebook and Pandas** for data inspection, cleaning, and preparation.

The objective of this step was to identify and handle data-quality issues such as missing values, incorrect data types, inconsistent categorical values, invalid IDs, invalid numerical values, date issues, and duplicate records, and to prepare a reliable dataset for further **Exploratory Data Analysis (EDA)** and dashboard development.

---

### 1. Dataset Loading

The dataset was loaded into a Pandas DataFrame using `read_csv()`.

```python
import pandas as pd

df = pd.read_csv("Superstoresalesdatasetfordataanalystinternship.csv")
```

The dataset contains:

* **9,800 rows**
* **18 columns**

---

### 2. Initial Data Inspection

The dataset was initially inspected using `head()`, `tail()`, `shape`, `columns`, `info()`, and `dtypes`.

This helped understand:

* The structure of the dataset
* The number of rows and columns
* Column names
* Data types
* Missing values
* Sample records from the beginning and end of the dataset

```python
df.head(30)
df.tail()
df.shape
df.columns
df.info()
df.dtypes
```

The initial inspection showed that:

* `Order Date` and `Ship Date` were stored as `object`
* `Postal Code` was stored as `float64`
* 11 values were missing in `Postal Code`
* Other columns did not contain missing values

---

# Column-Wise Data Cleaning

### 3. Date Columns — Data Type Correction

The `Order Date` and `Ship Date` columns were initially stored as text/object values.

They were converted into proper Pandas datetime format using the correct date format:

```python
df["Order Date"] = pd.to_datetime(
    df["Order Date"],
    format="%d/%m/%Y"
)

df["Ship Date"] = pd.to_datetime(
    df["Ship Date"],
    format="%d/%m/%Y"
)
```

This makes the date columns suitable for:

* Time-based analysis
* Year/month analysis
* Sorting and filtering
* EDA
* Power BI date-based visualizations

---

### 4. Postal Code — Data Type Correction

`Postal Code` was initially stored as `float64`.

Since postal codes are integer-like values and the column contained missing values, it was converted to Pandas nullable integer type:

```python
df["Postal Code"] = df["Postal Code"].astype("Int64")
```

---

### 5. Missing Values — Postal Code

Missing-value analysis was performed using:

```python
df.isnull().sum()
```

The initial inspection identified **11 missing values in Postal Code**.

These missing values belonged to records from:

* **City:** Burlington
* **State:** Vermont

The missing postal codes were filled with **5401**, representing ZIP code **05401**.

```python
df.loc[
    df["Postal Code"].isna(),
    "Postal Code"
] = 5401
```

The missing-value check was then performed again and showed **0 missing values across all columns**.

---

### 6. Column Name Standardization

Column names were standardized to make them easier to work with in Python and SQL-style analysis.

Spaces and hyphens were replaced with underscores for the relevant columns.

Examples:

* `Row ID` → `Row_ID`
* `Order ID` → `Order_ID`
* `Order Date` → `Order_Date`
* `Ship Date` → `Ship_Date`
* `Ship Mode` → `Ship_Mode`
* `Customer ID` → `Customer_ID`
* `Customer Name` → `Customer_Name`
* `Postal Code` → `Postal_Code`
* `Product ID` → `Product_ID`
* `Sub-Category` → `Sub_Category`
* `Product Name` → `Product_Name`

This produced cleaner and more consistent column names for further analysis.

---

### 7. Categorical Data Validation

The unique values of important categorical columns were checked to identify unexpected or inconsistent values.

The following columns were validated:

```python
[
    "Ship_Mode",
    "Segment",
    "Country",
    "Region",
    "Category",
    "Sub_Category"
]
```

The observed values were checked for consistency.

Examples included:

* Ship Mode: `Second Class`, `Standard Class`, `First Class`, `Same Day`
* Segment: `Consumer`, `Corporate`, `Home Office`
* Country: `United States`
* Region: `South`, `West`, `Central`, `East`
* Category: `Furniture`, `Office Supplies`, `Technology`

No unexpected categorical values requiring correction were identified.

---

### 8. ID Columns — Validation

Important identifier columns were checked for missing or invalid values.

The following columns were validated:

* `Row_ID`
* `Order_ID`
* `Customer_ID`
* `Product_ID`

Checks were performed for:

* Invalid/non-positive `Row_ID`
* Missing `Order_ID`
* Missing `Customer_ID`
* Missing `Product_ID`

Results showed:

* No `Row_ID` values less than or equal to zero
* No missing `Order_ID`
* No missing `Customer_ID`
* No missing `Product_ID`

---

### 9. Postal Code — Validity Check

After handling missing values, postal codes were checked for invalid non-positive values:

```python
print("Postal_Code <= 0:",
      (df["Postal_Code"] <= 0).sum())

print("Postal_Code missing:",
      df["Postal_Code"].isnull().sum())
```

Results:

* Invalid/non-positive postal codes: **0**
* Missing postal codes: **0**

The previously missing Burlington, Vermont records were also validated and all 11 records were associated with Burlington, Vermont and postal code 5401/05401.

---

### 10. Sales — Validity Check

The `Sales` column was checked for zero or negative values:

```python
print("Sales <= 0:",
      (df["Sales"] <= 0).sum())
```

Result:

* Sales values less than or equal to zero: **0**

Therefore, no invalid sales values required removal or correction.

---

### 11. Sales — Outlier Analysis

Statistical outlier analysis was performed on the `Sales` column using the **Interquartile Range (IQR)** method.

```python
Q1 = df["Sales"].quantile(0.25)
Q3 = df["Sales"].quantile(0.75)

IQR = Q3 - Q1

lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR
```

The calculated limits were:

* Lower limit: **-272.7875**
* Upper limit: **500.6405**

A total of **1,145 records** were identified as statistical outliers above the upper IQR limit.

The highest sales records were inspected along with their categories, sub-categories, and product names.

These high-value transactions represented legitimate-looking products and business transactions rather than obvious data-entry errors.

Therefore, the identified IQR outliers were **not removed or modified**.

This preserves genuine high-value sales transactions for further business analysis.

---

### 12. Date Range Validation

The minimum and maximum dates were checked for both date columns:

```python
print(
    "Order Date:",
    df["Order_Date"].min(),
    "to",
    df["Order_Date"].max()
)

print(
    "Ship Date:",
    df["Ship_Date"].min(),
    "to",
    df["Ship_Date"].max()
)
```

Results:

* Order Date: **2015-01-03 to 2018-12-30**
* Ship Date: **2015-01-07 to 2019-01-05**

An additional validation was performed to ensure that no product was shipped before it was ordered:

```python
(df["Ship_Date"] < df["Order_Date"]).sum()
```

Result:

* Ship Date before Order Date: **0**

---

# Row-Wise Data Cleaning & Validation

### 13. Duplicate Record Check

The dataset was checked for exact duplicate rows:

```python
df.duplicated().sum()
```

Result:

* **0 duplicate rows**

Therefore, no duplicate records needed to be removed.

---

### 14. Row ID Uniqueness Check

The uniqueness of `Row_ID` was also validated:

```python
df["Row_ID"].duplicated().sum()
```

Result:

* **0 duplicate Row_ID values**

This confirms that each row has a unique Row ID.

---

### 15. Category and Sub-Category Consistency

The relationship between `Category` and `Sub_Category` was validated using a cross-tabulation:

```python
pd.crosstab(
    df["Category"],
    df["Sub_Category"]
)
```

The results showed consistent category-sub-category relationships.

For example:

* Furniture → Bookcases, Chairs, Furnishings, Tables
* Office Supplies → Appliances, Art, Binders, Envelopes, Fasteners, Labels, Paper, Storage, Supplies
* Technology → Accessories, Copiers, Machines, Phones

No incorrect Category/Sub-Category combinations were identified.

---

# Final Clean Dataset

After completing the data cleaning and validation process, the cleaned dataset was exported as a new CSV file.

```python
df.to_csv(
    "Cleaned_Superstore_cleaned_Data_Set.csv",
    index=False
)
```

The original dataset was preserved separately, while the cleaned version was saved for further analysis.

### Final Dataset Status

The cleaned dataset contains:

* **9,800 rows**
* **18 columns**
* **0 missing values**
* **0 exact duplicate rows**
* **0 duplicate Row_ID values**
* Correct datetime columns
* Standardized column names
* Validated categorical values
* Validated ID columns
* Validated postal codes
* No zero/negative sales
* No impossible Ship Date before Order Date
* Valid Category/Sub-Category relationships
* Legitimate high-value sales transactions retained

The cleaned dataset is therefore **ready for the next stage: Exploratory Data Analysis (EDA)**.

The cleaned dataset will be used in **Excel for EDA** and subsequently in **Power BI for dashboard development**.
