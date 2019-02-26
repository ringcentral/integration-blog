---
layout: post
title:  "Data Security Of Salesforce"
date:   2019-02-26 22:27:00 +0800
categories: blog
author: Harry Hou
---

[Articles](https://trailhead.salesforce.com/content/learn/modules/data_security)

## Data Security of Salesforce

As a CRM application, it's important for the user that giving the right people access to the right data, this post will introduce how to control those stuff things in Salesforce.

### Levels of data access

In Salesforce you can control which users have access to which data in your **whole org**, a specific **object**, a specific **field**, or an individual **record**.

#### Control Access to the Organization

At the organization level's data security there has one thing you can do is that ensure only employees who meet certain criteria can log in to Salesforce. In Salesforce you can do this by managing authorized users, setting password policies, and limiting when and where users can log in.

#### Control Access to Objects

In Salesforce an object is a collection of records, like leads or contacts. So how to control access to objects? Here we must take about **profile**.

A profile is a collection of settings and permissions. Every user in Salesforce has his own's profile, You can set object permissions with profiles or permission sets to control whether a group of users can create, view, edit, or delete any records of that object.

#### Control Access to Fields

After controlling `object-level` access, defining `field-level` security for sensitive fields is also an important part of Salesforce data security.

In some cases, you want users to have access to an object, but limit their access to individual fields in that object. You can apply field settings by modifying profiles or permission sets just like the control in objects.

#### Control Access to Records

As just said, you can let particular users view specific fields in a specific object by set profiles and permission sets. But how to restrict the individual records they're allowed to see?

In Salesforce you control record-level access in four ways, `Org-wide defaults`, `Role hierarchies`, `Sharing rules` and `Manual sharing`.

![](/integration-blog/assets/2019-02-26-data-security-of-salesforce/data_security_records.jpeg)

`org-wide` specify the default level of access each records, if you defined this setting with *public*, All users can view on the record, but if you set with *private*, users except for the record owner whether can see it was controlled by `Role hierarchies` and `Sharing rules`.

`Role hierarchies` ensure managers have access to the same records as their subordinates. Each role in the hierarchy represents a level of data access that a user or group of users needs.

`Sharing rules` and `Manual sharing` give some user access to records they don’t own or can’t normally see.

### Example

If company X has 5 sales teams, each team has 1 manager, and also 1 super manager for all managers.

Role hierarchy should be: 

```
 Super manager 
       Manager1 <- Team1
       Manager2 <- Team2
       Manager3 <- Team3
       Manager4 <- Team4
       Manager5 <- Team5
```

Profile should be only 3:

  - **Sales Team** (Assigned to all Teams)
  - **Manager** (Assigned to all Managers)
  - **Super Manager** (Assigned to super manager) 

  
### Conclusion

Salesforce platform provides a flexible, layered sharing model, it’s easy to assign different data sets to different sets of users but we should think more before we design and implement our data model.
