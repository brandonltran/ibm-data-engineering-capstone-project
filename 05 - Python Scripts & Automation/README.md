# Python Scripts & Automation
> SoftCart needs to keep data synchronized between different databases/data warehouses as a part of our daily routine. One task that is routinely performed is the sync up of staging data warehouse and production data warehouse. Write a script that will automate the process of regularly updating the DB2 instance with new records from MySQL.

## 1. Connect to MySQL Database with Python

First I will install the `mysql-connector-python` dependency using pip. This assignment requires version `8.0.31`
```console
python3 -m pip install mysql-connector-python==8.0.31
```
> ```
> Successfully installed mysql-connector-python-8.0.31 protobuf-3.19.6
> ```

Now it's time to connect to our MySQL database `sales` using Python. I will create a new file `mysqlconnect.py`. The main script is already provided, I just need to modify the credentials to connect to our database. The script does the following:
- Use `mysql.connector` to connect to the database `sales`
- Create a table for `products`
- Insert product data
- Output the table records for confirmation
```python
# This program requires the python module mysql-connector-python to be installed.
# Install it using the below command
# pip3 install mysql-connector-python

import mysql.connector
username = 'root'
pw = 'PW'
host = '127.0.0.1'
db = 'sales'

# connect to database
connection = mysql.connector.connect(user=username, password=pw,host=host,database=db)

# create cursor

cursor = connection.cursor()

# create table

SQL = """CREATE TABLE IF NOT EXISTS products(

rowid int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
product varchar(255) NOT NULL,
category varchar(255) NOT NULL

)"""

cursor.execute(SQL)

print("Table created")

# insert data

SQL = """INSERT INTO products(product,category)
	 VALUES
	 ("Television","Electronics"),
	 ("Laptop","Electronics"),
	 ("Mobile","Electronics")
	 """

cursor.execute(SQL)
connection.commit()


# query data

SQL = "SELECT * FROM products"

cursor.execute(SQL)

for row in cursor.fetchall():
	print(row)

# close connection
connection.close()
```

Now I will run the script using the command line.
```console
python3 mysqlconnect.py
```

> ```
> Table created
> (1, 'Television', 'Electronics')
> (2, 'Laptop', 'Electronics')
> (3, 'Mobile', 'Electronics')
> ```

## 2. Connect to IBM DB2 with Python

First I will install the `ibm-db` dependency using pip.
```console
python3 -m pip install ibm-db
```
> ```
> Successfully installed ibm-db-3.1.4
> ```

Now I will create a new file `db2connect.py` containing the script provided. Again, I just need to modify the credentials to connect to our IBM DB2 instance. The script does the following:
- Use `ibm_db` to connect to our IBM DB2 instance
- Create a table for `products`
- Insert product data
- Output the table records for confirmation

