---
layout: post
title:  "How to do quick-review of tickets and bugs before release?"
date:   2018-12-06 22:49:39 +0800

---
As we all know, before GA, there are tickets or bugs that we need to decide if they should be fixed or lower their priority or translate to Improvement.
 This essay is for improving our efficiency toward review ticket.

Before we discuss this topic, I want to share my understanding toward the definition of Bug, P3 bug and improvement.

In my opinion, bug means current flow or operation will cause customer do mistake during work, which means our product not only lowers customer's efficiency, but also leads to customer's trouble.

**The examples:**
  * user can not dial out a phone number
  * the task that user is editing is overwrote by us, or at call    
  * At Call History or Active Call List, our product displays wrong phone numbers or wrong call direction/duration.


So following this definition, we can have a better understanding toward P3 bug and improvement.
As we know, P3 bug means it is a bug but not so important.

BUT what does 'not so important' mean?
It means it will cause user do mistake but the frequency of mistake is hardly.
Improvement means it will not cause user do mistake but if we have this improvement, we can save user's time and improve user's efficiency.
So if UI is probably not so beautiful, the feature is not so fancy, but it doesn't effect user's usage of this feature, it should belonged to an improvement not a bug.

According to above definitions, we can have a roughly estimation towards those tickets.
We assume total points is 10, we take apart 10 points as 6 points and 4 points.
6 points aim to feature, 4 points aim to UI. ( Because we think functionality is more important than UI display.)

**From functional point,** We will evaluate a ticket as 
* 5-6 points if this function/feature is very important to user
* 3-4 points if this function/feature is nice to have
* 1-2 points if this function/feature bring little effect

**From UI display,** We will evaluate a ticket as 
* 3-4 points if this UI display effects user's usage
* 1-2 points if this UI display doesn't effect user's usage

Now we add functional point and UI display point, we can get a total point.
* If the points >= 6, it means the ticket needs to fix at current release
* If the points is in 4-5,  it means the ticket doesn't need to fix at current release(It can be put to feature release)
* If the points is in 4-5,  it means the ticket doesn't need to fix at current release, and engineer can resolve this ticket as won’t fix directly

    
In my opinion, if engineer, QA, PM can reach this agreement, maybe we can handle those tickets more efficently.