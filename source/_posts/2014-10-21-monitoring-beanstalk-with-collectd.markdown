---
layout: post
title: "Monitoring Beanstalk with collectd"
date: 2014-10-21 17:51:02 -0700
comments: true
categories: 
---

## Our system 

Thinknear's [Real Time Bidding](http://en.wikipedia.org/wiki/Real-time_bidding) service is built on [AWS Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html).
We run around one-hundred instances to serve billions of auctions a day. 
The system must be highly available, performant and maintain high success rates. 
Operationally, engineers must be able to ship code throughout the week and change configurations without disrupting service. 
We make data driven decisions to meet these standards by using metrics gathering systems like [collectd](https://collectd.org/). 

> If you don't know where you are going, you'll end up someplace else -- Yogi Berra

If you run your services in the cloud, then you know there is a risk of overpaying. 
There are a lot of factors that go into your cloud’s cost. 
For example, we want to know when: 

- We are under-utilizing instances. 
- Hosts are suffering from [steal time](http://blog.scoutapp.com/articles/2013/07/25/understanding-cpu-steal-time-when-should-you-be-worried
). 
- We need a instance type that better suits our needs. 

Tracking memory, CPU and network statistics are key to meeting these goals and making sound decisions. 
But we don't stop there. 
There are multitudes of metrics to track. 

<!-- more -->

## Measure everything

At the instance level, AWS EC2 CloudWatch metrics from a Beanstalk environment didn't meet our needs. 
CloudWatch provides CPU, Disk IO and Network IO; however, it lacks load, memory, fine-grained configurations and a method to extend the set of collected metrics. 
Our engineering team needs more. 
That is why we are users of the rock-solid statistics collection daemon collectd.

Collectd can track any system metric we want -- the [official plugins list](https://collectd.org/wiki/index.php/Table_of_Plugins) is extensive. 
It works as a daemon running a loop over a set of plugins. 
There is no graphing built into collectd so one must be configured. 
[Graphite](http://graphite.readthedocs.org/en/1.0/tools.html) is a popular frontend -- but it only does graphing. 
We like [Librato](https://metrics.librato.com/), a paid service, which has beautiful graphs, aggregates metrics and provides alerting similar to CloudWatch. 

> Do not look where you fell, but where you slipped -- African proverb

We use collectd to track memory, load, cpu, network, JVM, process statistics and more. 
Every Beanstalk host is deployed with a collectd installation as a standalone metric collector and reporter that is configured. 
It is configured to report to Librato every minute. 
This gives us the detailed performance tracking and a monitor for unexpected events. 
When things go wrong, we can see what, when and how often an event happened. 
When we make instance type changes, we can see exactly how we use system resources we need for each host.

## Getting started with Elastic Beanstalk and collectd

Setting up collectd with Beanstalk is easy. 
Below is a simple [Elastic Beanstalk configuration](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html
) that will:

1. Install collectd on your instances.
2. Create a basic collectd configuration file for collecting CPU, network IO, load and memory statistics. 
  It uses the write_http plugin to report to Librato.
3. Start and supervise collectd daemon with the collectdmon monitor process. 
  There are alternatives like [monit](http://mmonit.com/monit/), but we like collectdmon because it is installed with the collectd package.

This configuration was tested on a 2014.03 Amazon Linux EC2 instance. 
This version of Amazon Linux comes with [collectd-5.4.1](http://aws.amazon.com/amazon-linux-ami/2014.03-packages/) as an available RPM package.

Put this file into your project's ```.ebextensions``` directory. 
Replace the ```User``` and ```Password``` fields with your credentials.

```yaml
# .ebextensions/collectd.config
files:
  "/etc/collectd.conf" :
    mode: "000664"
    owner: root
    group: root
    content: | 
      LoadPlugin syslog
      LoadPlugin cpu
      LoadPlugin interface
      LoadPlugin load
      LoadPlugin memory
      LoadPlugin write_http
      <Plugin write_http>
        <URL "https://collectd.librato.com/v1/measurements">
          User "user@example.com"
          Password "password"
          Format "JSON"
        </URL>
      </Plugin>
      Include "/etc/collectd.d"

packages: 
  yum:
    collectd: []

commands:
  start_collectd:
    command: killall collectd; collectdmon -c collectd
```

Collectd will start reporting metrics to Librato when you deploy your application. 

Then you can start building Librato instruments and dashboards that look like this:

![collectd CPU](/images/collectd_cpu.png)

![collectd dash](/images/collectd_dash.png)

## Summary 
At Thinknear, we strive to measure anything that will help us understand the behavior of our systems in production. 
Collectd is a great tool for us to track instance level statistics that CloudWatch does not provide.
