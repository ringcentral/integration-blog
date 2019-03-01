---
layout: post
title:  "how to keep safe as a client"
date:   2019-01-27 22:00:00 +0800
categories: blog
author: Joe Zhang
---

[Artical Link](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486481&idx=1&sn=87aee20e301d87030be2636cd0a124b7&chksm=9bd0a189aca7289f0a5e8a91907d21e32bd367341251c713e76c2fd97f6f64c06379ad7c4f93&scene=21#wechat_redirect): 

Previously I worked on a project which use many third party services, These third party services exposed restful APIs and my project will act as a client.
In such background, If some services were broken or having high latency, It will affect our system. What's more, It will block user request and may cause the whole system broken down.
In order to resolve such problem, one solution is using Circuit Breaker.

![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vSd4AzVWGyFNonEC5CuA1A09St2nUJ20ViaT2CPWl9GEBzuiazRoasZiaKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## What is Circuit Breaker

Circuit Breaker is a protection mechanism. Like switch blade in electronic system, If there are something wrong with one part, Circuit breaker will open and cut off electric. A Circuit Breaker in a system means: if downstream service is broken for some reason, upstream system will cut off connections with downstream services.

## How to implement a Circuit Breaker

Every system has a limitation because hardware has a upper bound, this is a basic knowledge we need to know when we design and work on a system.

### Define abnormal state

Abnormal state means the system is broken or having a high latency. Generally how to define a abnormal state based on two points:
- Can we access the system
- Do we spend much more time than usual when we invoke a interface

However all systems are built on unstable network, sometimes the system will unavailable for several seconds, But we can't take such condition as abnormal state. So we need introduce a concept: "Time Window". Which means if the system is unavailable many times in a fixed time, We can say system is in a abnormal state.

We define such state in two ways.
- Threshold, for example, if system occurs 100 "unable to access" response within 10 seconds or have 100 times latency within 5 seconds.
- Percentage, for example, about 30% of request can't get response or have a latency.

We can use pseudo code to express above definitions:

```python
errorCount = 0;
isOpenCircuitBreaker = false;
limitCount = 10;

if(success){
  return success;
}else{
  errorCount ++;
  if(errorCount >= limitCount){
    isOpenCircuitBreaker = true;
  }
}

```

### Cut off connection

Once our downstream system is broken, we need to cut off connection instantly, This action can both reduce useless remote call and reduce downstream system's load.
for example, we use RPC framework to communicate between systems.
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vS47MgWD0aWMEqAYpvxibphTppSvBia6tRo3wpDgBgTKys8MqtgzwcqWXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
ServiceA is a client, If we find ServiceB is unavailable, we can return failed response within ServiceA and do not invoke ServiceB
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vS7HACeboGiaUlzjRXWJhR2nGMJQIYkfAxZE1TRfTIXME68kC6YPmRaIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
This is so called "Fail Fast". We can use below code to implement fail fast.

```python
if(isOpenCircuitBreaker == true){
  return fail;
}

//Invoke remote service here

```
### Try to reconnect

After cut off connections, We need to recover as soon as possible, Because currently the system is not complete. In order to automatically recover connection we need to define a standard, like previously how we judge a system is unavailable, we need to define whether a system is available again.
- Threshold, for example, there are 100 success requests within 5 seconds.
- Percentage, for example, there are 95% success requests within 10 seconds.

It also contains time window, threshold and percentage. But there is a slight difference: most of the time, the system will keep unavailable for a moment, So we can check in a fixed times(30s for example).

```python
successCount = 0;
isHalfOpen = true;

if(success){
  if(isHalfOpen){
    successCount ++;
    if(successCount == threshold){
      isOpenCircuitBreaker = false;
    }
  }
  return success;
}else{
  errorCount ++;
  if(errorCount == threshold){
    isOpenCircuitBreaker = true;
  }
}

```
However reconnect may failed, In a large system we can't use all servers to try reconnecting. We can take these measures:
- 1.Use a few servers to check
- 2.Add a health check interface to check system conditions: CPU, IO for example.

### recover

Once the system is available, we need to recover.

![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vSUGPwicRFU6xJeNPxzKOWEhlM3yKSVdEfk1nusm5FfnRm1WyYkOVybBg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

This is the general process of Circuit Breaker, Frameworks like hystrix, resilience4j are all have such process flow.

## Best practice for Circuit Breaker

### What kind of situation require Circuit Breaker

Circuit Breaker is not suitable in every situations, There are some conditions we can consider using Circuit Breaker:
- The system we rely on is a shared system, and our client is just one of them. Other systems may abuse the shared system and affect us.
- Our system is not used in a regular base, Somethings the system may used very frequently while other times it is rarely used, we need to hold on the sudden flow.

### Pay attention to implement a Circuit Breaker

Circuit Breaker is not perfect, we need to pay attention to two things.
- Firstly, if we rely on a system that has many replicas, we need to keep in mind that exception in partial node does not mean exception of the whole system.
- Secondly, Circuit Breaker should be the last choice, We can to use "fall back" or "rate limit" prior than Circuit Breaker. Because partial is better than none, Although some users may find out service broke, we can try to keep our system working and serve other users. For example, abandon non-core services, or give a friendly hint.

## Conclusion

Circuit Breaker can be implemented before actual function or after actual function. It is very correspond to AOP concept, so generally most circuit breaker framework  use AOP as an implement strategy.
Circuit Breaker is a shell, which protect itself when error occurs. But in the long term is better to have regular pressure testing and understand limitations of our system, firstly we try "fall back" or "rate limit" then we can use Circuit Breaker.
