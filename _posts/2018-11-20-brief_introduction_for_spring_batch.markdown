---
layout: post
title: A brief introduction of Spring-Batch in theory
date: 2018-11-20 14:20:22 +0800
tags: [JAVA, Archiver, Spring-Batch]
---
This article will briefly introduce the background, scenarios of Spring-Batch and principles of designing a general batch.  
Also this article can be treat as a reading review of the Spring Batch - Reference Documentation, All I want to find out is “Whether Archiver is suitable for this so called industry standard framework?” 
**BACKGROUND**
Many applications within the enterprise require bulk processing to perform business operations. However there has been a lack of such a Java-based batch processing framework for years. So springSource and Accenture collaborated, try to standardized this area. 
In RingCentral the Archiver is also a bulk processing system, to figure out whether Archiver is suitable we should know what kind of features that Spring-Batch supply. 
**SCENARIOS**
Business Scenarios 
 	•	Periodically : allow to be execute periodically 
 	•	Concurrent : can be process in parallel lines 
 	•	Staged, enterprise message-driven processing
 	•	Massively : may have big amount of data to process 
 	•	Manual or scheduled : have multiple way to trigger 
 	•	Step by Step : sequential processing of dependent steps 
 	•	Partial processing: allow skip records 
 	•	Whole-batch transaction, for cases with a small batch size or existing stored procedures/scripts
Technical Objectives 
 	•	Allow developer concentrate on business logic and let the framework take care of infrastructure 
 	•	Separate the infrastructure, the batch execution environment 
 	•	Provide common, core execution services as interfaces that all projects can implement 
 	•	Provide simple and default implementations of the core execution interfaces that can be used ‘ootb’(out of the box) 
 	•	Easy to configure, customize, and extend services, by leveraging the spring framework in all layers. 
 	•	All existing core services should be easy to replace or extend, without any impact to the infrastructure layer. 
 	•	Provide a simple deployment model, with the architecture JARs completely separate from the application 
As above, most of the features matches Archiver. So my conclusion is we can get familiar with this framework first, and use it in some new jobs. In the nearly future if this is much more easier to maintain and it has some optimized database / IOPS practice as it says in the document. We can use it entirely and directly. 
After a short introduce of Spring-Batch I like to ask a question. Regardless of Spring-Batch how a normal batch job looks like? What principle should follow if you are designing a batch job?
**GENERAL BATCH PRINCIPLES**
 	•	Using common building blocks when possible 
 	•	Simplify as much as possible in one single step 
 	•	Keep processing and storage close together 
 	•	Minimize system resource use, especially I/O 
 	•	Analyze SQL statements to ensure that unnecessary physical I/O is avoided 
 	◦	Reading data for every transaction 
 	◦	Rereading data for a transaction 
 	◦	Causing unnecessary table or index scans 
 	◦	Not specifying key values in the WHERE clause of an SQL statement 
 	•	Do not do things twice in a batch run 
 	•	Allocate enough memory 
 	•	Always assume the worst case 
 	•	Plan and execute stress tests as early as possible 
 	•	Implement checksums for internal validation where possible
 	•	Backups will help, not only database backups but also file backups. File backup procedures should not only be in place and documented but be regularly tested as well.
In my opinion we have much space to improve for the last two points. If we implement checksum we can make our system more stable. We also able to make estimate of our system so that all hardware resources are controllable before jobs start. 
In the end, practice is the best way to test the theory. So there is no need to follow everything it says. Further more I hope this article build some common sense and consensus in our daily jobs.


