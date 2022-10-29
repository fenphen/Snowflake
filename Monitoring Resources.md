Monitoring Resource Usage


So we know that we mainly pay what we use in terms of data storage and also the usage of the compute

resources.  There are ways to view general usage through the interface. But we can also query the base tables to

see more detailed usage information
    
    use Accountadmin;
    
    -- Table Storage
    //This view monitors data storage for tables
    //we have tables and information about tables, schemas,  time travel bytes and fail safe bytes.
    SELECT * FROM "SNOWFLAKE"."ACCOUNT_USAGE"."TABLE_STORAGE_METRICS";

    -- How much is queried in databases
    SELECT * FROM "SNOWFLAKE"."ACCOUNT_USAGE"."QUERY_HISTORY";

    SELECT 
    DATABASE_NAME,
    COUNT(*) AS NUMBER_OF_QUERIES,
    SUM(CREDITS_USED_CLOUD_SERVICES)
    FROM "SNOWFLAKE"."ACCOUNT_USAGE"."QUERY_HISTORY"
    GROUP BY DATABASE_NAME;


    -- Usage of credits by warehouses
    SELECT * FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY";

    -- Usage of credits by warehouses // Grouped by day
    SELECT 
    DATE(START_TIME),
    SUM(CREDITS_USED)
    FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY"
    GROUP BY DATE(START_TIME);

    -- Usage of credits by warehouses // Grouped by warehouse
    SELECT
    WAREHOUSE_NAME,
    SUM(CREDITS_USED)
    FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY"
    GROUP BY WAREHOUSE_NAME;

    -- Usage of credits by warehouses // Grouped by warehouse & day
    SELECT
    DATE(START_TIME),
    WAREHOUSE_NAME,
    SUM(CREDITS_USED)
    FROM "SNOWFLAKE"."ACCOUNT_USAGE"."WAREHOUSE_METERING_HISTORY"
    GROUP BY WAREHOUSE_NAME,DATE(START_TIME);
