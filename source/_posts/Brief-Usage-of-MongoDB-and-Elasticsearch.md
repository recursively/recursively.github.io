---
title: Brief Usage of MongoDB and Elasticsearch
categories: Python
tags:
  - Python
  - Mongodb
  - Elasticsearch
description: >-
  The purpose of this post is to write down the common usage of MongoDB in my daily work. In addition, some manipulations for Elasticsearch are useful under several situations.

date: 2018-07-03 15:36:24
---

## Store Data in MongoDB

To store data in MongoDB database, I use the python module pymongo as shown below:

```python
import pymongo

client = pymongo.MongoClient("mongodb://%s:%s@127.0.0.1" % ('user', 'password'), port=22222)

def mongodbexec(name, contact, groupnumber, groupname, timeStamp, card, nick):
    member = {
        'groupcard': card,
        'groupnumber': groupnumber,
        'groupname': groupname,
        'contact': contact,
        'nickname': nick,
        'qqage': memberindex[name][1],
        'qq': memberindex[name][2],
        'timestamp': timeStamp,
    }
    QQres.insert_one(member)
```

And for security, adding an auth process is necessary. In mongo console:

```shell
use admin //switch to admin database
db.createUser({user:'root', pwd: 'password', roles: ['root']}) //create an administrator
```

If you want to change your user's password later, you can use the changeUserPassword command:

```shell
db.changeUserPassword('root','password1')
```

Then you should start mongod with argument '--auth':

```shell
mongod --port 27017 --dbpath /data/db1 --auth
```

If you want to login into mongo console, you must provide your username and password like this:

```shell
mongo admin -u root -p password
```

Export data to csv(https://docs.mongodb.com/manual/reference/program/mongoexport/):
```shell
mongoexport --username xxx --password xxx --authenticationDatabase admin --db xxx --collection xxx --type csv --fields xxx,xxx --out .../output.csv
```

## Export MongoDB Database Into Elasticsearch

There are some useful tools to finish this job such as mongo-connector, transporter, etc. But I met some problems when I use mongo-connector and I speculate it's the incompatibility of versions between mongo-connector and Elasticsearch. So I chose transporter which is an open source and high-efficiency tool built with go.

It's easy to build and configure. More details: https://github.com/compose/transporter.

When building finished, the first thing is to initialize the transporter:

```shell
transporter init mongodb elasticsearch
```

This step will generate a file pipeline.js under the directory of cmd/transporter, open and modify the file:

```js
var source = mongodb({
  "uri": "mongodb://127.0.0.1:22222/database"
  // "timeout": "30s",
  // "tail": false,
  // "ssl": false,
  // "cacerts": ["/path/to/cert.pem"],
  // "wc": 1,
  // "fsync": false,
  // "bulk": false,
  // "collection_filters": "{}",
  // "read_preference": "Primary"
})

var sink = elasticsearch({
  "uri": "http://elastic:password@localhost:9200/database"
  // "timeout": "10s", // defaults to 30s
  // "aws_access_key": "ABCDEF", // used for signing requests to AWS Elasticsearch service
  // "aws_access_secret": "ABCDEF" // used for signing requests to AWS Elasticsearch service
  // "parent_id": "elastic_parent" // defaults to "elastic_parent" parent identifier for Elasticsearch
})

t.Source("source", source, "/.*/").Save("sink", sink, "/.*/")
```

Finally run the transporter to transfer the database from MongoDB to Elasticsearch.

```shell
transporter run
```



## Retrieve Keywords With Elasticsearch

Reading the docs to learn about the usage of Elasticsearch: https://www.elastic.co/guide/index.html.

