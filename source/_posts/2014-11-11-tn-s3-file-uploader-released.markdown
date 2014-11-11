---
layout: post
title: "tn_s3_file_uploader released"
date: 2014-11-11 13:01:58 -0800
comments: true
categories: 
---

## Announcement
ThinkNear is delighted to announce that our `tn_s3_file_uploader` is released as open-source under [APLv2](http://www.apache.org/licenses/LICENSE-2.0.html). `tn_s3_file_uploader` is a Ruby gem that we use internally to upload log files to Amazon S3 where they can be stored until we need to retrieve them for further analysis or processing.

You can visit the gem's page on [Ruby Gems](https://rubygems.org/gems/tn_s3_file_uploader). Please, visit the project's [getting-started page](https://github.com/ThinkNear/tn_s3_file_uploader/wiki/Getting-started) for more information on how to install and use the gem.

## Motivation
ThinkNear's [Real Time Bidding](http://en.wikipedia.org/wiki/Real-time_bidding) service produces billions of log entries per day and we have set up a simple but robust pipeline around rotating those logs and storing them on [Amazon S3](http://aws.amazon.com/s3/). The two key parts of this pipeline have been released to the public as open-source software.

First our [AWS Elastic Beanstalk](http://aws.amazon.com/elasticbeanstalk/?sc_channel=PS&sc_campaign=beanstalk_awns&sc_publisher=google&sc_medium=beanstalk_b&sc_content=elastic_beanstalk_e&sc_detail=amazon%20elastic%20beanstalk&sc_category=beanstalk&sc_segment=177&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!50999240322!e!!g!!amazon%20elastic%20beanstalk&ef_id=VCxP0QAABUt7hwuO:20141111211834:s) [logrotate and config scripts](https://github.com/ThinkNear/aws_templates) rotates the logs produced by our service.

Second the aforementioned ruby gem uploads the rotated logs on S3 with a year/month/day/hour [partitioned destination path](https://github.com/ThinkNear/tn_s3_file_uploader/wiki/Getting-started#partition-uploaded-files-based-on-date) that makes it easy to retrieve and analyze the logs for a specific period of time.

## Contribute
ThinkNear Engineering values the openness of the open source software model and we would like to invite anyone that finds `tn_s3_file_uploader` useful to contribute. Visit the [project's issue tracker page](https://github.com/ThinkNear/tn_s3_file_uploader/issues) to submit questions, bugs or new features. 

If you want to hack around the code, [fork the project on GitHub](https://github.com/ThinkNear/tn_s3_file_uploader/fork) and see the [how to contribute page](https://github.com/ThinkNear/tn_s3_file_uploader/wiki/Contribute) if you want to submit your code to the project.