```python
# This program requires the python module ibm-db to be installed.
# Install it using the below command
# pip3 install ibm-db

import ibm_db

# connectction details

dsn_hostname = "6667d8e9-9d4d-4ccb-ba32-21da3bb5aafc.c1ogj3sd0tgtu0lqde00.databases.appdomain.cloud" # e.g.: "dashdb-txn-sbox-yp-dal09-04.services.dal.bluemix.net"
dsn_uid = "prx70334"        # e.g. "abc12345"
dsn_pwd = "PWD"      # e.g. "7dBZ3wWt9XN6$o0J"
dsn_port = "30376"                # e.g. "50000" 
dsn_database = "bludb"            # i.e. "BLUDB"
dsn_driver = "{IBM DB2 ODBC DRIVER}" # i.e. "{IBM DB2 ODBC DRIVER}"           
dsn_protocol = "TCPIP"            # i.e. "TCPIP"
dsn_security = "SSL"              # i.e. "SSL"

#Create the dsn connection string
dsn = (
    "DRIVER={0};"
    "DATABASE={1};"
    "HOSTNAME={2};"
    "PORT={3};"
    "PROTOCOL={4};"
    "UID={5};"
    "PWD={6};"
    "SECURITY={7};").format(dsn_driver, dsn_database, dsn_hostname, dsn_port, dsn_protocol, dsn_uid, dsn_pwd, dsn_security)

# create connection
conn = ibm_db.connect(dsn, "", "")
print ("Connected to database: ", dsn_database, "as user: ", dsn_uid, "on host: ", dsn_hostname)

# create table
SQL = """CREATE TABLE IF NOT EXISTS products(rowid INTEGER PRIMARY KEY NOT NULL,product varchar(255) NOT NULL,category varchar(255) NOT NULL)"""

create_table = ibm_db.exec_immediate(conn, SQL)


print("Table created")

# insert data

SQL = "INSERT INTO products(rowid,product,category)  VALUES(?,?,?);"
stmt = ibm_db.prepare(conn, SQL)
row1 = (1,'Television','Electronics')
ibm_db.execute(stmt, row1)

row2 = (2,'Laptop','Electronics')
ibm_db.execute(stmt, row2)

row3 = (3,'Mobile','Electronics')
ibm_db.execute(stmt, row3)


# query data

SQL="SELECT * FROM products"
stmt = ibm_db.exec_immediate(conn, SQL)
tuple = ibm_db.fetch_tuple(stmt)
while tuple != False:
    print (tuple)
    tuple = ibm_db.fetch_tuple(stmt)
# export the fields name and type from collection test into the file data.csv



# close connection
ibm_db.close(conn)
```
> ```
> Connected to database:  bludb as user:  prx70334 on host:  6667d8e9-9d4d-4ccb-ba32-21da3bb5aafc.c1ogj3sd0tgtu0lqde00.databases.appdomain.cloud
> Table created
> (1, 'Television', 'Electronics')
> (2, 'Laptop', 'Electronics')
> (3, 'Mobile', 'Electronics')
> ```

## 3. Modify `sales_data` Table

By default, the price and timestamp columns in our `sales_data` table have nullable records. I will run the following statement to fix this:

```sql
ALTER TABLE sales_data
ALTER COLUMN timestamp SET DATA TYPE TIMESTAMP;

ALTER TABLE sales_data
ALTER COLUMN timestamp SET NOT NULL;

ALTER TABLE sales_data
ALTER COLUMN timestamp SET DEFAULT CURRENT_TIMESTAMP;

ALTER TABLE sales_data
ALTER COLUMN price SET NOT NULL;

ALTER TABLE sales_data
ALTER COLUMN price SET DEFAULT 0;
```

## 4. Automate Syncronization Between MySQL and DB2

