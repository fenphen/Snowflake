# Snowflake

A task is just an object that stores a certain sequel's statement and this will be then executed at a specific time

or in a certain interval that we can specify. It is important to note that in this task we can only specify one specific secured statement.

We can create standalone tasks with just one secure statement, or we can also create dependencies so

that one task will be executed after another task has been completed.   So in this lab, we'll learn and understand what is a task, will also learn how we can create a

task and the different scheduling methods.   How can we schedule this and use the different methods? How do we create standalone tasks? We will see how to create these tasks with dependencies.  We will also learn how we can check the task history.


1. Create a  database called Tasks DB and we will set up a table that we will use.

        //Create the Tasks_DB
            CREATE OR REPLACE TRANSIENT DATABASE TASKS_DB;

            USE TASKS_DB;

2. Now, let's create a customer table.  It will have three columns. An ID column where we use an auto increment.

If no name is specified, then the first name will be 'Marcus'



        // Create our table
            CREATE OR REPLACE TABLE CUSTOMERS (
            CUSTOMER_ID INT AUTOINCREMENT START = 1 INCREMENT =1,
            FIRST_NAME VARCHAR(40) DEFAULT 'Marcus' ,
            CREATE_DATE DATE);


To create a task, we can use the create or replace task.  We specify the name customer insert and there then we have to specify a few parameters.  We specify a warehouse.  We are using the compute_wh warehouse but you can change this or create that warehouse if it doesn't exist.  The tasks are run to the minute - so just multiple by 60 to get to hours.


        // Create task
             CREATE OR REPLACE TASK CUSTOMER_INSERT
             WAREHOUSE = COMPUTE_WH
             SCHEDULE = '1 MINUTE'
                        AS 
             INSERT INTO CUSTOMERS(CREATE_DATE) VALUES(CURRENT_TIMESTAMP);
             
To see what tasks we have in our system we can run the following:

              //Show the tasks on our instance  
              SHOW TASKS;

The show tasks command will give us a wealth of information about our tasks.  Review each column to see the status and details of our tasks.

We need to start our task....this is done with an alter task statement.

          //// Task starting and suspending
                 ALTER TASK CUSTOMER_INSERT RESUME;
                 
                 ///wait 3 minutes
                 
                 //This should show a few records inserted with the task
                 SELECT * FROM CUSTOMERS;
                 
                 //let's stop the task
                 ALTER TASK CUSTOMER_INSERT SUSPEND;
                 
                 //Let's see the status of our task
                 SHOW TASKS;

Not the status has changed of our tasks in this example.
             
There is another method of scheduling. 

So in this situation, I have just used UTC because this is an easy one.

But we could also specify here a different time zone and then we have here after we have specified using

cron.

                //  
                CREATE OR REPLACE TASK CUSTOMER_INSERT
                   WAREHOUSE = COMPUTE_WH
                  SCHEDULE = 'USING CRON 0 7,10 * * 5L UTC'
                  AS 
                 INSERT INTO CUSTOMERS(CREATE_DATE) VALUES(CURRENT_TIMESTAMP);
    

                # __________ minute (0-59)
                # | ________ hour (0-23)
                # | | ______ day of month (1-31, or L)
                # | | | ____ month (1-12, JAN-DEC)
                # | | | | __ day of week (0-6, SUN-SAT, or L)
                # | | | | |
                # | | | | |
                # * * * * *
       
                // Example of how to run this Every minute
                SCHEDULE = 'USING CRON * * * * * UTC'

                //  Example of how to run thisEvery day at 6am UTC timezone
                SCHEDULE = 'USING CRON 0 6 * * * UTC'

                //  Example of how to run thisEvery hour starting at 9 AM and ending at 5 PM on Sundays 
                SCHEDULE = 'USING CRON 0 9-17 * * SUN America/Los_Angeles'


                CREATE OR REPLACE TASK CUSTOMER_INSERT
                 WAREHOUSE = COMPUTE_WH
                 SCHEDULE = 'USING CRON 0 9,17 * * * UTC'
                  AS 
                 INSERT INTO CUSTOMERS(CREATE_DATE) VALUES(CURRENT_TIMESTAMP);



We can also create what is called a tree of tasks.  These are tasks that are dependent on one another. This allows the tasks

to be developed and maintained independently to allow for modularity. It is possible for one task to have up to 100 child 

tasks.  First, let us use the task_db database, review our tasks, suspend the original task, and look at our customer records:

        USE TASK_DB;
 
        SHOW TASKS;

        SELECT * FROM CUSTOMERS;

        // Prepare a second table
        CREATE OR REPLACE TABLE CUSTOMERS2 (
            CUSTOMER_ID INT,
            FIRST_NAME VARCHAR(40),
            CREATE_DATE DATE);


        // Suspend parent task
        ALTER TASK CUSTOMER_INSERT SUSPEND;
        
Now we can create a child task that is dependent upon the customer_insert task we created above:

        
        // Create a child task
        CREATE OR REPLACE TASK CUSTOMER_INSERT2
          WAREHOUSE = COMPUTE_WH
          AFTER CUSTOMER_INSERT
          AS 
         INSERT INTO CUSTOMERS2 SELECT * FROM CUSTOMERS;
         
Let's go a step further: let's create a third table called Customer3 and another child task depended on our first child task:
    
        // Prepare a third table
        CREATE OR REPLACE TABLE CUSTOMERS3 (
            CUSTOMER_ID INT,
            FIRST_NAME VARCHAR(40),
            CREATE_DATE DATE,
            INSERT_DATE DATE DEFAULT DATE(CURRENT_TIMESTAMP))  ;  


        // Create a child task
        CREATE OR REPLACE TASK CUSTOMER_INSERT3
            WAREHOUSE = COMPUTE_WH
            AFTER CUSTOMER_INSERT2
            AS 
            INSERT INTO CUSTOMERS3 (CUSTOMER_ID,FIRST_NAME,CREATE_DATE) SELECT * FROM CUSTOMERS2;

Note the syntax: instead of this schedule property, we use the after property and then here we just specify this

parent task and the rest is just as we already know it.  Also, we can alter an existing task just by adding this after 

property here as well.

We can use these tasks for a variety of things just as you would schedule a cron job or other operating system job.

We can test this tree by resuming the first task:

         //alter the 3rd, then 2nd, then 1st task
         alter task customer_insert3 resume;
         alter task customer_insert2 resume;
         alter task cusomter_insert resume;       

         //if you successfully created the task tree, wait a few minutes, then check out the data!!
         select * from customers;
         select * from customers2;
         select * from customers3;
         
A task can be very useful when combined with inputs (such as a stream) to build an end-to-end data pipeline. We saw that the 

Snowflake task engine has CRON and NON-CRON scheduling abilities. The CRON variant should look familiar syntactically if you 

are a Linux person.

