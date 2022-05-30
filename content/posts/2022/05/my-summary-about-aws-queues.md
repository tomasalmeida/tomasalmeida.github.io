---
title: "My summary about AWS queues"
date: 2022-05-29T12:40:26+02:00
draft: false
tags: ["aws", "queue"]
categories: ["development"]
ShowToc: true
TocOpen: true
weight: 2
---

I was doubting about writing this post because AWS is evolving really fast and this post will be probably old very fast. So, I am adding the AWS documentation, so, please take your time to check the official documentation to confirm my findings (and if possible, write me a message so I can update it if needed).

## SQS
AWS SQS is a simple 1:1 queue, so it delivers messages from A to B.
* It is a pull service, that means SQS keeps the message and waits the consumer pulls the messsage out.
  * Retention period is 14 days
* Two types defined
  * Standard
  * FIFO (order guaranteed)

### Usages

* Great for decoupling and Lambda usage. If we have many queues maybe EventBridge is a better option.
* Better to use queues on internal microservice communication (between Lambda communication) and EventBridge for microservices external communication.
* To connect two systems with different throughput capabilities: the queue will make it easier for the receiver. For example connecting a new service with a legacy service.
* Use to batch jobs
  * receiving votes -> SQS -> counting votes

### Documentation
[Official documentation](https://aws.amazon.com/sqs/)

## SNS
Fully managed pub/sub messaging service.
* 1 to many, one publish the message, many can subscribe/consume it.
* Push service - messages are pushed into the consumer, messages cannot stay in the queue.
* Message filtering: to reduce the amount of message you consume.

### Usages
* Fan Out pattern: As SNS pushes the message to the consumer, we can put a SQS as consumer, so the consumer can pull the message from the SQS.
  * SNS -> SQS -> consumer.
* Allow parallelization of tasks
* Different queues for all the services subscribed to the topic
* Event Driven application: services listening for other services
* Send emails, text messages, etc.

### Documentation
[Official documentation](https://aws.amazon.com/sns/)

## Amazon EventBridge
Serverless event buss to connect applications with data from several sources. 
* Many to many: several sources can generate messages that can be consumed by several consumers
* Event bus
* Not the highest throughput
* Based on filters (rules):
* Push service - messages are pushed into the consumer, messages cannot stay in the queue.
* Events can be aws, 3rd party or custom events

### Usages
* Decoupling microservices
* Event Driven applications
* Get events from other AWS accounts
* Integrating with many SaaS providers that are able to connect to EventBridge

### Documentation
[Official documentation](https://docs.aws.amazon.com/eventbridge)

## Kinesis
High throughput for data streams in real time
* Many to many: several sources can generate messages that can be consumed by several consumers
* Highest throughput
* Stored and replayed (7 days)

### Usages
* Ingest a lot of data and real time analytics
* Gaming generates tons of events so usually they send petabytes of data so game creators can analyze players behavior and usage and improve the game

### Documentation
[Official documentation](https://docs.aws.amazon.com/kinesis)

## Unofficial documentation
There are tons of documents and videos about this subject, I do like this:
* https://www.youtube.com/watch?v=Id2IVs1f7S0

---
If you found any error or want to help me to improve this article, please [comment the pull request](https://github.com/tomasalmeida/tomasalmeida.github.io/pull/6).