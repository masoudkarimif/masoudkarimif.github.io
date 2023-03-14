---
date: "2023-03-14"
title: "Schedule Lambda Functions with AWS EventBridge"
tags: ["aws", "lambda", "eventbridge", "terraform"]
author: "masoudkf"
description: "There are times when you want to run your Lambda function on a pre-defined schedule: every hour, every day, every Tuesday at 5 PM. AWS EventBridge helps us do that."
---

## EventBridge
_"EventBridge is a serverless service that uses events to connect application components together."_ Most AWS services dispatch events in different situations. For instance, ECS dispatches an event when a task starts and Batch dispatches an event when a job succeeds or fails. With EventBridge, we can react to these events and build what they call an Event-driven architecture: a system that generates and reacts to certain events. For instance, we can invoke a Lambda function when a Batch job fails.

But besides events generated from AWS services, we can also create our own events that happen on a pre-defined schedule:
- an event that happens every hour at the 15 minutes of the clock (1:15, 2:15, ...)
- and event that happens every Friday at midnight
- ...

These events, just like the events emitted from AWS services, can be hooked up to a Lambda function. Therefore, we can run our Lambda function automatically based on a schedule. The most popular type of defining a schedule is via _cron_ expressions. _cron_ is a tool available on Unix-based systems that enable users to schedule a job. It has a certain format that must be followed:

```bash
<minute> <hour> <day-of-month> <month> <day-of-week>
```

The cron format on AWS, however, has an extra field at the end that defines the year:
```bash
<minute> <hour> <day-of-month> <month> <day-of-week> <year>
```

Explaining the cron syntax is not what this post is about. You can play around with it [here](https://crontab.guru/), though. 

## Shedule a Lambda Function with EventBridge using Terraform
Here, we create a Lambda function and have EventBridge invoke it every 5 minutes:


{{< gist masoudkarimif 8e07ee483328112dbd27b930b3250475>}}
