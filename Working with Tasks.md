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





But as you can see in here, it can be possible that one parent task can have multiple child tasks,

indeed, up to 100 child tasks for one parent task.

Xiaoli and also one complete tree of tasks can include up to 1000 tasks.

So it's almost unlimited FOX.

But now let's have a look at how we can implement these trees of tasks.

So the syntax for this is pretty simple.

Just instead of this schedule property, we use the after property and then here we just specify this

parent task and the rest is just as we already know it.

Also, we can alter an existing task just by adding this after property here as well.

So this is possible as well.

So you see, it is pretty simple to implement such a tree of tasks.

So therefore, now let's have a look at some practical example and see in practice how we can implement

these trees of tasks.


So the first step, we want to make sure we are in the right database, therefore, we want to use this

command.

So just that we have set here this appropriate context.

Now we want to just see what we have done up until now.

So we see we have created already one task.

So this is the task customer insert.

And now let's also verify that we have here in this table.

So this was what this task was doing, writing here, rows in this table.

So this has worked fine and we have already inserted twenty nine rolls.

So now we want to create child tasks to this task that is inserting these roles.

So therefore we will prepare a second table.

Note that we have the same column names, but we don't have these default values, so we will do something

a little bit different.

But first, if we want to create these child tasks, we should suspend this parent task.

So therefore we alter this customer task and suspend it.

So this was our parent task and we want to suspend this first.

Now let's create these child tasks.

So we see it is very simple.

We have just our normal create or replace task here.

Our task name, we have the same warehouse.

And then he just instead of this schedule, we use this key word after and then we just specify here

this parent task.

And here the definition or the sexual statement that should be executed is just to insert into this

newly created customer to table.

And we want to insert everything that is currently in this customer's table.

Note that currently there are already 29 rolls.

So that means that in the first execution of this task, twenty nine rolls will be inserted in the second

time, probably then 30 rolls will be inserted.

So this is here, this definition of this task.

Now, let's also execute this one.

So we have successfully created this child task.

Now let's go ahead and create even one more task.

So this will be the last one.

So therefore, we will also create a third table.

So in here, we also insert another date.

So just to make it a little bit more interesting.

So here we can have a default.

So it will be, again, the current timestamp.

And now we want to create this last task.

So this is then again, copying everything from this customer to table here in this customer three table.

And this will be executed after this customer insert to task will be completed.

So let's also create this task and now we can again show all of our tasks just to see that now we have

three tasks and we have these created here.

We also see that we are the owner.

And that's also important to note that these tasks have all of the privileges that are from the owner.

So these privileges from the owner will be inherited to these tasks.

So this is something that is important to note.

And also we have here currently the schedule of 60 Minutes.

Therefore, let's also go ahead and alter this task.

So we use the altered task and we want to alter the customer insert and we want to set the schedule

to one minute so that we can just verify the results a little bit quicker.

So therefore, we'll do this.

It has been altered successfully.

Now let's have a look at the tasks again.

And we see we have here one minute and also in here we see the Predecessors'.

So this is here, these dependencies that we have created.

And also we note that all of these tasks are currently suspended, of course.

So therefore, we need to resume then and we need to start with the parent task.

Therefore, let's start with resuming this first task, also the second one and then also the last one.

So now we have all of our task.

We can verify this using the tasks we have all of our tasks started here.

We have one schedule and here we have these predecessors'.

So this is what we have.

And now we are curious of.

Out the results here in this customer two, so let's wait a little bit and then we will come back and

see the results in here.

So now we see we have 30 rolls inserted, not that before in this customer's one, we had twenty nine

rolls, so then this first task got completed, so there was one more roll inserted.

So then in this first table there were 30 rolls.

And now all of these 30 rows are now copied or inserted into this customer to table.

And all of those results were then also copied into this customer's three table.

So in here they are also 30 rows.

We have here this created.

And also on top of that, we have this insert.

And so we have seen that we can easily also create dependencies just by specifying here this parent

task and the after proper.

Hope it was helpful and see you in the next lecture.

