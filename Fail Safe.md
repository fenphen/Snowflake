Fail Safe

So now that we've talked about the time travel feature, let's also talk about the next component or

the other component that is also part of the data protection lifecycle in Snowflake, and that is fail

safe. What actually is fail safe? So the purpose of failsafe is that it serves as a data protection of 

our data in case of a disaster.   So in case we lose the data or the data gets corrupted, some kind of 

disaster event where we lose all of this data and then we can still use this failsafe to recover this data.


We get a non configurable seven day period for permanent tables. Usually our tables are permanent tables.

So these are the common tables that we can see in our databases. We can configure a normal time travel of 0-90 days.  

So this will give us a seven day period AFTER that 90 days for these permanent tables. And this period starts immediately 

after the time travel period has ended.

First, let's look at our storage usage:

      use role accountadmin;


      SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE ORDER BY USAGE_DATE DESC;
      

Here we can see how much storage was used by date and also see how much fail safe data has been created.  Once the designated 

time travel period has ended, this table gets moved into this failsafe area. And here we can still restore this data.

But it's important to say that we cannot restore this data by ourselves, but we have to contact snowflake

support directly and then they will recover this data.  So unlike with the time travel, we can also not query this data 

directly. So with time travel, we could just select and then say before then the certain query, ID or timestamp.

Also, if a table gets moved into this failsafe zone, then this also contributes to our overall storage

costs.


We can also look at our storage used on a more readable level:

// Storage usage on account level formatted

            SELECT 	USAGE_DATE, 
                        STORAGE_BYTES / (1024*1024*1024) AS STORAGE_GB,  
                        STAGE_BYTES / (1024*1024*1024) AS STAGE_GB,
                        FAILSAFE_BYTES / (1024*1024*1024) AS FAILSAFE_GB
            FROM SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE ORDER BY USAGE_DATE DESC;



But for all of our other tables, the permanent tables, we have this failsafe period for seven days,

which we can recover that data by contacting the snowflake support in case of a dramatic event where

we lose all of our data or our data gets corrupted in any other way.


Now let's look at all of our storage use at a more granular level:


            // Storage usage on table level

            SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS;
            //This shows a very good overview of what is being stored for us.
            
            
We might want the results to be a bit more user friendly, so we can run the query below. So this is an overview of the usage 

of data by this failsafe. If we want to have a little bit more information, we can also query this information using the 

snowflake database.

            SELECT 	ID, 
                        TABLE_NAME, 
                        TABLE_SCHEMA,
                        ACTIVE_BYTES / (1024*1024*1024) AS STORAGE_USED_GB,
                        TIME_TRAVEL_BYTES / (1024*1024*1024) AS TIME_TRAVEL_STORAGE_USED_GB,
                        FAILSAFE_BYTES / (1024*1024*1024) AS FAILSAFE_STORAGE_USED_GB
            FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
            ORDER BY FAILSAFE_STORAGE_USED_GB DESC;


So like this, you can get an overview of all of your tables and also how much storage at these tables

are using.

