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

Here we can see how much storage was used by date and also see how much fail safe data has been created.


So once this time travel period has ended, this table gets moved into this failsafe area.

And here we can still restore this data.

But it's important to say that we cannot restore this data by ourselves, but we have to contact snowflake

support directly and then they will recover this data.

So unlike with the time travel, we can also not query this data directly.

So with time travel, we could just select and then say before then the certain query, ID or timestamp.

And this is not possible for us in a failsafe.

So this is only possible by contacting the snowflake Sopot.

And on top of that, it's also worth mentioning that this failsafe zone, we have to also pay storage

costs.

So if a table gets moved into this failsafe zone, then this also contributes to our overall storage

costs.

Now, let's have a closer look at how this failsafe exactly works and also how it relates to the time

travel component in Snowflake.

So we know we have our current data storage.

These are the tables we can see.

We can access them directly.

We can query the data and so on.

Then also on top of that, we learned that we have the time travel capability where we can use a select

at or before statement, and we can also drop the data through just a simple query.

And here we have a retention time, depending on the additions we are using and the parameter we set

for this retention time between zero and 90 days.

And then once this period ends, the data gets automatically moved into this failsafe zone.

So this will happen automatically.

Let's say we have one day of retention time, then after one day this data gets moved into this failsafe

zone.

But it's important to note that we can make no use for operations.

So this can not be queried directly by us, as we've said.

But this is only recoverable by the snowflake support.

And as we've said, this is also for seven days period, then available beyond the time travel.

So this is, as we've said also for this permanent tables and for the transient tables.

We have zero days transient tables.

We will also learn about in the following lectures where we talk about the different table types.

So this is just that we save the storage cost if this table is needed for some kind of development and

we don't need a normal and permanent table in here.

But for all of our other tables, the permanent tables, we have this failsafe period for seven days,

which we can recover that data by contacting the snowflake support in case of a dramatic event where

we lose all of our data or our data gets corrupted in any other way.

So now that we've learned what this failsafe is, let's have a look again in the snowflake account to

see the storage that is consumed by.

Our failsafes.
So now we are back in our account and we want to find out how we can get the information of how much

storage is used by the failsafe, and we have two ways of doing that.

And we will learn about both of these ways.

And for both of these methods, we need to switch here this role so we select which role.

And here we have to select account statement so that we are able to use both of these methods.

So for the first one, we will select here this account statement and here we have the storage used.

So therefore let's select the storage used.

And here we have the components of our total storage.

So we see that in general.

In total here it was two point three gigabytes and here it was around seven point six.

And this has not changed since then.

But if we navigate the databases, we see that here we had two point three.

This was the same in here.

And then on the 13th, we had here some data loaded into our database.

But then we have apparently deleted some of this data.

So we see that from here on, the data consumed for that databases was much less.

But if we navigate to the failsafe area, we see that this was exactly also the day where this data

that we have probably deleted has been moved into the failsafe zone.

So this is now here in this failsafe area.

So this is an overview of the usage of data by this failsafe.

If we want to have a little bit more information, we can also query this information using the snowflake

database.

So in here we see that we have also not only been here in this role of the account, Atman, but also

we should switch here this role and change this year also to the account atman so that we have enough

rights to query these views that we want to see in here.

And if we have switched here, the roll to the account atman, we see that there is an additional database

appearing and this is what we work where we here also we have here this schema account usage and here

we are interested in the view storage usage.

So we find this view in here.

And this is the first view that we would query to get information on the storage that is used.

So we see here the storage used in bytes grouped by the days.

And this is not always so easy to read with these bytes.

Therefore, we can just format this a little bit nicer to just see the results in gigabytes.

So in here we can now compare the storage of every day with the failsafe of these debts.

And we can see, as we have seen before, that here the failsafe can be also higher than the storage

of our current databases.

But if we would like to have more detailed information on the tables now, then we can use the view

table storage metrics.

So this will give us more detailed information on a table level, which is also very interesting for

us.

So now we could also order this by the storage of that is used by the failsafe.

So, again, we can format this a little bit nicer so that we see only what we want to see and that

we have still also here this shown in gigabytes.

So here we see now this audit by the different tables and we can see that here we have multiple tables

that are called order caching.

This can be, of course, different in your case if you have different table names and different tables

created.

But here you see that this table I have created multiple times currently, this table is available and

here the storage used is one point two gigabytes.

And apparently I have done a lot of changes and maybe have dropped this table also in the past.

And therefore, there's a lot of data related to this table in the failsafe area.

And in here, we also see the time travel storage, so apparently yesterday there has been no data changes

and therefore there is no data here in this table.

So this gives us just a little bit more detailed information on the data that is here consumed by our

tables and also here that split between the storage, then the time travel storage and then the failsafe

storage used.

So like this, you can get an overview of all of your tables and also how much storage at these tables

are using.