Now, to get a little bit more advanced, let's also have a look at how we can call a stored procedure

so we will not talk about how our procedures work in general.

Just here, we will create a small and simple example stored procedure.

So, first of all, again, we want to set the right context and we want to verify that in this customer's

table.

Currently, in our case, there are 48 rules.

So just to verify this and now we first want to create our little example, the standard procedure,

so we can do this just using the create or replace, using the key word procedure.

And this is the name.

And we will use one variable.

So this will be created.

So later on we can pass this argument into this thought procedure and it will be used then.

So here we will just use this secure command here.

This syntax of this stored procedure is almost always pretty much the same.

So we always have to use the language and JavaScript and we want to, in our case, return a string.

We can also set the constraint if we want.

Not, not.

And then here this is the statement that we are using.

So we first define a variable.

So insert into the customer's table.

And here we only want to insert into the column created.

Also here we are using these values.

So this is what we want to insert.

And what is this Kolon one?

Well, this is just referring to our variable or to our argument that we are passing in here.

And this is also what we are specifying in here.

So what is this in here?

So this is just that this secure command will be executed.

So you see that we've defined this variable and this is what we also use here for this secure text.

And also, as you can see in here, this is just referring to the variable that we want to use in our

secure command.

And then in the end, also we just want to give here this return.

So once this has been returned successfully, this is also what we want to return.

So this is just our stored procedure, just serving as an example just to see basically how we can call

a stored procedure within a task.

So as I've said, this is not so important how this stored procedure work, but more how we can call

this stored procedure within a given task.

So let's have a look at how we can do this.

So this is the command for calling a stored procedure.

And as you can see, it's actually really not that difficult again.

So we have the same syntax just using this create or replace task.

Yes.

Our task name, we use the compute warehouse as our warehouse.

And then in this case, again, we schedule this for the interval of one minute and now we call it s

call.

And instead of just inserting here some rows, we are just calling this procedure.

And here we are again, parsing the value.

And this is the current timestamp that we want to paste here into this created.

So this is the value that we are passing here.

So calling a stored procedure in a task is, as we can see, not that complicated.

So therefore, let's also just create this task and let's have a look at these tasks.

So we first, of course, want to resume.

And so currently it is suspended.

And once we execute this and we now want to verify if this works.

So we remember there were 48 rows.

And now after one minute, we'll be back and check if this has worked well.

So now, one minute later, we see that now there are 49 rolls.

So this is just demonstrating that it's also an alternative to call a standard procedure in a task.

And as we've seen, this is also very simple to call this procedure within our task.

So let's have a look now also at how we can check the history on our task and get an overview of our

tasks, so like this, we can handle errors and can see what kind of tasks have been executed.

So the first step to get an overview of our existing tasks would be to use additional tasks come up

like this.

We can see that in our case, we have here for tasks created this insert insert to insert three and

this stored procedure code.

So here we see all of this information.

We see where this is in which database this is, what is the owner and what is also the schedule time

or hear the parent task and also if this task is here, started or suspended currently.

So this is just to get a little bit of an overview of our existing task.

If we want to have a little bit more detail, we should use the table function task history to be able

to query this function.

We have to be in the context of this demo DB.

So like this, we can set this context and now we are able to query from this task history or use this

task history function.

And in this case, I'm also ordering the results by the scheduled time.

And like this we get now the last one hundred records of our tasks.

So the every task that has been executed, no matter if it was failing or if it has worked.

So in here we see that these have been succeeded.

We have some that are just scheduled and therefore they have no where we start yet, but we still have

the value for the next schedule.

So you can see that here we see a lot of interesting information.

So what will be most interesting to us?

So first, this message here.

Error message is something that is really important to us because sometimes there can be some errors.

So we see this, of course, here in this state.

So let's maybe quickly also just order it here by the state.

So here we see that I have created this stored procedure and here we have the query text, which is

really nice.

So we can see this in here.

And this was the query text and here is the error message.

So this is what has been returned as an error message.

And this will help us to fix this task and the statement or the procedure that is used or call it in

this task.

So this is really helpful.