Using the methods above, I can write a script `automation.py` that automates the syncronization process between our MySQL table and IBM DB2 instance. The script will do the following:
- Use `mysql.connector` and `ibm_db` to connect to MySQL and IBM DB2 respectively
- Return the `rowid` value of the last column in DB2 as `last_row_id`
- Return a list of all records from MySQL whose `rowid` is greater than `last_row_id` as `new_records`
- Insert `new_records` into the DB2 instance
```python
# Import libraries required for connecting to mysqlimport mysql.connector
import mysql.connector

# Import libraries required for connecting to DB2
import ibm_db

# Connect to MySQL
username = 'root'
pw = 'PW'
host = '127.0.0.1'
db = 'sales'
connection = mysql.connector.connect(user=username, password=pw,host=host,database=db)
cursor = connection.cursor()

# Connect to DB2
dsn_hostname = "6667d8e9-9d4d-4ccb-ba32-21da3bb5aafc.c1ogj3sd0tgtu0lqde00.databases.appdomain.cloud" # e.g.: "dashdb-txn-sbox-yp-dal09-04.services.dal.bluemix.net"
dsn_uid = "prx70334"        # e.g. "abc12345"
dsn_pwd = "PW"      # e.g. "7dBZ3wWt9XN6$o0J"
dsn_port = "30376"                # e.g. "50000" 
dsn_database = "bludb"            # i.e. "BLUDB"
dsn_driver = "{IBM DB2 ODBC DRIVER}" # i.e. "{IBM DB2 ODBC DRIVER}"           
dsn_protocol = "TCPIP"            # i.e. "TCPIP"
dsn_security = "SSL"              # i.e. "SSL"
dsn = (
    "DRIVER={0};"
    "DATABASE={1};"
    "HOSTNAME={2};"
    "PORT={3};"
    "PROTOCOL={4};"
    "UID={5};"
    "PWD={6};"
    "SECURITY={7};").format(dsn_driver, dsn_database, dsn_hostname, dsn_port, dsn_protocol, dsn_uid, dsn_pwd, dsn_security)
conn = ibm_db.connect(dsn, "", "")
print ("Connected to database: ", dsn_database, "as user: ", dsn_uid, "on host: ", dsn_hostname)

# Find out the last rowid from DB2 data warehouse
# The function get_last_rowid must return the last rowid of the table sales_data on the IBM DB2 database.

def get_last_rowid():
    last_rowid_sql = 'SELECT rowid FROM sales_data ORDER BY rowid DESC LIMIT 1'
    last_rowid_stmt = ibm_db.exec_immediate(conn, last_rowid_sql)
    while ibm_db.fetch_row(last_rowid_stmt) != False:
        return ibm_db.result(last_rowid_stmt, 0)


last_row_id = get_last_rowid()
print("Last row id on production datawarehouse = ", last_row_id)

# List out all records in MySQL database with rowid greater than the one on the Data warehouse
# The function get_latest_records must return a list of all records that have a rowid greater than the last_row_id in the sales_data table in the sales database on the MySQL staging data warehouse.

def get_latest_records(rowid):
    latest_records_sql = f'SELECT * FROM sales_data WHERE rowid > {last_row_id} ORDER BY rowid ASC'
    cursor.execute(latest_records_sql)
    return cursor.fetchall()

new_records = get_latest_records(last_row_id)

print("New rows on staging datawarehouse = ", len(new_records))

# Insert the additional records from MySQL into DB2 data warehouse.
# The function insert_records must insert all the records passed to it into the sales_data table in IBM DB2 database.

def insert_records(records):
    insert_sql = "INSERT INTO sales_data (rowid, product_id, customer_id, quantity) VALUES (?,?,?,?);"
    insert_stmt = ibm_db.prepare(conn, insert_sql)
    for row in records:
        ibm_db.execute(insert_stmt, row)

insert_records(new_records)
print("New rows inserted into production datawarehouse = ", len(new_records))

# disconnect from mysql warehouse
connection.close()

# disconnect from DB2 data warehouse
ibm_db.close(conn)

# End of program
```

Before I can call this script, I need to call a statement on our DB2 instance to avoid `Exception Error: Code "7" SQLSTATE=57007 SQLCODE=-668`. More info here: https://www.ibm.com/docs/en/db2-for-zos/11?topic=codes-668
```sql
call sysproc.admin_cmd('reorg table PRX70334.SALES_DATA');
```

Now I can run `automation.py` and test the syncronization of our data.
```console
python3 automation.py
```
> ```
> Connected to database:  bludb as user:  prx70334 on host:  6667d8e9-9d4d-4ccb-ba32-21da3bb5aafc.c1ogj3sd0tgtu0lqde00.databases.appdomain.cloud
> Last row id on production datawarehouse =  12289
> New rows on staging datawarehouse =  1650
> New rows inserted into production datawarehouse =  1650
> ```

The console outputs are returned as expected, and after inspecting the DB2 instance data, I can confirm the syncronization has succeeded.

## About This Lab
##### Environment/IDE
To complete this lab, we will be using the Cloud IDE based on Theia and MySQL database running in a Docker container. You will also need an instance of DB2 running in IBM Cloud

##### Tools/Software
- MySQL Server
- IBM DB2 Database on IBM Cloud

[<kbd> <br> ← Previous Assignment <br> </kbd>](/04%20-%20IBM%20Db2%20Production%20Data%20Warehouse)
[<kbd> <br> → Next Assignment <br> </kbd>](/06%20-%20Apache%20Airflow%20ETL%20&%20Data%20Pipelines)
