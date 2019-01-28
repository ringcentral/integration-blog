---
layout: post
title:  "how to keep safe among distributed systems"
date:   2019-01-27 22:00:00 +0800
categories: blog
author: Joe Zhang
---

When we develop on a distributed systems, generally we need to rely on functions/interfaces provided by other systems, since most large systems will split into many smaller systems and expose a restful API or RPC services to be used by other systems. we use such technique because it will make our system more flexible and easy to maintain. 
In such background, If some services break down or have high latency, it will affect other services. what's more, it will block user request and may cause the whole system broken down.
In order to resolve this problem, one solutino is to use Circuit Breaker.

![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vSd4AzVWGyFNonEC5CuA1A09St2nUJ20ViaT2CPWl9GEBzuiazRoasZiaKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## What is Circuit Breaker

Circuit Breaker is a protection mechanism, like switch blade in a electronic system, if there are something wrong with the system, it will open and cut off electric. A Circuit Breaker in a system means: if downstream service is broken down for some reason, upsrteam system will cut off connections with downsream services.

## How to Implement a Circuit Breaker

Every system has a limitation because hardware has a upper bound, this is a basic knowledge we need to know when we working on a system. especially sometimes our system design didn't take full use of the hardware.

### Define abnormal state

Abnormal state means the system is broken down or having a high latency.
Generally, how to define a abnormal state based on two points:
- Can we access the system
- Do we spend much more time than usual when we invoke a interface

However all systems are built on a unstable network, so sometimes the system will unavaiable for several seconds, But we can't take this such condition as unabnormal state. So we need to use "Time Window". Which means if the system is unavaliable many times in a fixed time, We can say system is in a unabnormal state.

We define such state in two ways.
- Threshold, for example, if system occures 100 "unable to access" response within 10 seconds or have 100 times latency within 5 seconds.
- Percentage, for example, about 30% of request can't get response or have a latency.

We can use pseudo code to express above definitions:

```python
errorCount = 0;
isOpenCircuitBreaker = false;
limitCount = 10;

if(success){
  return success;
}else{
  errorCount++;
  if(errorCount >= limitCount){
    isOpenCircuitBreaker = true;
  }
}

```

### Cut off connection

Once our downstram system is broken, we need to cut off connection instantly, This action can both reduce useless remote call and reduce downstream system's load.
for example, we usually use RPC framework to communicate between systems.
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vS47MgWD0aWMEqAYpvxibphTppSvBia6tRo3wpDgBgTKys8MqtgzwcqWXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
ServiceA is a client, If we find ServiceB is unavaliable, we can return failed response within ServiceA and do not call ServiceB
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vS7HACeboGiaUlzjRXWJhR2nGMJQIYkfAxZE1TRfTIXME68kC6YPmRaIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
this is so called "Fail Fast". we can use these code to implement Fail Fast.

```python
if(isOpenCircuitBreaker == true){
  return fail;
}

// we invoke remote service here

```
### Try to reconnect

After cut off connections, We need to recover as soon as possbile, Because currently the system is not complete. How to automatically recover is a new problem.
Generally we need to define a standard, like previously how we judge a system is unavaliable, we need to define whether a system is abaliable again.
- Threshold, for example, we invoke our downstram system successfully for 100 times within 5 seconds.
- Percentage, for example, there are 95% success request within 10 seconds.

It also contains time window, threshold and percentage.
but there is a slight difference, most of the time, the system will keep unavalible for a momment, So we can check in a fixed times, For example, we can try to connect other system every 30 seconds. and we can check in a dynamic time range.

```python
successCount = 0;
isHalfOpen = true;

if(success){
  if(isHalfOpen){
    successCount++;
    if(successCount == threshold){
      isOpenCircuitBreaker = false;
    }
  }
  return success;
}else{
  errorCount++;
  if(errorCount == threshold){
    isOpenCircuitBreaker = true;
  }
}

```
We need to know that try to reconnect may be failed, In a large systems we can't use all servers to try reconnecting.
- 1.Use a few servers to check
- 2.Add a health check interface to judge system conditions: CPU, IO for example.

### recover

Once the system is avaliable, we need to recover.
![](https://mmbiz.qpic.cn/mmbiz_png/oB5bd6W6hI31YcnTS8xgHlND7GJnk9vSUGPwicRFU6xJeNPxzKOWEhlM3yKSVdEfk1nusm5FfnRm1WyYkOVybBg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
This is the general process of Circuit Breaker, there are some framework can do this, for example Hystrix, resilience4j.

## Best practise for Circuit Breaker

### What kind of situation need Circuit Breaker

Circuit Breaker is not suitable in every situations, There some conditions suitable to use Circuit Breaker:
- The system we rely on is a shared system, and our client is just one of them. Other systems may abuse the shared system and affect us.
- Our system is not used in a regular base, Somethings the system may used very frequently while other times it is seldomly used, we need to hold on the sudden flow.

### Pay attention to implement a Circuit Breaker

Circuit Breaker is not perfect, we need to pay attention to two things.
- Firstly, if we rely on a system that has many replicas, we need to keep in mind that exception in partial node does not mean exception of the whole system.
- Secondly, Circuit Breaker should be the last choice, We can to use "Fall Back" or "Ratelimit" prior than Circuit Breaker. Because partical is better than none, Although some users may find out service broke, we can try to keep our system working and serve other users. For example, abandon non-core services, or give a friendly hint.

## Conclusion

Circuit Breaker can be implemented before actual function or after actual function. it is very correspond to AOP concept, so generally most Circuit Braker framework  use AOP as a implement strategy.
Circuit Breaker is a shell, which protect itself when error occurs. But in the long term is better to have regular pressure testing and understand limitations of our system, firstly we try "Fall Back" or "RateLimit" then we can use Circuit Breaker.
