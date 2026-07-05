# Data Preparation & Cleaning

## Data Exploration

### DataCo - Supply Chain Dataset

- **Customer Email** : Anonymized and Unsuable (e.g. XXXXXXX) --> Remove Column
- **Customer Password** : Anonymized and Unsuable (e.g. XXXXXXX) --> Remove Column
- **Customer State** : Mainly US (CA, MI, NY) but few values with Numbers (e.g. 91732, 95758)
- **Customer Lname** : Some Missing Values --> How to handle
- **Customer Zip Code** : Some Missing Values --> How to handle
- **order date (DateOrders)** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd
- **Order Zipcode** : Some Missing Values --> How to handle
- **Product Description** : Only Blanks --> Remove Column
- **Product Status** : Only O Values, adding no information --> Remove Column
- **shipping date (DateOrders)** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd

```sql
```

### DataCo - Tokenized Access Logs

- **Date** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd

## Data Cleaning

When we are data cleaning we usually follow a few steps
1. check for duplicates and remove any
2. standardize data and fix errors
3. Look at null values 
4. remove any columns and rows that are not necessary - few ways

```sql
```
