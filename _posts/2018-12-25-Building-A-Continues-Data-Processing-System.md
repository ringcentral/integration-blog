---
layout: post
title:  "Building a continues data processing system"
date:   2018-12-25 23:00:00 +0800
categories: [blog]
tags: [elixir, backend, back-pressure]
author: Jun Lin
---

## A brief introduction to Archiver

At RingCentral, we have a system called **Archiver**, which is a system automatically uploads customer’s call recordings, SMS and fax to their cloud storages.

For example, a company connects its Dropbox account with Archiver, then whenever its employees make phone calls and recording the audio, Archiver will upload the call logs to the customer’s Dropbox storage.

It seems like a really simple system, but it is still challenging to implement it *correctly*. The heavy lifting of Archiver is its background workers, as some of our customers are large companies like airline companies, they may generate many call recordings in a day.

To make sure the customer’s call recordings can be uploaded in time, our system starts an archiving job every 5 minutes. The job will first iterate all customers to find out how many items are pending to upload, then bootstrap a thread per customer to upload the call logs. For each customer, we then start a new thread per item to upload it to the Cloud Storage. It’s running pretty well, but as RingCentral are rapidly growing, we have encountered some issues.

## What’s the problem

Recently, we found many HTTP 429 errors in our logs and the whole data processing rate is slowing down. The root cause is a customer with thousands of employees has connected Archiver to their Dropbox account, and they have hundreds of thousands of calls a day, so, with the Scheduler running every 5 minutes, we may hit the rate limit of Dropbox.

So, to make sure the customer’s recordings will be uploaded, we have to estimate the amount of data we can process in one batch. It's affected by how frequently we start a job, how much data we need to fetch in a batch, how many threads we can run at the same time, and also the cloud storage’s rate limiting. We can eventually set a reasonable number by doing some calculation to ensure customer’s data will be uploaded eventually, but there are the complexity of the network, database IO and other bottlenecks, so we definitely need to find a better solution.

## A introduction to GenStage

While we were looking for a solution, a talk [GenStage and Flow](https://www.youtube.com/watch?v=XPlXNUXmcgE) by José Valim came to my mind. It should provide a perfect solution to our problem.

So what’s **GenStage**? From the project’s [README.md](https://github.com/elixir-lang/gen_stage), it describes itself as:

> a specification for exchanging events between producers and consumers.

But what does that mean? It means GenStage provide us a toolkit to define a data processing pipeline, which is assembled by stages, and events flow between these stages.

Let’s try to simulate a simple pipeline:

```
[A] -> [B] -> [C]
```

In this example, the `A` is a producer, `B` is a consumer, also a producer, `C` is a consumer. `A` produce some events that consumed by `B`, `B` do some calculation, transform the events into new events which are consumed by `C`.

Until now, it seems that GenStage is just another boring Cron-like system, a message queue or something. But they are actually different.

With GenStage, the producers wait for demand from consumers, the consumer sends the demand to it’s upstream when the producer receives demand, it generating some events, then pass it to the consumer. With this mechanism, we can achieve a back-pressure data processing system,

## An Example

Let’s try to simulate a simple Archiver implementation: A system that fetches items from the database, then upload these items to Dropbox.

Let’s describe this system in GenStage:

```
Database --> [Fetcher] --> [Uploader] --> Dropbox
                 |             |
                 |             |
             (producer)    (consumer)
```

In this example, the `Fetcher` is a producer, `Uploader` is a consumer. `Fetcher` fetches items from the database that consumed by `Uploader `, `Uploader` then upload these items to Dropbox.

The `Fetcher` will wait for demand from `Uploader`, and the `Uploader` consumes items from Fetcher.

What does it mean for a producer to **wait** for demand? Instead of having a scheduler to bootstrap a job every 5 minutes, the consumer will send the demand to it’s upstream, when the producer receives demand, it starts generating events, then pass it down to the consumer.

This facility is known as a back-pressure mechanism. Back-pressure make sure that the producers will not flood the consumers with events when the consumers are busy.

## See it in Action

Let’s see how to implement this system in GenStage.

Fetcher:

{% gist 3cf6275c8823310f7c33539753d9500a fetcher.ex %}

Uploader:

{% gist 3cf6275c8823310f7c33539753d9500a uploader.ex %}

When Unloader is started, it subscribes to the Fetcher and sends demand for items. The `handle_demand` callback is called, items are retrieved from the Database and passed over to `handle_events` in Uploader, which starts Tasks to process these items.

With this implementation, when we hit the rate limit of Dropbox, the Uploader will tell the Fetcher: *Hey, stop, I cannot process any more items*, then the Uploader will stop reading more items from the database. Once the Uploader successfully uploaded some items, it demands Fetcher for more items, then Fetcher will start reading demanded items from the database, then pass these items to Uploader to continue the uploading.

## Moving forward

We have seen the power of the GenStage. With only 34 lines of code, we implemented a continues data processing system, which has a back-pressure mechanism to ensure the system is running at the maximum processing rate.

On the Java land, there are [Akka Stream](#) and [RxJava](#) have a similar back-pressure mechanism. I hope we can introduce them to Archiver soon.
