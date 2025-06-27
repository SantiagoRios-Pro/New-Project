# 🛠️ Data Upload to MySQL Using Command Line

To demonstrate the use of command line to upload and manage data in a database, I have uploaded the [`invest_data.zip`](https://github.com/SantiagoRios-Pro/Tableau-Projects/blob/main/invest_data.zip) 
datasets into the 'invest' MySQL database using **PowerShell**, **Command Prompt**, and native **SQL commands**.

This process involved converting the CSV files to UTF-8 format to ensure compatibility, then loading them into the database
using `LOAD DATA LOCAL INFILE`. The steps below outline the full procedure I went through from preparation to upload.

---

## 📚 Table of Contents

- [Part A – PowerShell: Converting CSV Files to UTF-8 Format](#part-a--powershell-converting-csv-files-to-utf-8-format)
- [Part B – Command Prompt: Uploading UTF-8 Files to MySQL](#part-b--command-prompt-uploading-utf-8-files-to-mysql)
- [Part C - Why upload data to MySQL using the command line?](#️-why-upload-data-to-mysql-using-the-command-line)

---

## Part A – PowerShell: Converting CSV Files to UTF-8 Format

1. Open **PowerShell**.
2. Navigate to the folder where the CSV files are stored using:
   ```powershell
   cd "C:\Your\Path\To\Files"
3. Type dir to verify that the desired files are listed in the directory.

4. Confirm the presence of the file(s) to convert (e.g., holdings_current.csv).

5. Convert the file to UTF-8 using the following command:

   ```powershell
   import-csv "holdings_current.csv" | export-csv "holdings_current_bis.csv" -NoTypeInformation -Encoding UTF8
   ```
  This creates a new UTF-8 encoded version of the file in the same directory, ready for upload.

## Part B – Command Prompt: Uploading UTF-8 Files to MySQL

1. Open Command Prompt.

2. Navigate to the directory where the UTF-8 files are saved:

```cmd
cd "C:\Your\Path\To\Files"
```
3. Use dir to confirm that the converted .csv files are in the folder.

4. Ensure your MySQL tables are already created with the correct column order and data types matching your CSV files.

5. Thorugh command prompt, log in to your MySQL server using:

```cmd
mysql -u your_user -p -h your_host.mysql.database.azure.com --local-infile=1
```
6. Inside MySQL, check that local file upload is enabled:

```sql
SHOW VARIABLES LIKE 'local_infile';
```
7. Upload each UTF-8 CSV file using the following SQL command:

```sql
LOAD DATA LOCAL INFILE 'holdings_current_bis.csv'
INTO TABLE holdings_current
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES;
```
If there are any issues (e.g., data type mismatches), MySQL will return error messages. Make adjustments as needed to either your table structure or CSV format.

8. Verify the upload by running:

```sql
SELECT * FROM holdings_current LIMIT 10;
```
If no warnings or errors are returned, your data is now successfully stored in the invest database and ready for analysis.

## Why upload data to MySQL using the command line?

Uploading data through the command line may seem tedious/technical at first, but it offers some big advantages:

- **Speed**: It’s much faster than manual uploads, especially with large datasets.
- **Efficiency**: You can import thousands of rows with just one command.
- **Control**: You decide exactly how the data is formatted and imported.
- **Automation**: Once set up, the process is easy to repeat for other files or projects.
- **Professional Practice**: This is how data engineers and analysts work with real-world databases, especially when
    using cloud platforms like MySQL on Azure.

Overall, it's a reliable, scalable, and a professional way to manage data uploads.
