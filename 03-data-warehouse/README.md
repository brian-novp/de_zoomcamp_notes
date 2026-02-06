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
- It is an OLAP solution. Used for reporting and data analysis. See [Data Warehouse](https://en.wikipedia.org/wiki/Data_warehouse).
- Data Analysts make use of the data marts while Data Scientists best to access the warehouse (because the need to build models of ML from half-raw data).  

# BigQuery
- A serverless data warehouse using columnar storage (Colossus) and distributed query engine (Dremel). No servers to manage or databases software to install. This lessen the pain points of configuring and maintaining servers.
- Scalability and high-availability . Organization can start in gigabyte-level data and then easily scale to petabyte-level data volume easily. This is because the decoupled architecture of Colossus (storage), Dremel (compute), along with other proprietary technology and Jupiter Network (1 petabit per sec network). BigQuery uses columnar format called Capacitor and Google's own cluster management system called Borg (predecessor to Kubernetes).
- Built-in features:
    - Machine Learning
    - Geospatial Analysis
    - Business Intelligence

- By default, BigQuery cache result. This means, if we execute the same query, it will produce the same result as the first-time the query ran. BQ done this because it makes the query faster. Disable this cache data to make the data 'live'. Read [BigQuery cached result](https://docs.cloud.google.com/bigquery/docs/cached-results)

## BigQuery Pricing
- On-demand Pricing
By default, queries are billed using the on-demand (per TiB) pricing model, where you pay for the data scanned by your queries.  

With on-demand pricing, you will generally have access to up to 2,000 concurrent slots, shared among all queries in a single project. Periodically, BigQuery will temporarily burst beyond this limit to accelerate smaller queries. In addition, you might have fewer slots available if there is a high amount of contention for capacity in a specific location. 

- Capacity Compute Pricing
BigQuery offers a capacity-based compute pricing model for customers who need additional capacity or prefer a predictable cost for query workloads rather than the on-demand price (per TiB of data processed). The capacity compute model offers pay-as-you-go pricing (with autoscaling) and optional one year and three year commitments that provide discounted prices. You pay for query processing capacity, measured in slots (virtual CPUs) over time.  

More details on [BigQuery Pricing](https://cloud.google.com/bigquery/pricing?hl=en)  


# Important Notes
BigQuery has dry-run feature that lets us know how many data will be scanned for the query that we want to run. It is helpful as first, we know the query cost before we execute the query, second, we can understand whether our table can or need to be optimized. For example:  
- Partition our table based on some columns. Partition here means dividing large tables into smaller partitions. Partitioning can help the query engine to look for the data faster and scanning less data by using the partitioned column in our query (e.g. in `where` clause). Most common example, we make column with time-based or date related or integer-range based as a partition column. This partition is done when ingesting the data, creating the table in BigQuery. More on [partitioning](https://docs.cloud.google.com/bigquery/docs/partitioned-tables)
- Clustering the tables. More on [Clustering](https://docs.cloud.google.com/bigquery/docs/clustered-tables)
- Order of the column clustered is important
- The order of the specified columns determines the sort oder of the data. For example, if we cluster column month_year, location_id, customer_id, then the sort order would be starting from month_year and then location_id and then customer_id. So, be mindful and ask the analysts which column is more likely to be consumed. It depends on their analysis.
- Up to four clustering columns
- Table with data size < 1GB, don't show significant improvement with partitioning and clustering

## Partitioning vs Clustering
|Clustering|Partitioning|
|---|---|
|Cost benefit unknown| Cost known upfront (by looking the dry-run feature)|
|Need more granularity than partitioning alone allows| Need partition-level management (Creating, deleting or moving paretition between storage). Cant do this in clustering|
|Queries commonly use filters or aggregation against multiple particular columns| Filter or aggregate on single column|
|Use clustering if the cardinality of the number of values in a column or group of column is large|Can't do this in clustering because the limitation of 4000 partitions per table|

## When to use Clustering over Partitioning
- If partitioning results in a small amount of data per partition (approx < 1GB)
- If partitioning results in a large number of partitions beyond the limits on partitioned tables
- If partitioning results in your mutation operations modifying the majority of partitions in the table frequently (for example, every few minutes)


# BigQuery Best Practices
DataTalksClub said there are two groups of best practices.

## Cost Optimization
1. Avoid using `SELECT *` . Because it will scan the whole table. Scanning the whole table will incur higher cost. BigQuery is a columnar storage, maximize it by always select the necessary columns with filtering. (Be mindful about query pricing)
2. Try to always use partitioned column in a `WHERE` clause. This will keep the query to scan less data. Therefore optimizing cost.
3. Be mindful if we use streaming insert (why?)
4. 