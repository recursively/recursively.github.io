---
title: Build A Contact-Retrieve Framework of QQ With MongoDB+Elasticsearch
categories: Python
tags:
  - Python
  - Mongodb
  - Elasticsearch
description: >-
  In order to analyze the communication contents and collect the files shared by
  group members with an automated method, I built a simple framework consist of
  MongoDB and Elasticsearch, which can store the communication contents in the
  MongoDB database and retrieve them later with Elasticsearch.

date: 2018-07-03 15:36:24
---
## Grab the Communication Contents From QQ Contacts

As the subtitle above, grabbing the QQ communication contents is the first thing to do. If you want to complete the scripts by yourself, Tencent has provided some APIs to use. You can also use a smart robot named QQbot to avoid massive and complex work: https://github.com/pandolia/qqbot.git. But there are some bugs in this project, you'd better fix them before deploying your code to the production environment.

Here gives an example:

```python
from qqbot import QQBotSlot as qqbotslot, RunBot

@qqbotslot
def onQQMessage(bot, contact, member, content):
    wrapped_information = []
    now = datetime.datetime.now()
    try:
        if contact.ctype == 'group' or contact.ctype == 'discuss':
            # if contact.ctype == 'group':
            combine = (member.name + '_' + str(groupindex[contact.name]))
            timeStamp = time.mktime(now.timetuple())
            # times = datetime.datetime.fromtimestamp(un_time)
            try:
                mongodbexec(name=combine, contact=content, groupnumber=groupindex[contact.name], groupname=contact.name, timeStamp=timeStamp, card=member.name, nick=member.nick)
                contactfilter(content, groupname=contact.name, groupnumber=groupindex[contact.name], card=member.name, nick=member.nick)
            except Exception as e:
                print(e)
            checked_url = []
            grabgroup = []
            pattern = re.compile(
                r'((http|ftp|https)://)(([a-zA-Z0-9\._-]+\.[a-zA-Z]{2,6})|([0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}))(:[0-9]{1,4})*(/[a-zA-Z0-9\&%_\./-~-]*)?([ ]*)(密码：[a-zA-Z0-9]{1,4})?')
            for m in pattern.finditer(content):
                if m.group().strip() not in checked_url and groupindex[contact.name] in grabgroup:
                    host_get = socket.gethostbyname(urlparse.urlparse(m.group()).netloc)
                    if re.search('pan.baidu', m.group()):
                        wrapped_information = '[' + str(member) + ']' + str(
                            m.group()) + '\r\n\r\n\r\n'
                    # wrapped_informations.append(wrapped_information)
                    # elif not re.search('192.168', host_get) and not re.search('10.', host_get) and not re.search('172.', host_get):
                    elif not host_get.startswith('192.168') and not host_get.startswith(
                            '10.') and not host_get.startswith('172.'):
                        url_r = requests.get(str(m.group()), timeout=30)
                        soup = BeautifulSoup(url_r.text, 'html.parser')
                        pattern = re.compile('<title>(.*?)</title>', re.DOTALL)
                        find_title = pattern.search(str(soup.find_all(["title"]))).group(1)
                        wrapped_information = '[' + str(member) + ']' + find_title.strip() + '\r\n' + str(
                            m.group()) + '\r\n\r\n\r\n'
                    # wrapped_informations.append(wrapped_information)
                    checked_url.append(m.group().strip())
                    with open(
                            r"/Users/Eric_ZYH/Documents/CODES/PYTHON/python3/qqbot_email/wrapped_informations_" + now.strftime(
                                    '%x').replace('/', '_') + ".txt", 'a')as f:
                        f.write(str(wrapped_information))
    except Exception as e:
        print(e)
```

This example aims at getting the URLs appeared in the communication contents and grab some information from these websites for the next work. And remember paying attention to the security problems such as injection, SSRF, etc.

If you have read the source code of the QQbot project, you can find a function called onStartupComplete() which starts after the smart QQbot startups. For better understanding, I put the startup flow chart of QQbot here:

![](http://i38.photobucket.com/albums/e117/bucketuser111/Blog/main_zpsbrikodii.png)

 To use the onStartupComplete() function, you can add the decorator @qqbotslot:

```python
@qqbotslot
def onStartupComplete(bot):
    grab = GrabQQList()
    grab.start()
    memberindex.update(grabinfo()[0])
    groupindex.update(grabinfo()[1])
```

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

Then you should start mongod with argument '--auth':

```shell
mongod --port 27017 --dbpath /data/db1 --auth
```

If you want to login into mongo console, you must provide your username and password like this:

```shell
mongo admin -u root -p password
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

