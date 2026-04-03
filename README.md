# bike-data-engineering-project


Applied Z-Ordering to the final tables to ensure lightning-fast performance for BI dashboards.
=======
## Project Overview: Medallion Architecture
-  The pipeline follows the industry-standard Medallion Architecture to ensure data quality and historical integrity.

## Bronze Layer (Ingestion):

- Used Python to extract .zip files from the public AWS S3 trip data bucket into a Unity Catalog Volume.
- Implemented Databricks Autoloader to incrementally ingest CSVs into a Delta table, using Checkpointing to handle cluster restarts and Schema Evolution for future-proofing.

## Silver Layer (Standardization & Historization):

- Cleaning: Standardized column names, cast data types (Timestamps, Integers), and filtered "junk" data (trips < 60 seconds).
- Engineering: Calculated Haversine distance and extracted temporal features (Hour, Day of Week).
- SCD Type 2: Built a Station Dimension table using MERGE logic to track changes in station names and locations over time, preserving historical accuracy.

## Gold Layer (Analytics):

- Aggregated data into a Commuter Route table, identifying peak travel patterns for Subscribers during morning and evening rush hours.
- Applied Z-Ordering to the final tables to ensure lightning-fast performance for BI dashboards.
>>>>>>> Stashed changes
