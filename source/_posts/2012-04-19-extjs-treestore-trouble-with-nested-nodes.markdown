---
comments: true
date: 2012-04-19 10:23:07
layout: post
slug: extjs-treestore-trouble-with-nested-nodes
title: ExtJS TreeStore trouble with nested nodes
wordpress_id: 455
categories:
- Programming
- Web Development
tags:
- javascript extjs workaround
---

At work, we're building an app to edit objects in a database--a classic CRUD application. For now, we're trying out ExtJS as the client-side UI framework. One of the use cases is selecting and editing nested objects, represented in our relational database with foreign keys. Let's call the root object a Task, which consists of nested Goals, which have Steps. Each of those is defined by a model on the backend that is more or less mimicked by an Ext.data.Model on the client-side, and each model has a proxy to a RESTful endpoint on the backend for create/retrieve/update/delete operations. We want to use an Ext.tree.TreePanel for the UI, so we hold the data in an Ext.data.TreeStore. So far so good. 

We coded up our prototype, but when a user selects a Task, Ext JS throws this error: `Uncaught TypeError: Cannot read property 'internalId' of undefined`. Hmm. Everything seems to be working. Our models are loading the correct data. No obvious bugs. A lot of inspecting and googling and reading documentation later, I discover [this thread](http://www.sencha.com/forum/archive/index.php/t-160068.html?s=03fb3a67ebf1e1ef856bc5f277ad12e8). The key quote: 



> It doesn't matter if the [model] ids are unique within the JSON [or any data]. It must be unique within the tree. 

If you add the first json to the tree with for example the id 4_1 and you add the second json with again a node 4_1 then you have two nodes with the same id. 



In other words, TreeStore doesn't distinguish the types of roots and their children (or children's children, etc). To TreeStore, they are **all** nodes, and ids must be unique across all nodes. If you have an instance of a Task model with id=1 and it has a foreign key to a Goal that also has id=1, TreeStore has a problem with that. Apparently it doesn't introspect the objects enough to see that, say, one is a Task and its children are Goals, despite the Task model having a `hasMany` field that defines its relation to the Goal model. That seems counterintuitive to me, maybe even misleading. Perhaps that's why we're not the only ones who've [had](http://www.sencha.com/forum/showthread.php?129524-CLOSED-Selection-of-Association-in-DataView) [this](http://www.sencha.com/forum/showthread.php?135285-TreeStore-Model-and-quot-id-quot-field) [problem](http://www.sencha.com/forum/showthread.php?196396-How-to-add-children-tree-nodes-dynamically). 

My quick fix was to write a `stringify_id()` function to wrap ids passed to the TreeStore with a prefix unique to each type, so the id of Task id=1 becomes "task-1". `destringify_id()` unwraps the ids that come back through the proxy. 

TreeStore's [docs](http://docs.sencha.com/ext-js/4-0/#!/api/Ext.data.TreeStore) do not mention this restriction, as far as I can tell. Maybe if you purchase Ext JS, you get better docs, I'm not sure. We may be doing just that, so I could have a chance to find out. One of the complaints you sometimes hear about open source is that the docs aren't that great, so I'm curious to see how a for-profit company's docs stack up against the documentation culture of the communities I'm most familiar with (Python and Django), which tend to be pretty solid.
