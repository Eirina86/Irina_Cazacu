# Lab 01: Analyse data with Apache Spark in Microsoft Fabric

Hands-on lab from the DP-700 learning path, run in my own Fabric trial workspace.

**What it does:** loads three years of sales order CSVs into a Spark DataFrame with an explicit schema, transforms them (Year and Month columns, split customer names, column clean-up), saves the result as Parquet, partitions it by Year and Month, and queries it with Spark SQL and charts.

**Problems I hit and fixed:**
- The CSVs have no header row, so `header=true` turned the first order into column names and broke `display()`. Fixed by reading with `header=false` and defining a schema.
- The lab's partitioning step referenced `orders_df`, which was never created (NameError). Fixed by using `transformed_df`.

**Skills:** PySpark DataFrames · schemas · `withColumn` · partitioning · Spark SQL · Fabric Lakehouse
