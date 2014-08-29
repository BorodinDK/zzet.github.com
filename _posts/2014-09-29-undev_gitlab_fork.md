---
layout: post
title: "About Undev Gitlab fork"
date: 2014-09-29 18:18
meta: true
comments: true
categories: git gitlab undev fork gitlab-vagrant elasticsearch-git elasticsearch
tags: git gitlab undev fork gitlab-vagrant elasticsearch-git elasticsearch
---

## About article

I get a lot of mail with questions about our ([Undev](http://undev.ru)) [Gitlab fork](https://github.com/Undev/gitlabhq). And... I so lazy... In this article I try to describe our fork and them features.

If you have some question or can't understand something - please write comments, I'll update article.

## Start. Gitorious.

At September 2012 I joined to Infrastructure projects in Undev. It was very interesting... but... old Gitorious, Rails 2, ruby 1.8.7ee... Shit. Ok. After some time of terrible work I start discussion with our manager about another system. We have a lot of problems with Gitorious and support of them.

## Gitlab

Challenge accepted and we start research. At November 2012 we have big (very big) compare table. After some testing Gitlab win and... no... Gitlab do not have some important for us features... Okay. Let's go, Guys! We start new features.

* At first - postgresql support. Yes, I do not like MySQL :)
* Migration script from Gitorious to Gitlab
* Mail notifications
* Teams support for permissions management
* Snippets
* etc.

At May 2013 we have good installation. But with some changes in architecture, about which I'll write later. We run Gitlab in production at May 2013.

## Most valuable Changes

### Teams support

In Gitlab 3 we do not have teams. Management of user access in a lot of projects was terrible work and we start **Teams feature**. First version  of this features was not beautiful, some times - hardcoded, and in Gitlab 6 this feature was replaced with Groups by Dmitry Zaporozhets. I agree with them - for little company and teams Team feature is overhead. But we can not abandon Team feature and support them in our form today.
We make 3 level of user access to projects:

* Add user directly to project
User -> Project

* Add user to group, in which located project
User -> Group -> Project

* Add user in Team, which can be assigned to project directly, or to group of projects
User -> Team -> Project
User -> Team -> Group -> Project

This schema very comfortable for our Managers, Project Masters/Owners and we support them ;)

### New event model

