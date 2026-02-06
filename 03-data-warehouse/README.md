# OLAP vs OLTP
Online Analytical Processing vs Online Transaction Processing

| | OLTP | OLAP |
|---|--- |--- |
|Purpose | Control and run essential business operations in real-time | Plan, solve problems, support decisions, discover hidden insights|
|Data Updates| Short, fast updates intiated by user | Periodically refreshed with scheduled, long-running batch jobs (or streaming)|

BigQuery has dry run feature that lets us know how many data will be scanned for the query that we want to run. It is helpful, we can understand whether our table can be optimized. For example
1. Partition our table based on some columns. Partitioning can help the query engine to look for the data faster and simpler by using the partitioned column in our query.

# BigQuery Best Practices
DataTalksClub said there are two groups of best practices.

## Cost Optimization
1. Avoid using `SELECT *` . Because it will scan the whole table. Scanning the whole table will incur higher cost. BigQuery is a columnar storage, maximize it by always select the necessary columns with filtering. (Be mindful about query pricing)
2. Try to always use partitioned column in a `WHERE` clause. This will keep the query to scan less data. Therefore optimizing cost.
3. Be mindful if we use streaming insert (why?)
4. 