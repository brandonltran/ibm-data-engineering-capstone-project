# IBM Db2 Production Data Warehouse
> Using the adjusted schema design, create an instance of IBM DB2 and load the sample datasets into their respective tables. Write aggregation queries and create a Materialized Query Table for future reports.

## 1. Prepare an Instance of IBM DB2
First I will create the following tables and load their respective datasets.
- `DimDate`
- `DimCategory`
- `DimCountry`
- `DimItem`
- `FactSales`

## 2. Aggregation Queries
The first aggregation query will be a `GROUPING SETS` query using the columns `country`, `category`, and `totalsales`
```sql
SELECT
	category,
	country,
	sum(amount) as totalsales

FROM
	factsales as sales
	
INNER JOIN dimcategory as cat
	ON cat.categoryid = sales.categoryid

INNER JOIN dimcountry as country
	ON country.countryid = sales.countryid

GROUP BY
	GROUPING SETS (
		(cat.category, country.country),
		(cat.category),
		(country.country),
		()
	)

ORDER BY
	country.country
```
This query produces the following output (first 10 rows). The full output can be viewed here: [`grouping_sets_data.csv`](grouping_sets_data.csv)
| CATEGORY    | COUNTRY              | TOTALSALES |
|-------------|----------------------|------------|
|             | Argentina            | 21755581   |
| Books       | Argentina            | 4285010    |
| Electronics | Argentina            | 4338757    |
| Software    | Argentina            | 4292153    |
| Sports      | Argentina            | 4450354    |
| Toys        | Argentina            | 4389307    |
|             | Australia            | 21522004   |
| Books       | Australia            | 4340188    |
| Electronics | Australia            | 4194740    |
| Software    | Australia            | 4410360    |

Next I will write a `ROLLUP` query using the columns `year`, `country`, and `totalsales`
```sql
SELECT
	year,
	country,
	sum(amount) as totalsales

FROM
	factsales as sales
	
INNER JOIN dimdate
	ON dimdate.dateid = sales.dateid

INNER JOIN dimcountry as country
	ON country.countryid = sales.countryid

GROUP BY
	ROLLUP (
		dimdate.year,
		country.country
	)

ORDER BY
	dimdate.year
```
This query produces the following output (first 10 rows). The full output can be viewed here: [`rollup_data.csv`](rollup_data.csv)
| YEAR | COUNTRY              | TOTALSALES |
|------|----------------------|------------|
| 2019 |                      | 399729036  |
| 2019 | Argentina            | 7163167    |
| 2019 | Australia            | 7259016    |
| 2019 | Austria              | 7320233    |
| 2019 | Azerbaijan           | 7097729    |
| 2019 | Belgium              | 7093441    |
| 2019 | Brazil               | 7116253    |
| 2019 | Bulgaria             | 7312912    |
| 2019 | Canada               | 7158271    |
| 2019 | Cyprus               | 7250287    |

Finally, I will write a `CUBE` query using the columns `year`, `country`, and `averagesales`
```sql
SELECT
	year,
	country,
	avg(amount) as averagesales

FROM
	factsales as sales
	
INNER JOIN dimdate
	ON dimdate.dateid = sales.dateid

INNER JOIN dimcountry as country
	ON country.countryid = sales.countryid

GROUP BY
	CUBE (
		dimdate.year,
		country.country
	)

ORDER BY
	dimdate.year,
	country.country
```
This query produces the following output (first 10 rows). The full output can be viewed here: [`cube_data.csv`](cube_data.csv)
| YEAR | COUNTRY              | AVERAGESALES |
|------|----------------------|--------------|
| 2019 | Argentina            | 4017         |
| 2019 | Australia            | 4066         |
| 2019 | Austria              | 4078         |
| 2019 | Azerbaijan           | 3967         |
| 2019 | Belgium              | 3978         |
| 2019 | Brazil               | 4025         |
| 2019 | Bulgaria             | 4087         |
| 2019 | Canada               | 4005         |
| 2019 | Cyprus               | 4045         |
| 2019 | Czech Republic       | 3979         |

## 3. Materialized Query Table (MQT)
I will create an MQT named `total_sales_per_country` based on the columns `country` and `totalsales`
```sql
CREATE TABLE total_sales_per_country (total_sales, country) AS (
    SELECT sum(amount), country
    FROM FactSales
    LEFT JOIN DimCountry
    ON FactSales.countryid = DimCountry.countryid
    GROUP BY (), country
)
DATA INITIALLY DEFERRED
REFRESH DEFERRED
MAINTAINED BY SYSTEM;
```
Because of the configurations `DATA INITIALLY DEFERRED` and `REFRESH DEFERRED`, our MQT will not contain any data initially. To populate it, I can run this simple SQL statement:
```sql
REFRESH TABLE total_sales_per_country;
```
Now I will display the contents of the table to confirm the population of the data.
```sql
SELECT * FROM total_sales_per_country;
```
This produces the following output:

| TOTAL_SALES | COUNTRY              |
|-------------|----------------------|
| 21755581    | Argentina            |
| 21522004    | Australia            |
| 21365726    | Austria              |
| 21325766    | Azerbaijan           |
| 21498249    | Belgium              |
| 21350771    | Brazil               |
| 21410716    | Bulgaria             |
| 21575438    | Canada               |
| 21500526    | Cyprus               |
| 21334142    | Czech Republic       |
| 21331097    | Denmark              |
| 21379967    | Egypt                |
| 21493054    | Estonia              |
| 21336188    | Finland              |
| 21341055    | France               |
| 21301931    | Germany              |
| 21412393    | Greece               |
| 21338015    | Hungary              |
| 21330883    | India                |
| 21498960    | Indonesia            |
| 21095982    | Ireland              |
| 21581548    | Israel               |
| 21844913    | Italy                |
| 21765341    | Japan                |
| 21429604    | Jordan               |
| 21452597    | Malaysia             |
| 21455404    | Mexico               |
| 21383019    | Netherlands          |
| 21329461    | New Zealand          |
| 21544439    | Norway               |
| 21433121    | Oman                 |
| 21704462    | Peru                 |
| 21651762    | Philippines          |
| 21845131    | Poland               |
| 21554611    | Portugal             |
| 21473762    | Qatar                |
| 21738876    | Russia               |
| 21326437    | Saudi Arabia         |
| 21534966    | Singapore            |
| 21394755    | South Africa         |
| 21598368    | South Korea          |
| 21465342    | Spain                |
| 21415934    | Sudan                |
| 21000525    | Sweden               |
| 21526097    | Switzerland          |
| 21060208    | Taiwan               |
| 21517513    | Tajikistan           |
| 21848776    | Thailand             |
| 21270737    | Turkey               |
| 21474740    | Ukraine              |
| 21444221    | United Arab Emirates |
| 21415113    | United Kingdom       |
| 21007997    | United States        |
| 21321382    | Uruguay              |
| 21416080    | Uzbekistan           |
| 21280572    | Vietnam              |

## About This Lab
##### Environment/IDE
This portion of the project will be using the Cloud IDE based on Theia and an instance of DB2 running in IBM Cloud.

##### Tools/Software
- Cloud instance of IBM DB2 database

[<kbd> <br> ← Previous Assignment <br> </kbd>](/03%20-%20PostgreSQL%20Staging%20Data%20Warehouse)
[<kbd> <br> → Next Assignment <br> </kbd>](/05%20-%20Python%20Scripts%20%26%20Automation)
