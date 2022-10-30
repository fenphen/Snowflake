Dyanmic Data Masking

One more feature that we can use for security purposes is dynamic data masking. So if we have a table and we have some 

personal information in there that we want or need to protect, then we can also use data masking.  this features ensures

that only masked results are returned.  This masking is done at the column level and can be down by role.


And now let's have a look and practice how we can implement this and how we can use this in snowflake.

Let's start by creating a database to do all of our work and make sure we are using the accountadmin role:
      
      USE ROLE ACCOUNTADMIN;
      Create or replace database demo_db;
      
      USE DEMO_DB;
      
Now let us create a sample table to fill with sample data.

      -- Prepare table --
      create or replace table customers(
        id number,
        full_name varchar, 
        email varchar,
        phone varchar,
        spent number,
        create_date DATE DEFAULT CURRENT_DATE);

Ok, now we can fill the table with data:

      insert into customers (id, full_name, email,phone,spent)
      values
        (1,'Mike MacDwyer','mike@un.org','262-665-9168',140),
        (2,'Mark Pettingall','mark@mayoclinic.com','734-987-7120',254),
        (3,'Marlee Spadazzi','marlee@txnews.com','867-946-3659',120),
        (4,'Carolyn Tearney','heywood@patch.com','563-853-8192',1230),
        (5,'Marcus Seti','odilia@globo.com','730-451-8637',143),
        (6,'Mike Fenick','mfenick@broward.edu','568-896-6138',600);


Let's set up 2 sample roles that we can use to test our mask.

      CREATE OR REPLACE ROLE ANALYST_MASKED;
      CREATE OR REPLACE ROLE ANALYST_FULL;

Now let us give some permissions to the roles so that anyone in that role can see (or not see) or sample data:

      -- grant select on table to roles
      GRANT SELECT ON TABLE DEMO_DB.PUBLIC.CUSTOMERS TO ROLE ANALYST_MASKED;
      GRANT SELECT ON TABLE DEMO_DB.PUBLIC.CUSTOMERS TO ROLE ANALYST_FULL;

      GRANT USAGE ON SCHEMA DEMO_DB.PUBLIC TO ROLE ANALYST_MASKED;
      GRANT USAGE ON SCHEMA DEMO_DB.PUBLIC TO ROLE ANALYST_FULL;

      -- grant warehouse access to roles
      GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST_MASKED;
      GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST_FULL;


      -- assign roles to a user
      create or replace user testuser;
      GRANT ROLE ANALYST_MASKED TO USER testuser;
      GRANT ROLE ANALYST_FULL TO USER testuser;



      -- Set up masking policy

      create or replace masking policy phone 
          as (val varchar) returns varchar ->
                  case        
                  when current_role() in ('ANALYST_FULL', 'ACCOUNTADMIN') then val
                  else '##-###-##'
                  end;


      -- Apply policy on a specific column 
      ALTER TABLE IF EXISTS CUSTOMERS MODIFY COLUMN phone 
      SET MASKING POLICY PHONE;




      -- Validating policies
      use role accountadmin;
      grant role analyst_full to role accountadmin;
      grant usage on database demo_db to role analyst_full;
      
      
      USE ROLE ANALYST_FULL;
      SELECT * FROM CUSTOMERS;

      grant role analyst_full to role accountadmin;
      grant role analyst_masked to role accountadmin;
      grant usage on database demo_db to role analyst_full;
      grant usage on database demo_db to role analyst_masked;
      
      USE ROLE ANALYST_FULL;
      SELECT * FROM CUSTOMERS;
      
      use role accountadmin;
      
      USE ROLE ANALYST_MASKED;
      SELECT * FROM CUSTOMERS;

