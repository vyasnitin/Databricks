# Databricks

"Tell me about a performance optimization you implemented in Databricks."

You:

"In one of my Databricks pipelines, we had a large Delta table containing transaction data. Initially the downstream reporting queries were performing well, but as the data volume increased, query performance degraded significantly.

I first investigated the issue instead of immediately applying optimization. I looked at the Spark UI, query execution plan, number of files, input data volume, shuffle and task execution times.

One of the major issues I identified was a large number of small files created by frequent incremental loads. Because the pipeline was continuously ingesting smaller batches, the Delta table accumulated many small files.

We addressed this using Delta file compaction with OPTIMIZE. For a large table, I would avoid unnecessarily optimizing the entire table and target the relevant data based on the workload.

I also reviewed the table's data-access pattern. Since most queries were filtering on transaction date and sometimes customer attributes, I evaluated the appropriate data-layout strategy to improve data skipping.

For joins, I checked the size of the datasets involved. Where a dimension table was sufficiently small, I considered a broadcast join to reduce shuffle. I also checked for data skew because a highly skewed join key can result in a few long-running tasks.

At the query level, we reduced unnecessary columns, pushed filters as early as possible and avoided unnecessary transformations.

Finally, I validated the changes using the Spark UI and query execution metrics rather than assuming the optimization had worked.

So my optimization approach is generally: identify the bottleneck first, select the appropriate optimization technique, implement it, and then measure the result."
