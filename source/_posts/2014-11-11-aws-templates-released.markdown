---
layout: post
title: "aws_templates released"
date: 2014-11-11 14:45:14 -0800
comments: true
categories: 
---

## Announcement
Thinknear is releasing our `aws_templates` as an open-source project under [APLv2](http://www.apache.org/licenses/LICENSE-2.0.html).
`aws_templates` is an example deployment and configuration setup for [AWS Elastic Beanstalk](http://aws.amazon.com/elasticbeanstalk/).

You can visit the project's [GitHub page](https://github.com/ThinkNear/aws_templates). The README has more information on the features and behavior of the configuration.

## Motivation
Thinknear's Elastic Beanstalk environments are low-latency, high-throughput systems serving billions of auction requests per day from dozens of exchanges.
The `aws_templates` works with Beanstalk's `.ebextensions` to configure EC2 instances with
* RAID 0 
* log rotation to AWS S3 using [tn_s3_file_uploader](https://github.com/ThinkNear/tn_s3_file_uploader)
* collectd and JMX remote monitoring
* AWS ELB configuration
* Custom system settings

This project is intended to share a method of deployment and configuration using a collection of scripts and Beanstalk's `.ebextensions`. 

## Contribute
Thinknear Engineering would like to invite anyone who finds `aws_templates` useful to contribute to the project. Visit the project's [issue tracker page](https://github.com/ThinkNear/aws_templates/issues) to submit questions, bugs, or new features.