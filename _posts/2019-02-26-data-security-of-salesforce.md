---
layout: post
title:  "Data Security in Salesforce"
date:   2019-02-26 22:27:00 +0800
categories: blog
author: Harry Hou
---

[Articles](https://trailhead.salesforce.com/content/learn/modules/data_security)

## Data Security in Salesforce

As a CRM application, it is important for the user to give the right people access to the right data. This post will introduce how to manage access control in Salesforce.

### Levels of data access

In Salesforce, you can control the level of access of your users to the data in your whole org, down to specific objects, fields, or even inidividual records.

#### Control Access to the Organization

For data security at organization level, what you can do is to ensure that only employees who meet certain criteria are allowed to log in to Salesforce. You can do this by managing authorized users, setting password policies, and limiting when and where users can log in.

#### Control Access to Objects

In Salesforce, an object is a collection of records, like leads or contacts. So how to control access to objects? Here we must talk about **profiles**.

A profile is a collection of settings and permissions. Every user in Salesforce has his own profile, You can set object permissions with profiles or permission sets to control whether a group of users can create, view, edit, or delete any records of that object.

#### Control Access to Fields

After controlling `object-level` access, defining `field-level` security for sensitive fields is also an important part of Salesforce data security.

In some cases, you want users to have access to an object, but to limit their access to individual fields in that object. You can apply field settings by modifying profiles or permission sets just like the access control of objects.

#### Control Access to Records

As I mentioned before, you can let particular users view specific fields in a specific object by setting profiles and permission sets. But how to control the individual records that they're allowed to see?

In Salesforce you control record-level access in four ways, `org-wide defaults`, `role hierarchies`, `sharing rules` and `manual sharing`.

![](/integration-blog/assets/2019-02-26-data-security-of-salesforce/data_security_records.jpeg)

`Org-wide defaults` specify the default level of access for each records, if you set this to *public*, all users can view these records. However, if you set this to *private*, whether the users other than the record owner can see the records is controlled by `role hierarchies` and `sharing rules`.

`Role hierarchies` ensure managers have access to the same records as their subordinates. Each role in the hierarchy represents the level of data access for a user or a group of users.

`Sharing rules` and `manual sharing` give some user access to records they don’t own or can’t normally see.

### Example

If company X has 5 sales teams, each team has 1 manager, and there is 1 super manager for all managers.

Role hierarchy should be: 

```
 Super manager 
       Manager1 <- Team1
       Manager2 <- Team2
       Manager3 <- Team3
       Manager4 <- Team4
       Manager5 <- Team5
```

There should only be 3 profiles:

  - **Sales Team** (Assigned to all Teams)
  - **Manager** (Assigned to all Managers)
  - **Super Manager** (Assigned to super manager) 

  
### Conclusion

Salesforce platform provides a flexible, layered sharing model. It is easy to assign different data sets to different sets of users. But we should think more before we design and implement our data model.
