---
layout: post
title: "Rails Migration and Dependencies on User Defined Tables/Views"
date: 2015-03-16 16:47:39 -0700
comments: true
categories: 
is_newest: true
author: Tassos Livogiannis
---


Thinknear Engineering values reporting because it allows us to monitor how our mobile advertising platform behaves in its [real time bidding](http://en.wikipedia.org/wiki/Real-time_bidding) environment.
We also value business intelligence, because it gives Thinknear's operations and data science teams a tool to perform analytics and fine tune our marketing platform to ensure it meets our customers' needs and exceeds their expectations.

To this purpose, we are using [AWS' Amazon Redshift](http://aws.amazon.com/redshift/) as the main tool of our BI stack and we handle data loads and migrations through our [Ruby on Rails](http://rubyonrails.org/) applications.
Being a BI tool, our redshift clusters need to accommodate tables and views created not only by our applications but by our operations and data science teams, as well.
It is quite common for user defined tables and views to rely on application defined tables and views, which makes migrations a challenge.

In the following post, we present two SQL queries that are useful when trying to identify dependencies before running migrations.

## Find locks held by running queries

Queries ran by users can hold locks on tables that the migration might operate on.
If a user query holds a lock on a table that is bound to change by a migration, the migration will hang until the user query finishes (which in our case can sometimes be several hours).
Empirically, we have found that querying the catalog view [PG_LOCKS](http://www.postgresql.org/docs/9.4/static/view-pg-locks.html) for finding existing locks tends to be more accurate than querying the [STV_LOCKS](http://docs.aws.amazon.com/redshift/latest/dg/r_STV_LOCKS.html) table.
Based on that, the following query returns all the locks on database objects (in any schema) in a redshift cluster.

    SELECT getdate(),
    c.relname,
    n.nspname,
    l.database,
    l.transaction,
    l.pid,
    a.usename,
    l.mode,
    l.granted
    FROM pg_catalog.pg_locks l
    JOIN pg_catalog.pg_class c ON c.oid = l.relation
    JOIN  pg_namespace n ON n.oid = c.relnamespace
    JOIN pg_catalog.pg_stat_activity a ON a.procpid = l.pid;

In the query above:

1. ```getdate()``` is the current timestamp. [See GETDATE documentation](http://docs.aws.amazon.com/redshift/latest/dg/r_GETDATE.html)
1. ```c.relname``` is the name of the table or view the lock is held on.
1. ```n.nspname``` is the name of the schema that the table or view belongs to.
1. ```l.database``` is the current database id or a special value as described in [PG_LOCKS table documentation](http://www.postgresql.org/docs/9.4/static/view-pg-locks.html).
1. ```l.transaction``` is the transaction the query belongs to.
1. ```l.pid``` is the pid of the query that obtained the lock.
1. ```a.usename``` is the name of the user that issued the query that holds the lock.
1. ```l.mode``` is the lock type (e.g. AccessShareLock).
1. ```l.granted``` is true or false if the lock was granted or not.

This query is slightly better than the one provided in the [STV_LOCKS example](http://docs.aws.amazon.com/redshift/latest/dg/r_STV_LOCKS.html) since it will give the username and table or view name out of the box.
As a result, the engineer who runs the migration knows which table or view might be an impediment and they would also know who to talk to, before running the migration.

## Find dependencies for a specific column

One type of potential dependency between user defined objects and application defined ones could be a _foreign key_ constraint.
This type of constraint can be found using the query mentioned [here](http://stackoverflow.com/a/1152321).
However, there are different types of dependencies apart from _foreign key_ constraints.

An example of such direct dependency is a user defined view that is built up by columns of application defined table(s).
Imagine there is a table called ```orders``` that is partitioned monthly in the redshift database.
Now, let us assume that the operations team wants to create views that gather all orders to produce quarterly reports.
As a result, one of these quarterly views might be defined by the operations team like this:

    CREATE VIEW orders_2015_Q1 AS 
        SELECT column_name1, column_name2... from orders_201501 UNION ALL
        SELECT column_name1, column_name2... from orders_201502 UNION ALL
        SELECT column_name1, column_name2... from orders_201503 UNION ALL
        SELECT column_name1, column_name2... from orders_201504;

This view does not have a foreign key constraint to any of the four partitions of the orders table it uses.
However, it has a dependency on the columns of those partitions.
If a migration wants to drop or alter the type of any of columns `column_name1,column_name2...`, the migration will fail.
In the case of a drop column, a `cascade` clause can be used to drop the column to all dependent objects.
Since the tables or views that reference the altered columns might be user defined, we need to ensure it is safe to use the `cascade` clause before running the migration.

The following query can be used to spot dependencies like the one explained above:

    SELECT distinct dependee.relname   
    FROM pg_depend   
    JOIN pg_rewrite ON pg_depend.objid = pg_rewrite.oid   
    JOIN pg_class as dependee ON pg_rewrite.ev_class = dependee.oid   
    JOIN pg_class as dependent ON pg_depend.refobjid = dependent.oid   
    JOIN pg_attribute ON pg_depend.refobjid = pg_attribute.attrelid   
    AND pg_depend.refobjsubid = pg_attribute.attnum   
    WHERE dependent.relname = 'orders_201503'  
    AND pg_attribute.attnum > 0   
    AND pg_attribute.attname = 'column_name1';   

The query above will find any database objects (tables, views) that use column `column_name1` from table `orders_201503`.

Alternatively to using a `cascade` clause, one might want to drop the `orders_2015_Q1` view and restore it after the migration has finished.
The definition of a view can be found before the migration:

    SELECT definition FROM pg_views WHERE viewname='orders_2015_Q1';

After the migration, the view can be restored using the new types of columns or omitting columns that were dropped.

## Handle everything in the application

Obviously, the best approach is to strive for having only application code handle the schema definition in your BI database.
If operations and data science are heavily using a view or table created outside of the application, then that view or table needs to be part of the application defined schema.
However, there are cases where that is not efficient, since SQL proficient data scientists can quickly write SQL code to perform their analysis without having to request the product development team for a migration.
In the end, it's a trade-off that the engineering team needs to measure, but if the schema can be manipulated by both users and the application, then the queries presented above can help identify potential migration problems in advance.