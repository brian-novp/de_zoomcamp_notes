# OLAP vs OLTP
Online Analytical Processing vs Online Transaction Processing

| | OLTP | OLAP |
|---|--- |--- |
|Purpose | Control and run essential business operations in real-time | Plan, solve problems, support decisions, discover hidden insights. Mainly used by data analyst and data scientist|
|Data Updates| Short, fast updates intiated by user | Periodically refreshed with scheduled, long-running batch jobs (or streaming)|
|Database Design|Normalized databases for efficiency. Normalized means minimizing data redundancy and maximize data integrity. See Third Normal Form for example. |Denormalized databases for analysis|
|Space Requirements|Generally small if historical data is archived|Generally large due to aggregating large datasets|
|Backup and Recovery|Regular backups required to ensure business continuity and meet legal and governance requirements|Lost data can be reloaded from OLTP database as a needed in lieu of regular backups|
|Productivity|Increases productivity of end users|Increases productivity of business managers, data analyst and executives|
|Data View|List day-to-day business transactions|Multi-dimensional view of enterprise data|
|User examples| Customer-facing personnel, clerks, online shoppers|Knowledge worker such as data analysts, business analysts, and executives|

# What is a data warehouse?
It is an OLAP solution. Used for reporting and data analysis. See [Data Warehouse](https://en.wikipedia.org/wiki/Data_warehouse). Data Analysts make use of the data marts while Data Scientists best to access the warehouse (because the need to build models of ML from half-raw data).  

# BigQuery
- A serverless data warehouse using columnar storage (Colossus) and distributed query engine (Dremel). No servers to manage or databases software to install. This lessen the pain points of configuring and maintaining servers.
- Scalability and high-availability . Organization can start in gigabyte-level data and then easily scale to petabyte-level data volume easily. This is because the decoupled architecture of Colossus (storage), Dremel (compute), along with other proprietary technology and Jupiter Network (1 petabit per sec network). BigQuery uses columnar format called Capacitor and Google's own cluster management system called Borg (predecessor to Kubernetes).
- Built-in features:
    - Machine Learning
    - Geospatial Analysis
    - Business Intelligence

- By default, BigQuery cache result. This means, if we execute the same query, it will produce the same result as the first-time the query ran. BQ done this because it makes the query faster. Disable this cache data to make the data 'live'. Read [BigQuery cached result](https://docs.cloud.google.com/bigquery/docs/cached-results)
- 




BigQuery has dry run feature that lets us know how many data will be scanned for the query that we want to run. It is helpful, we can understand whether our table can be optimized. For example
1. Partition our table based on some columns. Partitioning can help the query engine to look for the data faster and simpler by using the partitioned column in our query.

# BigQuery Best Practices
DataTalksClub said there are two groups of best practices.

## Cost Optimization
1. Avoid using `SELECT *` . Because it will scan the whole table. Scanning the whole table will incur higher cost. BigQuery is a columnar storage, maximize it by always select the necessary columns with filtering. (Be mindful about query pricing)
2. Try to always use partitioned column in a `WHERE` clause. This will keep the query to scan less data. Therefore optimizing cost.
3. Be mindful if we use streaming insert (why?)
4. 