Then we (with [Andrey Kulakov](https://github.com/Andrew8xx8)) start release **mail notification** feature, we select event-based mail generation. User has subscription on base Entities: Project, Group, Team, User. After create Event, related to base entity on User action we create mail notification based on this Event and async send they. More detailed I'll describe mail notification later. Now about events.

At start we have one huge problem. All events in Gitlab related to project. We has not events for Group or Team or another Entity. Only Project.

``` ruby
Event(id:          integer,
      target_type: string,  # Polymorth association with
      target_id:   integer, # Entity, in which was event
	  data:        text,    # Serialized event data
	  project_id:  integer, # Only project ;,(
	  created_at:  datetime,
	  action:      integer, # Action number.
	  author_id:   integer)
```

And all events described with 9 action constant:

``` ruby
  CREATED   = 1
  UPDATED   = 2
  CLOSED    = 3
  REOPENED  = 4
  PUSHED    = 5
  COMMENTED = 6
  MERGED    = 7
  JOINED    = 8 # User joined project
  LEFT      = 9 # User left project
```
So, it was problem and we start new Events.

We has next requirements:

* Any entity in system may be Event target (entity, related to which was created event)
* Any entity can be Event source (entity, which triggered event)
* Ability to describe action with human like name
* Store event data to future usage of them

As result we have some solution:
Any entity can be target
![In new events any entity may be target](http://puu.sh/bcPBR/ececc5fafb.png)

Any Entity can be source
![Simple example of relation between different entities with project](http://puu.sh/bcPGQ/83a806b906.png)

And rich action description:

``` ruby
  GENERAL = [
    :created,
    :updated,
    :commented,
    :deleted,
    :added,
    :removed,
    :joined,
    :left,
    :transfer,
  ]

  COMMENTS = [
    :commented_merge_request,
    :commented_issue, # not used in our fork
                      # because we use another issue tracker
    :commented_commit,
    :commented,       # not used after remove Project Wall
  ]

  MERGE_REQUESTS = [
    :opened,
    :closed,
    :reopened,
    :merged,
    :assigned,
    :reassigned,
    :resigned,
  ]

  MASS = [
    :imported,
    :members_added,
    :members_updated,
    :members_removed,
    :teams_added,
    :teams_removed,
    :groups_added,
    :projects_added
  ]

  GIT = [
    :pushed,
    :created_branch,
    :deleted_branch,
    :created_tag,
    :deleted_tag,
    :protected,
    :unprotected,
    :blocked,
    :activate,
  ]

  # NOTE actions which can be parent
  BASE = [
      :create,
      :update,
      :delete,
      :open,
      :close,
      :reopen,
      :merge,
      :block,
      :activate
  ]
```

Event can have parent event. For example case:

* User pushed code to server
	* After was created note
	* After push was closed MergeRequest
	* After push was closed Issue

We create events for any actions in system. But save parent-child relation.

``` ruby
Push created |- Event(action: pushed)
             |- Event(action: commented_commit)
             |- Event(action: closed) # for MergeRequest
             |  |- Event(action: created) # Note was created
 		     |     |- Event(action: commented_merge_request)
  	     |
             |- Event(action: closed) # for Issue
                |- Event(action: created) # Note was created
                   |- Event(action: commented_issue)
```

Based on this events we can send email in different subscriptions without duplications and research source of some troubles. I think it awesome! :)

After rewrite events we have:

* Ability detect who and what doing in Gitlab on fuckups
* Full information dashboard for Main and Project, Group, Team, User pages.
* Flexible mail subscriptions with notifications
* Ability to send mail digests

### Awesome mail notifications
After rewriting events we create own mail notifications.

Our workflow:

* User subscribe to base entity
	* Project
	* Group
	* Team
	* User
![](http://puu.sh/bcRvH/ae02e0d349.png)
* User can edit detailed settings for any subscriptions
![](http://puu.sh/bcRpA/90114af5ae.png)
* Another user do something in Gitlab
* After user actions - Gitlab trigger event create, and we have tree of events
* On created events base we create notifications for subscribers
* And send mails for subscribers

At this moment we have more 100 different mails for different cases.
And can be more :)

TODO: Our plans create notification page with option to show notifications or send them on email.
TODO: Add ability to subscribe on MergeRequest and Issue

### New Services logic
Gitlab has integration with different services and it's OK, but it's ok for them, as SaaS.

We rewrote services, because:

* For one organization it is ugly to write some configs many time while service added to different projects. We want to write config one time and enable service for different projects many time.
* We want to have ability for add deploy key for service.
* We want to have ability for integration of Gitlab with different other systems
* etc

Now:

* In Admin panel we add Service pattern with default values
![](http://puu.sh/bcSXJ/40ec7e593e.png)
![](http://puu.sh/bcSTK/a62e2073f8.png)
* We can add sha-key for service (like deploy key)
![](http://puu.sh/bcT25/0da1c3a2e3.png)
* In Any project we can enable this service pattern (we can edit config, if it need)
![](http://puu.sh/bcT7s/1132856552.png)

### Elasticsearch as search engine
### Jenkins integration
### More performance with websockets
### Favorited projects
### Access to files via token
### Git protocol support

## Architecture changes
### Deploy via capistrano
## Related links

[Fork address (Undev/gitlabhq)](https://github.com/Undev/gitlabhq)

[Gitlab-shell fork for git protocol ability](https://github.com/zzet/gitlab-shell)

[Our vagrant vw for development](https://github.com/zzet/gitlab-wvm)

[Elasticsearch-git gem for Integration with ElasticSearch](https://github.com/zzet/elasticsearch-git)
