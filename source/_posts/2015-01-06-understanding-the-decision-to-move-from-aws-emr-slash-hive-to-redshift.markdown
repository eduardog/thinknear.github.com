---
layout: post
title: "Understanding The Decision To Move From AWS EMR/Hive To Redshift"
date: 2015-01-06 17:40:47 -0800
comments: true
categories: 
---

At Thinknear we always want to make sure we are doing our best to use the right tool for the job. So when Redshift came out we decided to evaluate our current reporting and analytics pipeline and see if Redshift could help us improve. At the time we were using Hive/Hadoop on EMR for all our reporting and analytics purposes. We saw Redshift as a way to speed up our reporting infrastructure without completely rearchitecting and give our business team a much easier way to access the data. Given these goals we evaluated Redshift against our current Hive/Hadoop solution and found the following pros and cons.

Pros:

1. Redshift is fast.
 - For a lot of our critical reporting jobs, once the data was loaded, jobs that were taking 10-30 minutes were down to the 1-2 minute range.
2. Ad-hoc analysis is easier.
 - The learning curve to use redshift is much less than hive/hadoop. For our business and data analysts it was often extra overhead to explain and understand hadoop clusters worked. 
 - Redshift has also been much less surprising so far. Hive has some limitations with more advanced queries that we have not seen with Redshift.
3. No longer dealing with EMR/Hadoop versions
 - A lot of issues we had with EMR was with new versions of hadoop/EMR. Things would break with bad builds or different performance characteristics and they were often a pain to deal with. It seems with Redshift more of this is abstracted away and we haven’t had to deal with any of these types of issues.
4. Cost
 - For a lot of our Hive tasks we had to roll up our data in various ways (hourly, daily, weekly, etc.), before we could use it in our reports. This caused us to have a lot of clusters and was getting expensive. With redshift we can sometimes skip roll ups and the jobs are running much faster. It turned out that the costs were less for Redshift than hive for some of our reporting pipeline.


Cons:

1. Redshift does not easily scale up for dynamic workloads. 
 - The reporting pipeline at thinknear can fluctuate for large advertising days. With EMR we can spin up extra worker nodes to scale up easily. If Redshift becomes too slow or backed up due to high data or query volumes, adding nodes is very time consuming. Resizing is the most straight forward, but puts your cluster into read only mode for an extended period, which we can’t live with. This means that snapshotting and reloading our data is the only approach and that can be a multi day process for us. 
2. Redshifts workload management does not fully solve the parallel job problem. 
 - With EMR, running jobs in parallel is pretty safe, one job is not going to affect another job. Redshift has workload management which allows you to split workers into queues. These queues have limits on the amount of memory they can use but not the amount of CPU. A lot of our jobs are CPU intensive so this limits the amount of jobs we are able to run in parallel.
 - Redshift is also limited by disk space for some queries. When running multiple jobs with big joins you may run into disk space limitations. To be fair this may be solvable by writing more efficient queries.
3. Loading data into your Redshift cluster takes time. 
 - We have some jobs that run hourly and so we need data loaded into our cluster hourly. If we want to maintain acceptable performance we need to vacuum our data (Sidenote: Amazon docs say that you may not need to vacuum your data if you load it in sequential order, we saw performance improvements when we ran the Vacuum anyway). For us the overall time to load and run the queries were still faster than running Hive but it is still something to take into consideration.
4. With Redshift you have to define schemas with lengths and encodings.
 - In EMR you do not need to have encodings and lengths defined when dealing with data. You just define what it looks like in terms of data types and hive handles the rest. Redshift needs the encoding and lengths to be more efficient to be more efficient on disk space. This means you have some upfront costs to figure these things out before importing a table into redshift.
5. Redshift does not offer any custom function or array support. 
 - We didn’t use a lot of custom functions in Hive (UDF/UDAF), but the ones we did use were very useful. Some of the data we wanted to analyze used arrays so in order to get it into redshift it required some pre-processing or ugly queries. 
6. Redshift has a higher cost of failure
 - This is one we didn’t really think about until it happened to us. One of our clusters had a failure and they had to replace a node, causing a complete rebalance of data. This slows performance for an extended period of time (depending on how much data you have) and can become a major issue if you have critical processes running in Redshift. EMR would fail as well but the cost of booting up another cluster and running it was much less.


In the end we moved the processes that made the most sense to Redshift and left some on EMR. The real win was for our analytics team, we ended up moving a lot of our data onto a separate redshift cluster for them. We had some different third party solutions and Hive for dealing with analytics and the ease of using redshift really made them more productive and in turn allowed us to get rid of other services in favor of Redshift.

