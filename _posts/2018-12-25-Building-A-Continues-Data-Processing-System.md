---
layout: post
title: "Building a continues data processing system"
date: 2018-12-25 23:00:00 +0800
categories: [blog]
tags: [elixir, backend, back-pressure]
author: Jun Lin
---

## A brief introduction to Archiver

At RingCentral, we have a system called **Archiver**, which is a system automatically uploads customer’s call recordings, SMS and fax to their cloud storages.

For example, a company connects its Dropbox account with Archiver, then whenever its employees make phone calls and recording the audio, Archiver will upload the call logs to the customer’s Dropbox storage.

It seems like a really simple system, but it is still challenging to implement it _correctly_. The heavy lifting of Archiver is its background workers, as some of our customers are large companies like airline companies, they may generate many call recordings in a day.

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

In this example, the `Fetcher` is a producer, `Uploader` is a consumer. `Fetcher` fetches items from the database that consumed by `Uploader`, `Uploader` then upload these items to Dropbox.

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

With this implementation, when we hit the rate limit of Dropbox, the Uploader will tell the Fetcher: _Hey, stop, I cannot process any more items_, then the Uploader will stop reading more items from the database. Once the Uploader successfully uploaded some items, it demands Fetcher for more items, then Fetcher will start reading demanded items from the database, then pass these items to Uploader to continue the uploading.

## Moving forward

We have seen the power of the GenStage. With only 34 lines of code, we implemented a continues data processing system, which has a back-pressure mechanism to ensure the system is running at the maximum processing rate.

