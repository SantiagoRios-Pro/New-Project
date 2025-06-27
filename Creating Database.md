## 🧱 Database Schema: `invest`

This section documents the SQL schema and purpose of each table used in the `invest` database.

---

```sql
CREATE DATABASE invest;
USE invest;
🟢 account_dim
sql
Copy
Edit
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

account_id (PK): Unique identifier for the account.

opt_38: Optional account flag or classification.

opt38_desc: Description of the opt_38 code.

client_id: Reference to the client who owns the account.

main_account: Indicates if this is the primary account.

acct_open_date: The date the account was opened.

acct_open_status: Status code indicating if the account is active, closed, etc.

🟢 customer_details
```sql
Copy
Edit
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

customer_id (PK): Unique ID for the customer.

full_name, first_name, last_name: Name fields (stored redundantly).

email: Email address.

customer_location: Geographical or market location of the customer.

🟢 holdings_current
```sql
Copy
Edit
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

row_names (PK): Internal row ID.

account_id: Links to account_dim.

ticker: The security's ticker symbol.

date: Date of the holding record.

variable: Financial metric or classification.

value: Metric value (e.g., market value).

price_type: Type of price used (e.g., adjusted, closing).

quantity: Number of units held.

🟢 pricing_daily_new
```sql
Copy
Edit
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

row_names (PK): Internal row ID.

date: The trading date.

variable: Type of financial variable (e.g., close, volume).

value: Value of the variable.

ticker: Ticker symbol.

price_type: Indicates if price is raw, adjusted, etc.

🟢 security_masterlist
```sql
Copy
Edit
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

ticker (PK): Unique ticker symbol for the security.

security_name: Full name of the security.

sp500_weight: Weight in the S&P 500 (if applicable).

sec_type: Type of security (e.g., stock, bond).

major_asset_class: High-level asset classification.

minor_asset_class: More granular asset classification.
