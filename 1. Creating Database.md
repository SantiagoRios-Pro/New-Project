# 🧱 Database Schema: `invest`

This section of the project outlines the SQL code used to create the invest database, along with the purpose of each table. Below each table definition, you’ll find a data dictionary that explains the meaning of each attribute.

---

```sql
CREATE DATABASE invest; -- This is going to be our database for this project
USE invest;
```
## Table: account_dim
```sql
CREATE TABLE account_dim (
    row_names INT,
    account_id INT PRIMARY KEY,
    opt_38 INT,
    opt38_desc VARCHAR(50),
    client_id INT,
    main_account INT,
    acct_open_date DATE,
    acct_open_status INT
);
```
Description:
Stores account-level information for each customer. Each row represents an individual account and its associated metadata.

* row_names: A temporary or internal identifier used during data import or processing.
* account_id (PK): Unique identifier for the account.
* opt_38: Optional account flag or classification.
* opt38_desc: Description of the opt_38 code.
* client_id: Reference to the client who owns the account.
* main_account: Indicates if this is the primary account.
* acct_open_date: The date the account was opened.
* acct_open_status: Status code indicating if the account is active, closed, etc.

## Table: customer_details
```sql
CREATE TABLE customer_details (
    row_names VARCHAR(250),
    customer_id INT PRIMARY KEY,
    full_name VARCHAR(250),
    first_name VARCHAR(250),
    last_name VARCHAR(250),
    email VARCHAR(250),
    customer_location VARCHAR(250)
);
```
Description:
Contains personal and contact information for each customer.

* row_names: A temporary or internal identifier used during data import or processing.
* customer_id: A unique integer that serves as the primary key for each customer.
* full_name: The customer’s full name, potentially redundant if first and last names are also used.
* first_name: The customer's first name.
* last_name: The customer's last name.
* email: The customer's email address.
* customer_location: The location or region where the customer is based.

## Table: holdings_current
```sql
CREATE TABLE holdings_current (
    row_names INT PRIMARY KEY,
    account_id INT,
    ticker VARCHAR(8),
    date DATE,
    variable VARCHAR(250),
    value DOUBLE,
    price_type VARCHAR(250),
    quantity DOUBLE
);
```
Description:
Captures current holdings per account and ticker, including position details and pricing type.

* row_names: A temporary or internal identifier used during data import or processing.
* account_id: Links to account_dim.
* ticker: Ticker symbol of the security being held (e.g., AAPL, SLV).
* date: The date the holding information was recorded.
* variable: Identifies the security and its price adjustment type (e.g., SLV.Adjusted).
* value: The monetary value or market value of the holding.
* price_type: Indicates how the price was calculated (e.g., Adjusted, Close).
* quantity: Number of shares or units held for that specific security.

## Table: pricing_daily_new
```sql
CREATE TABLE pricing_daily_new (
    row_names INT PRIMARY KEY,
    date DATE,
    variable VARCHAR(100),
    value DOUBLE,
    ticker VARCHAR(8),
    price_type VARCHAR(100)
);
```
Description:
Contains daily pricing information for securities, possibly for multiple variables (e.g., price, volume).

* row_names: A temporary or internal identifier used during data import or processing.
* date: The trading date associated with the price data.
* variable: Specifies the type of metric (e.g., adjusted close, volume).
* value: Numerical value of the specified metric for a given ticker and date.
* ticker: Ticker symbol representing the security.
* price_type: Indicates how the price is derived (e.g., raw, adjusted).

## Table: security_masterlist
```sql
CREATE TABLE security_masterlist (
    row_names INT,
    id BIGINT,
    security_name TEXT,
    ticker VARCHAR(8) PRIMARY KEY,
    sp500_weight DOUBLE,
    sec_type VARCHAR(100),
    major_asset_class VARCHAR(100),
    minor_asset_class VARCHAR(100)
);
```
Description:
Reference table listing all known securities with classification details.

* row_names: A temporary or internal identifier used during data import or processing.
* id: Unique identifier for the security (may differ from ticker).
* security_name: Full name or description of the security.
* ticker: Unique ticker symbol used to identify the security (Primary Key).
* sp500_weight: Security's weight in the S&P 500 index, if applicable.
* sec_type: Type of security (e.g., equity, bond, ETF).
* major_asset_class: Broad classification category (e.g., equities, fixed income).
* minor_asset_class: More specific sub-category within the major asset class (e.g., large-cap, municipal bonds).
