---
layout: post
title: Mongodb document size limit
date: 2013-02-26 21:50
comments: true
categories: mongodb youtube bson
---

    As a MongoDB beginner
    When designing a database for large quantities of data
    Then I should be aware of the document size 16MB limit in BSON format

The Document limit size is 16MB unless you’re using [GridFS](http://docs.mongodb.org/manual/applications/gridfs/) (but it seems like if you can’t get GridFS from 3rd parties like MongoHQ or MongoLab?). Remember that MongoDB by default puts the working set of your database in RAM.

So if you expect to have millions of some things then you better put those things in a separate collection, even if semantically it may make more sense to embed them in another document. If you’re using [mongoid](http://mongoid.org/en/mongoid/) that simply means having a standalone model referencing another model.

Useful discussions on Stack Overflow:

* [Understanding MongoDB BSON document size limit](http://stackoverflow.com/questions/4667597/understanding-mongodb-bson-document-size-limit)
* [How to determine a document’s size with Mongoid](http://stackoverflow.com/questions/8773557/determine-size-of-a-document-using-mongoid)
* [What’s a MongoDB working set](http://stackoverflow.com/questions/6453584/what-does-it-mean-to-fit-working-set-into-ram-for-mongodb)

## reference

* Check out [This blog](http://architects.dzone.com/articles/get-know-gridfs-mongodb) to get to know GridFs
* [Intro to bson on youtube](https://www.youtube.com/watch?v=K3J6WvDW-Hc)
* [Using gridfs](https://github.com/nickleefly/mongodb101/blob/master/week3/using_gridfs.py)
