---
layout: post
title: Metrics to track by a Web Service
categories: [Metrics, Web-service]
---

## What to measure?
We like the philosophy: measure everything. But sometimes we need a little inspiration to think of the specific metrics we should be measuring. To help get the creative juices flowing, we like to ask myself these kinds of questions:

- We need a little inspiration to think about specific metrics that will deliver the most value for the team. To get to the specific metrics and alarm we want to have, we can answer some specific questions:
- What are our customers trying to achieve when they use our software?
- What decisions do customers have to make when using our software (click this or click that)? What do they choose?
- What are our important business objectives?
- When our software fails, how will that manifest?
- What are the basic operating needs of our software (CPU, memory, network access, disk usage, the performance of downstream dependencies)? How can we measure that those needs are being met?
- What are the performance requirements of my software?

If you’re building a web application or web service, here are some more specific ideas to get you started:

- Tp50, Tp75, Tp90, Tp99, Tp99.99 latency: This is the amount of time the server spends processing each HTTP request, between the time the request arrives at your code, and the time your code generates the response. Slice this per URL pattern and the HTTP verb.  You ask why not [average](https://stackoverflow.com/questions/17435438/what-do-we-mean-by-top-percentile-or-tp-based-latency)?
- Tp50, Tp75, Tp90, Tp99, Tp99.99 latency breakdown: time spent in application code, time spent waiting for the database, cache, or downstream services.
- Sum of requests that result in response code 2xx, 3xx, 4xx, and 5xx. 
- Sum of GET, PUT, POST, DELETE, etc. requests
- Total number of requests
- Time to load, time to interact: Some example of this tweeter’s time to first tweet. Can use Performance API to get this info.
- Tp50, Tp90, Tp99 request, and response payload sizes.
- Tp50, Tp90, and Tp99 gzip compression ratio. How much is gzip helping your clients? Are your payloads actually compressible?
- The number of load balancer spillovers. How often are your inbound requests rejected because your web servers are at capacity?
- Cache hit and miss counts. P0, P50, and P100 for the sizes of objects stored in your cache.
- The basic host metrics: disk utilization, disk I/O rate, memory utilization, and CPU utilization. Support for load average.
- Sum of connections open to each web server, database host, queue server, and any other service you have

## Why to use Alarms?

A metrics without an alarm will kill you with the operational burden of checking it everyday, dooming you to one of two fates:
- You will never look at it
- You will force yourself to review it on a schedule

**How to avoid this?**
For any metric worth tracking, there should be some threshold that causes you to get notified.
Creating good alarms is the key to metric productivity, but it can also bury you in false positives (commonly called “spurious alarms”), eventually causing you to ignore alarms. There are ways to avoid this pitfall:

**Preventing Alarm Noise: Grace periods**
For every alarm, you should not only have a threshold, but also a grace period before the alarm fires. In other words, the metric must show some “sticking power” when it crosses the threshold before it turns into an alarm. For example, when some disk or router stalls out for a few seconds, but recovers on its own, we don’t want our on-call engineer to get paged at 3am.

Take into account the metric sampling period when setting the grace period. If your metric samples every 5 minutes, and you set a 5-minute grace period, you get no benefit.

Many alarm systems will allow you to set up two grace periods: one for getting into an alarm state, and one for getting out of an alarm state. An alarm will only clear when the metric moves back into safe territory, and stays there.

**Preventing Alarm Noise: Daytime Alarms**
Some alarms are not important enough to get out of bed in the middle of night. A daytime alarm will send an alert, but only during normal working hours. If such an alarm happens at night, the system will wait to notify you until the next day. Smart daytime alarms can also avoid weekends!

My team uses daytime alarms for early warning indicators. For example, if available host memory drops, but not dangerously low, the system will send us a daytime alarm. But if available memory drops to a dangerous level, the system will send us a regular alarm, regardless of the time of day or day of week.

## Changing Thresholds

Most metrics have predictable cycles. Perhaps your traffic grows during daytime hours and shrinks during nighttime hours. A flat threshold won’t notify you if there is an unusual traffic spike at night.

But even if it doesn’t make sense to adapt your alarm thresholds based on the time of day, you should still change your thresholds as your software evolves.

## Missing Metrics

It’s natural to think about alarm thresholds, but what if a metric stops reporting? The absence of a metric can be just as bad as a metric that breaches a threshold. For every alarm you create, you should have a corresponding alarm that fires if the metric stops reporting. Like regular alarms, these should also have a grace period to avoid false alarms if the metric recovers on its own (or maybe use a daytime alarm in this case).

## Regularly Review your metrics

Things change. Make sure your alarms change with them. For example, let’s say you recently shipped some new code which drastically improves the runtime speed of your application. You have some alarms that will tell you if performance gets worse, but their thresholds are based on the old (slow) performance. These need to be updated now, but how would you remember that? The answer is that you should build mechanisms to take care of this for you.