On the Java land, there are [Akka Stream](#) and [RxJava](#) have a similar back-pressure mechanism. I hope we can introduce them to Archiver soon.

---

---

# 使用 GenStage 构建一套稳定的持续数据处理系统

## 简单介绍一下 Archiver

在 RingCentral，我们有一个名叫 Archiver 的系统，它能够自动将客户的通话录音，短讯和传真讯息同步到客户连接的云盘上。

例如，一家公司将 Archiver 和他们自己的 Dropbox 账户相关联之后，当这家公司的员工拨打了录音电话，Archiver 就会自动将这些通话录音同步到客户的 Dropbox 里。

初看起来，Archiver 似乎是一个很简单的系统，但是要把它*正确*地实现，还是很富有挑战性的。我们的客户里有很多像航空公司这样的大型公司客户，一家公司一天能够产生数万条通话录音，这些录音文件都需要被及时同步到客户连接的云盘上，因此 Archiver 最核心的部分是就是那些后台运行着的工作进程。

我们的后台工作进程使用了调度的方式，每五分钟就会启动一次存档的工作。每次工作执行的时候，会先遍历所有的客户，找到需要同步的文件信息，然后对每个文件开启一个线程来同步文件。使用这种机制，整个系统运行良好，客户的信息都能够及时同步完成；但是 RingCentral 的客户正在高速增长，有一些问题正在慢慢暴露出来。

## 问题出在哪？

就在近期，我们发现了整体的数据处理速率变慢了，通过查看日志文件，发现日志里有很多 HTTP 429 (Too many requests) 类型的错误。通过调查后，发现是最近有许多大体量的公司开始使用 Archiver 关联了它们的 Dropbox 账户，它们可能有上千名员工，每个员工每天产生几百通的电话录音。文件量变大之后，我们每五分钟执行一次同步的操作，轻易就触碰到了 Dropbox 的同步速率上限。

使用现有的机制，为了确保客户的文件可以及时被同步成功，我们必须对 Archiver 可以处理的数据总量做一次评估。影响处理速率的因素有：触发同步操作的频率、每次同步的总量、Dropbox 的速率上限等等。我们最终估算了一个比较合理的数值，确保了客户的文件可以最终被同步成功。但是因为网络，数据库 IO 的瓶颈，以及其他各种不确定性，我们急需一个更健壮的解决方案。

## GenStage 初探

在我们团队寻找解决方案的时候，我想起了 José Valim 2017 年的演说 [GenStage and Flow](https://www.youtube.com/watch?v=XPlXNUXmcgE)，发现这是一个符合 Archiver 的解决方案。

所以，什么是 **GenStage** 呢？项目的[简介](https://github.com/elixir-lang/gen_stage)里面描述自己为：

> GenStage is a specification for exchanging events between producers and consumers.

字面翻译过来是：「生产者和消费者之间交换事件的一套规范」，这是什么意思呢？**GenStage** 提供给我们的是一套用来实现「事件流」处理的工具箱，它由「阶段」和各个「阶段」之间的「事件流」组成。

让我们来模拟这样一个场景：

```
[A] -> [B] -> [C]
```

上图里，`A` 是一个「生产者」；`B` 是一个「消费者」，同时也是「生产者」；`C` 则是「消费者」。`A` 会生成「事件」，然后传递给 `B` 消费，`B` 将这些「事件」进行一系列的计算、变换之后，生成新的「事件」，并传递给 `C` 消费。

咋一看上面的场景，只是又一个无聊的定时任务或者一个消息队列系统，但是，GenStage 的实现和它们有本质的不同：GenStage 里的「生产者」会等待「消费者」的请求。

那么，**等待** 请求是什么意思呢？相较于使用一个调度器来每五分钟启动一次「生产者」来产生事件，GenStage 独特的实现方式是：「生产者」不会直接去获取数据，而是会等待「消费者」的请求，当「生产者」收到「消费者」的请求之后，「生产者」才会去生成「事件」，然后把这些「事件」传递给「消费者」。

这种方式就实现了「背压」的机制，这种机制保证了当「消费者」繁忙的时候，「生产者」不会持续再往「消费者」端发送数据，造成「消费者」处理不了而崩溃，这样就保证了整个数据处理系统可以稳健地以最大的处理速率运行。

## 举个例子 🌰

假设构建这样一个应用：数据库中持续有客户的文件记录写入，需求是构建一个数据处理系统，从这个数据库中将这些文件同步至客户的 Dropbox。

所以，我们可以这样描绘这个系统：

```
Database --> [Fetcher] --> [Uploader] --> Dropbox
                 |             |
                 |             |
             (producer)    (consumer)
```

上图里，Fetcher 先从数据库里取出记录，然后 Uploader 将这些记录又同步到 Dropbox。

## 眼见为实

我们来看一看如何使用 GenStage 实现这样一个系统：

Fetcher

```elixir
defmodule Archiver.Fetcher do
  use GenStage

  def start_link(args) do
    GenStage.start_link(__MODULE__, args, name: __MODULE__)
  end

  def init(state), do: {:producer, state}

  def handle_demand(demand, state) do
    items = Database.get_items()

    {:noreply, items, state}
  end
end
```

Uploader:

```elixir
defmodule Archiver.Uploader do
  use GenStage

  def start_link(args) do
    GenStage.start_link(__MODULE__, args, name: __MODULE__)
  end

  def init(_state) do
    {:consumer, :the_state_does_not_matter, subscribe_to: [Archiver.Fetcher]}
  end

  def handle_events(items, _from, _state) do
    items
    |> Enum.map(&Task.start_link(Dropbox, :upload, [&1]))

    {:noreply, [], :the_state_does_not_matter}
  end
end
```

当 Uploader 开始运行的时候，它将会订阅到 Fetcher，并且发送消息请求，这时候，Fetcher 的 `handle_demand/2` 将会被调用，然后 Fetcher 就从数据库里取出数据，传递给 Uploader 的 `handle_events/3`，接着 Uploader 将会使用 Task 来处理这些数据。

当我们的系统处理速率超过了 Dropbox 的限速，Uploader 就会告诉 Fetcher：_停一停，我无法处理更多的内容了_，这时，Fetcher 就会暂停从数据库读取更多内容；等待 Uploader 消化完已有的内容，整个系统又继续执行，这样就不会导致系统过载而崩溃。

以上，只用了 34 行代码，我们就使用 GenStage 实现了一个简洁的，稳健的数据处理系统：通过背压的机制，使系统得以以最大的处理速率运行。我们无需估算整个系统的处理能力，来决定要多久执行一次同步工作，只需定义好整个数据处理流程和边界即可。

## 更进一步

由于 Archiver 现在依然是使用 Java 作为主要语言的，所以之后希望我们可以引入 [Akka Stream](https://doc.akka.io/docs/akka/2.5/stream/) 或者 [RxJava](https://github.com/ReactiveX/RxJava) 来实现更佳的数据处理系统。

## 链接

-   [RingCentral Archiver](https://www.ringcentral.com/apps/ringcentral-archiver)
-   [ElixirConf 2016 - Keynote by José Valim](https://www.youtube.com/watch?v=srtMWzyqdp8&feature=youtu.be&t=244)
-   [GenStage](https://github.com/elixir-lang/gen_stage)
-   [示例代码](https://gist.github.com/linjunpop/3cf6275c8823310f7c33539753d9500a)
