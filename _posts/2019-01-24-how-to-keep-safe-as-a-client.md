---
layout: post
title:  "Enhancing System Stability with Circuit Breaker"
date:   2019-01-27 22:00:00 +0800
categories: blog
author: Joe Zhang
---

[Article Link](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486481&idx=1&sn=87aee20e301d87030be2636cd0a124b7&chksm=9bd0a189aca7289f0a5e8a91907d21e32bd367341251c713e76c2fd97f6f64c06379ad7c4f93&scene=21#wechat_redirect): 

Previously I worked on a project which used many third-party services, these third-party services exposed restful APIs and my project acted as a client.
As such, if some services were broken or had high latency, they would affect our system. Furthermore, they would block user request and could cause the whole system to break down.
Circuit Breaker is a solution for resolving this problem.

![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vSd4AzVWGyFNonEC5CuA1A09St2nUJ20ViaT2CPWl9GEBzuiazRoasZiaKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## What is Circuit Breaker

Circuit Breaker is a protection mechanism. Like switches in electronic systems. If there is something wrong with a part of the circuit, the Circuit breaker will kick in and cut off electricity. Having a Circuit Breaker in a system means that if for some reason the downstream services are broken, the upstream system will cut off connections to the downstream services.

## How to implement a Circuit Breaker

Every system has a limitation because hardware has an upper bound, this is a basic knowledge we need to know when we design and work on a system.

### Define abnormal state

Abnormal state means the system is broken or is having high latency. Generally, how to define an abnormal state is based on two points:
- Can we access the system
- Do we spend much more time than usual when we invoke an interface

Since all systems are built on unstable networks, sometimes the system will unavailable for several seconds. We can't consider such a condition as an abnormal state. So we need to introduce a concept: "Time Window". It means if the system is unavailable many times in a fixed time, we can say the system is in an abnormal state.

We define such a state in two ways.
- Threshold: for example, if the system encounters over 100 "unable to access" response within 10 seconds or encountered over 100 high-latency requests within 5 seconds.
- Percentage: for example, about 30% of request can't get a response or have high latency.

We can use pseudo code to express the definitions above:

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

Once our downstream system is broken, we need to cut off connection instantly. This action can both reduce useless remote call and reduce downstream system's load.
For example, we use the RPC framework to communicate between systems.
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vS47MgWD0aWMEqAYpvxibphTppSvBia6tRo3wpDgBgTKys8MqtgzwcqWXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
ServiceA is a client, If we find ServiceB is unavailable, we can return a failed response within ServiceA and do not invoke ServiceB
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vS7HACeboGiaUlzjRXWJhR2nGMJQIYkfAxZE1TRfTIXME68kC6YPmRaIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
This is so-called "Fail Fast". We can use the code below to implement fail fast.

```python
if(isOpenCircuitBreaker == true){
  return fail;
}

//Invoke remote service here

```
### Try to reconnect

After cutting off connections, we need to recover as soon as possible, because the system is now in an incomplete state. Similar to how we define a system as unavailable, we need to define a standard to view a system as available again.
- Threshold: for example, there are 100 success requests within 5 seconds.
- Percentage: for example, there are 95% success requests within 10 seconds.

The standard also contains a time window, threshold, and percentage. But there is a slight difference: most of the time, the system will stay unavailable for a moment, So we can check in fixed times(30 seconds for example).

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
Since the reconnection can still fail, we cannot use all servers to try to reconnect in a large system. We can take these measures:
- 1. Use a few servers to check
- 2. Add a health check interface to check system conditions: CPU, IO, etc.

### recover

Once the system is available, we need to recover.

![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vSUGPwicRFU6xJeNPxzKOWEhlM3yKSVdEfk1nusm5FfnRm1WyYkOVybBg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

This is the general process of Circuit Breaker, frameworks like hystrix, resilience4j are all had such process flow.

## Best practice for Circuit Breaker

### What kind of situation require Circuit Breaker

Circuit Breaker is not suitable for every situation. There are some conditions we can consider for using Circuit Breaker:
- If the target system is a shared service: when other systems abuse this shared service, our client can be affected.
- The target system is not in regular use, sometimes the system may be under heavy load, while other times it may be rarely invoked, but we need to maintain stable service during sudden spikes in load.

### Things to watch out for when implementing a circuit breaker

Circuit Breaker is not perfect, we need to pay attention to two things.
- Firstly, if we rely on a system that has many replicas, we need to keep in mind that the exception in a partial node does not mean the exception of the whole system.
- Secondly, Circuit Breaker should be the last resort, we can to use "fall back" or "rate limit" prior to Circuit Breaker. Because partial is better than none, although some users may find out that the service is broken, we can try to keep our system working and serve other users. For example, by abandoning non-core services, or by giving friendly hints, we can keep our core services working.

## Conclusion

Circuit Breaker can be implemented before the actual function or after the actual function. It meets the AOP(aspect-oriented programming) concept, so generally, most circuit breaker frameworks use AOP as an implemented strategy.
The Circuit breaker is a shell that protects the system when the error occurs. But in the long run, it is better to have regular pressure tests and a better understanding of the limitations of our system. We should try "fallbacks" or "rate limits" before we implement circuit breakers.
