# Databricks Small Files Problem

## 1. How are small files generated in real-time scenarios?

Small files are usually generated when data is written frequently in small batches.

### Example

Suppose a pipeline runs every 15 minutes:

```text
10:00 AM → 20 MB
10:15 AM → 15 MB
10:30 AM → 25 MB
10:45 AM → 18 MB
```

Each pipeline execution writes new Delta files.

Typical flow:

```text
Source
   ↓
ADF
   ↓
Databricks
   ↓
Delta Silver Table
```

Example write:

```python
df.write \
  .format("delta") \
  .mode("append") \
  .saveAsTable("silver.sales")
```

After many runs, the Delta table may contain:

```text
sales/
  part-0001.parquet   5 MB
  part-0002.parquet   8 MB
  part-0003.parquet   3 MB
  part-0004.parquet   12 MB
  ...
```

This is called the **small files problem**.

---

## Common Reasons for Small Files

### 1. Frequent incremental loads

If a pipeline runs every 5, 10, or 15 minutes, every run may generate new files.

Example:

```text
Every 15 minutes → small batch → new Delta files
```

Over time, thousands of small files can be created.

---

### 2. Too many Spark partitions

Suppose the input data is only 1 GB, but Spark is processing it with 200 partitions.

```python
df.rdd.getNumPartitions()
```

Output:

```text
200
```

Spark may write many output files.

A large number of Spark partitions for a small amount of data can create excessive small files.

---

### 3. Over-partitioning the Delta table

Example:

```python
.partitionBy("country", "state", "city", "date")
```

This can create too many partition combinations.

Example:

```text
country=India/state=MP/city=Indore/date=2026-10-25/
    2 MB file

country=India/state=Maharashtra/city=Pune/date=2026-10-25/
    3 MB file
```

If each partition contains very little data, the table will have many small files.

This is a common architecture problem.

---

### 4. Frequent MERGE / UPDATE operations

Example:

```sql
MERGE INTO silver.sales
USING updates
ON sales.id = updates.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *
```

Frequent MERGE operations can rewrite files and contribute to file fragmentation.

---

### 5. Streaming or micro-batch ingestion

Typical flow:

```text
Kafka / Event Hub
      ↓
Databricks Structured Streaming
      ↓
Delta Table
```

If every micro-batch contains only a small amount of data, frequent commits can create many small files.

---

# Impact of Small Files

Small files can cause:

- High metadata overhead
- More file-open operations
- Slower query performance
- Slower joins
- Slower MERGE operations
- More Spark scheduling overhead
- Increased cloud storage request overhead

---

# Interview Answer

> Small files are generally created because of frequent incremental or streaming writes, too many Spark partitions, or over-partitioning of Delta tables. For example, if a pipeline runs