Of course, all of the other information are also helpful, such as the scheduled time where we start

and so on.

But now, as there are a lot of results, let's have a look at how we can make this a little bit more

organized so we can just use this function.

We know this, but we can also, in this function, just get the results of the last four hours or in

general, just specify here the scheduled time range.

So just starting from a certain point and we can specify this very easily using the date add function

here, we can specify our minute and so on.

And here we can then set how many hours we want to go back, starting, of course, in our case from

the current timestamp.

So like this, we would get all of the results that have been in the last four hours.

Also, we can limit the results to the latest five.

So here in this case, the number five.

And also on top of that, we can also get only results for a certain task.

So in here, we can insert the task name and like this we can get a little bit more specific results.

And so that this is a little bit more organized and we get a better overview of this.

So this is what we get.

So in here we get all of the executions of this, insert two, all of them have succeeded.

And this is much more organized for us.

Also, we can not only specify if this start range, but we can also give a period or a range with a

start and an end date as well.

So if we don't specify this.

This will be default the current time, but if we want to specify this, we can actually do that.

So this is what we can use.

So here we have the timestamp of the start.

So I have just used here a period of roughly seven minutes.

And if we execute this, we see.

So currently here I don't have any specific tasks, but we see that here we have seven tasks executed.

So always this customer tasks procedure.

And if you don't know what the current timestamp is, you can also use this select just to find out

about the current timestamp here in this format.

So this is what you can use, because this will not always be the same as in your local time.

So like this, you can check the history of your tasks and also find out about the errors.

Hope it was helpful and see you in the next lecture.

Now, let's also have a look at one really interesting TASC option, and that is to set a condition

on our task.

So when this condition is evaluated to true, then this task will be executed.

Otherwise this will not be executed.

So let's have a look at an example for this.

So first of all, we want to use the task DB and also make sure that you have here selected the role

of the account Atman in our case.

And then we just want to create or replace our customers table.

So this we already know and now we want to create a task and this has a condition.

So before we use this definition of this statement, in this task, we can set here this when option

and in here we can select a condition.

So a boolean value, any expression that is evaluated to a boolean expression.

So either to true or false.

And if this is evaluated to false, as in this case, then this will not be executed, this task, and

therefore let's execute or create this task.

And we see here another example.

So in this example, we have another condition.

And this, of course, will be evaluate to true and this will be evaluated to false.

Therefore, we expect that this task here will be executed.

So this command or this secure insert statement and as this here is evaluated to false, we expect that

this will not be executed.

So after we have created both of these tasks, we expect that once we start these tasks, which we can

do with this command, we expect that we will have only these names in our customer table.

So since this has been evaluated to force, we will expect not to see any inserts that include Mike

as a first name.

So let's wait for one minute and then also have a look at the results.

And we see that we have already one record here available.

And of course, this is Debka and not Mike.

So we have scheduled both of these tasks and we have only here this name included.

So if we refresh this, this will be validated that there is only this record here included.

Now, let's have a look at this command that we've learned about in the last lecture.

So here we see that we have the task history.

And in here we can see that this has here this condition text.

So we see that here.

These are the conditions.

And also let's have a look at the error messages.

And here we see that this condition is evaluated to force.

So we have here the error message and that is saying that this has not been executed because the conditional

expression for this task evaluated to false.

So this sounds like a great feature that we can use to set conditions.

Unfortunately, there is one little downside, and that is that we can currently only use one function

in this condition.

So let's demonstrate this.

If we want to use this function, for example, current timestamp, then this unfortunately will not

work because currently there is only one function available and possible to use in this condition.

And this is the system stream has data function, which is indeed quite useful that we can use this

function.

And later on, we will also see about these streams and how we can combine these streams with tasks.

So if we are using a task and a condition with this function, this will work.

But of course, later on we will learn about this function.

So for now, we know that we can also set conditions on our tasks.

And anyways, I think it's still worth mentioning and knowing that we can set this option here with

this condition.

And this is also useful if we are later on talking about streams.

Hope it was helpful.

And see you in the next lecture.




