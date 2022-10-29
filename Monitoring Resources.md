Monitoring Resource Usage


So we know that we mainly pay what we use in terms of data storage and also the usage of the compute

resources.  There are ways to view general usage through the interface. But we can also query the base tables to

see more detailed usage information.   We pay for compute resources and storage - so monitoring both are important.
    
    use role Accountadmin;
    
Table Storage: this view monitors data storage for tables, schemas, time travel bytes and fail safe bytes.

        SELECT * FROM "SNOWFLAKE"."ACCOUNT_USAGE"."TABLE_STORAGE_METRICS";

How much is being queried in databases?  this query gives us the historical view of queries so we can see who 

has been doing what in our system.
        
        SELECT * FROM "SNOWFLAKE"."ACCOUNT_USAGE"."QUERY_HISTORY";

This query will tell us what database credits are used for compute and what is being used for cloud services.
           
        SELECT 
        DATABASE_NAME,
        COUNT(*) AS NUMBER_OF_QUERIES,
        SUM(CREDITS_USED_CLOUD_SERVICES)
        FROM "SNOWFLAKE"."ACCOUNT_USAGE"."QUERY_HISTORY"
        GROUP BY DATABASE_NAME;

Here we can monitor the usage of credits by warehouse

    SELECT * FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY";

Usage of credits Grouped by day to look for usage patterns.
         
         SELECT 
            DATE(START_TIME),
            SUM(CREDITS_USED)
            FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY"
            GROUP BY DATE(START_TIME);

Usage of credits by warehouses // Grouped by warehouse

            SELECT
            WAREHOUSE_NAME,
            SUM(CREDITS_USED)
            FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY"
            GROUP BY WAREHOUSE_NAME;

Usage of credits by warehouses // Grouped by warehouse & day

            SELECT
            DATE(START_TIME),
            WAREHOUSE_NAME,
            SUM(CREDITS_USED)
            FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY"
            GROUP BY WAREHOUSE_NAME,DATE(START_TIME);
