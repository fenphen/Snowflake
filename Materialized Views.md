Materialized Views

Some queries take a long time to run.  The users get angry AND the costs can get fairly high for compute time.  A materialized view is one approach to minimize costs and frustration of users.  A regular view is just the text of a query saved to a named object but a materialized view actually stores data!  An MV is queried with a regular select statement.  When one of the base tables of the materialized view is updated, the MV is also updated.  Snowflake monitors changes to the base tablea and keeps the materialized views updated

  //-- Remove caching just to have a fair test -- 
  
      //if compute_wh warehouse does not exist be sure to create it!

    ALTER SESSION SET USE_CACHED_RESULT=FALSE; -- disable global caching to get clearer results
    ALTER warehouse compute_wh suspend;
    ALTER warehouse compute_wh resume;

    -- Prepare a transient database (orders), a schema (TPCH_SF100) and a table named orders all in the snowflake_sample_data
    database
    
    //let's use a transient database
    CREATE OR REPLACE TRANSIENT DATABASE ORDERS;
    use database orders;
    
 Here we create a schema to work with:
        
    CREATE OR REPLACE SCHEMA orders.TPCH_SF100;

    CREATE OR REPLACE TABLE orders.TPCH_SF100.ORDERS AS
    SELECT * FROM SNOWFLAKE_SAMPLE_DATA.TPCH_SF100.ORDERS;

    SELECT * FROM ORDERS LIMIT 100;
    
Here is a SQL query that could be replaced with a materialized view:
    
    -- Example statement view -- 
      SELECT
      YEAR(O_ORDERDATE) AS YEAR,
      MAX(O_COMMENT) AS MAX_COMMENT,
      MIN(O_COMMENT) AS MIN_COMMENT,
      MAX(O_CLERK) AS MAX_CLERK,
      MIN(O_CLERK) AS MIN_CLERK
      FROM ORDERS.TPCH_SF100.ORDERS
      GROUP BY YEAR(O_ORDERDATE)
      ORDER BY YEAR(O_ORDERDATE);
           
      //Here is our materialzied view code:
      
    -- Create materialized view
      CREATE OR REPLACE MATERIALIZED VIEW ORDERS_MV
      AS 
      SELECT
      YEAR(O_ORDERDATE) AS YEAR,
      MAX(O_COMMENT) AS MAX_COMMENT,
      MIN(O_COMMENT) AS MIN_COMMENT,
      MAX(O_CLERK) AS MAX_CLERK,
      MIN(O_CLERK) AS MIN_CLERK
      FROM ORDERS.TPCH_SF100.ORDERS
      GROUP BY YEAR(O_ORDERDATE);
      
  let's have a look and see if the mv exists.  Note that you can see the base query text used to create the view
        
        SHOW MATERIALIZED VIEWS;

       // Query the view and not the query duration
        SELECT * FROM ORDERS_MV
        ORDER BY YEAR;
        
       -- UPDATE or DELETE values in the base tables.
    UPDATE ORDERS
    SET O_CLERK='Clerk#99900000' 
    WHERE O_ORDERDATE='1992-01-01';

         -- Test updated data --
      -- Example statement view --   How long does the query duration take?
      SELECT
      YEAR(O_ORDERDATE) AS YEAR,
      MAX(O_COMMENT) AS MAX_COMMENT,
      MIN(O_COMMENT) AS MIN_COMMENT,
      MAX(O_CLERK) AS MAX_CLERK,
      MIN(O_CLERK) AS MIN_CLERK
      FROM ORDERS.TPCH_SF100.ORDERS
      GROUP BY YEAR(O_ORDERDATE)
      ORDER BY YEAR(O_ORDERDATE);



      -- Query materialized view - what was the query duration?
      SELECT * FROM ORDERS_MV
      ORDER BY YEAR;


      SHOW MATERIALIZED VIEWS;
        
Use an MV when the query would normally take a long time due to joins, etc...and when the underlying data does not change that frequently.  Joins, some aggregated functions, UDFs, HAVING clauses, ORDER BY clauses, and LIMIT clauses are not allowed inside MVs (for now) but Snowflake is always adding functionality, so stay tuned!

