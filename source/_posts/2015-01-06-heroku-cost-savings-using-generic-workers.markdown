---
layout: post
title: "Heroku cost optimization using generic workers"
author: Ciprian Costin
date: 2015-01-06 17:43:10 -0800
comments: true
categories: 
---

During the early days of Thinknear, <a href="https://github.com/resque/resque" target="_blank">Resque</a> was the most prevalent background job processor for our Rails applications.
However, Resque was not multithread-friendly, and, as our applications grew, this put a toll on our Heroku monthly bill.

We tried out <a href="http://sidekiq.org" target="_blank">Sidekiq</a>, really liked it, and now use it exclusively. Unlike Resque, Sidekiq supports multiple threads 
working on multiple jobs concurrently.
According to Sidekiq's main developer, Mike Perham, one large Resque farm with a 68GB RAM footprint can be brought down to 1 GB RAM by using threads instead of processes. 
[<a href="https://github.com/mperham/sidekiq/wiki/Internals" target="_blank">*source*</a>]

<!-- more -->

We analyzed the distribution of background jobs across dynos. 
We use a family of dynos for each queue, running up to 50 dynos per queue and different families being on for 750 hours per month at the low end, and up to to 40,000 hours 
per month at the high end.

We utilize a home-grown type of autoscaling to dial up and down workers based on the jobs in the queue, since all dyno families needing it are background job focused. 
Its simplicity provided long-term reliability.

While this setup fit the production processing needs, it was inefficient for non-production environments.
With 19 always on dyno families, and 3 non-production environments (sandbox, test, and integration) used 24/7 for unit, contract level and end-to-end integration testing, 
something needed to change.

Sidekiq supports assigning multiple queues to a single worker process. In non-production environments, we assigned every queue to a single, generic worker and 
noticed a couple of standard dynos are enough to process the non-production load. 

As a result, we're now saving 90-95% on the Heroku bill of development and testing environments.

Sample Procfile:

`web: bundle exec unicorn -p $PORT -c ./config/unicorn.rb`  
`clock: bundle exec clockwork config/clock.rb`

`core_worker: bundle exec sidekiq -c 1 -q core`  
`creator_worker: bundle exec sidekiq -c 1 -q creator`  
`emr_worker: bundle exec sidekiq -c 1 -q emr`  
`maintenance_worker: bundle exec sidekiq -c 1 -q maintenance`  
`measure_worker: bundle exec sidekiq -c 1 -q measure`

`generic_worker: bundle exec sidekiq -c 1 -q core -q creator -q emr -q maintenance -q measure`
