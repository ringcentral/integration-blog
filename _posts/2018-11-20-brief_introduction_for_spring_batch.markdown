---
layout: post
title: A brief introduction of Spring-Batch in theory
date: 2018-11-20 14:20:22 +0800
tags: [JAVA, Archiver, Spring-Batch]
---

This article will briefly introduce the background, scenarios of Spring-Batch and principles of designing a general batch.  
Also this article can be treat as a reading review of the "Spring Batch - Reference Documentation". What I want to find out is “Whether this framework, that is known for being the industry standard, is suitable for Archiver?” 

#### **BACKGROUND**
Many applications within the enterprise require bulk processing to perform business operations. However, there has been a lack of such a Java-based batch processing framework for years. So, springSource and Accenture collaborated, try to standardized this area. 
In RingCentral the Archiver is also a bulk processing system, to figure out whether Spring-Batch is suitable for Archiver we should know what kind of features that Spring-Batch supply. 

#### **SCENARIOS**
###### Business Scenarios 
 	•	Periodically : can be executed periodically 
 	•	Concurrent : can be processed in parallel lines 
 	•	Staged : enterprise message-driven processing
 	•	Massively : may have big amount of data to process 
 	•	Manual or scheduled : have multiple way to trigger 
 	•	Step by Step : sequential processing of dependent steps 
 	•	Partial processing : allow skiping records 
 	•	Whole-batch transaction : for cases with a small batch size or existing stored procedures/scripts
  
###### Technical Objectives
 	•	Allow developers to concentrate on business logic and let the framework take care of infrastructure 
 	•	Separate the infrastructure and the batch execution environment 
 	•	Provide common/core execution services as interfaces that all projects can implement 
 	•	I'd just say provide simple default implementations of the core execution interfaces that can be used ‘ootb’(out of the box) 
 	•	Easy to configure, customize, and extend services, by leveraging the spring framework in all layers. 
 	•	All existing core services should be easy to replace or extend, without any impact to the infrastructure layer. 
 	•	Provide a simple deployment model, with the architecture JARs completely separate from the application 
As shown above, most of the features match Archiver's needs. So my conclusion is that we can get familiar with this framework first, and use it in some new jobs.In the near future, if this is much easier to maintain and has some optimized database/IOPS practices as claimed in the document, we can use it directly. 
After a short introduction to Spring-Batch, I'd like to ask some questions : How does a normal batch job look like without Spring-Batch? What principles should be followed when designing a batch job?

#### **GENERAL BATCH PRINCIPLES**
 	•	Using common building blocks when possible 
 	•	Simplify as much as possible in one single step 
 	•	Keep processing and storage close together 
 	•	Minimize system resource use, especially I/O 
 	•	Analyze SQL statements to ensure that unnecessary physical I/O is avoided
 	•	Reading data for every transaction
 	•	Rereading data for a transaction 
 	•	Causing unnecessary table or index scans 
 	•	Not specifying key values in the WHERE clause of an SQL statement 
 	•	Do not do things twice in a batch run 
 	•	Allocate enough memory 
 	•	Always assume the worst case 
 	•	Plan and execute stress tests as early as possible 
 	•	Implement checksums for internal validation where possible
 	•	Backups will help, not only database backups but also file backups. File backup procedures should not only be in place and documented but be regularly tested as well.
  
In my opinion we have much room to improve for the last two points. If we implement checksum we can make our system more stable. We can also make estimations of our system so that all hardware resources are controllable before jobs start. 
In the end, practice is the best way to test the theory. So there is no need to follow everything it says. Furthermore, I hope this article will build some common sense and consensus in our daily jobs.


