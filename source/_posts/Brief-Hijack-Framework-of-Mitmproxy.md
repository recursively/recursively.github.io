---
title: Brief Hijack Framework of Mitmproxy
date: 2019-11-16 17:14:00
categories: Network
tags: [MITM, mitmproxy]
keywords: [MITM, mitmproxy]
description: Mitmproxy is an efficient and lightweight proxy tool with convenient scripting mechanism which makes it suit most scenarios. We will disscuss how to perform a Man-in-the-middle attack with a little code in this post.
---
## How Will This Framework Work
* If you query information using www.baidu.com, it will response with code 404, and it will suggest you to use www.bing.com to search whatever you want.
* When you open www.bing.com and enter what you want to search, you will find that no matter what you have entered, the response result wiil always be yandex.
* Again, you click the link and open yandex to query information, it seems that everything is working properly this time.
* Wait a moment, you can find that the search button has been replaced with "Try Google".

## Scripting
Here gives all the code:
```python
import mitmproxy.http
from mitmproxy import ctx, http


class Tamper:
    def request(self, flow: mitmproxy.http.HTTPFlow):
        if flow.request.host == "www.baidu.com":
            flow.response = http.HTTPResponse.make(
                404, 
                b"Don't use baidu to query anything please, try bing.com. :)", 
                {"Content-Type": "text/html"}
            )

        if flow.request.host != "cn.bing.com" or not flow.request.path.startswith("/search?"):
            return

        if "q" not in flow.request.query.keys():
            ctx.log.warn("can not get search word from %s" % flow.request.pretty_url)
            return

        ctx.log.info("catch search word: %s" % flow.request.query.get("q"))
        flow.request.query.set_all("q", ["Yandex"])

    def response(self, flow: mitmproxy.http.HTTPFlow):
        if flow.request.host != "yandex.com":
            return

        text = flow.response.get_text()
        text = text.replace("Search", "Try Google :)")
        flow.response.set_text(text)


class Counter:
    def __init__(self):
        self.num = 0

    def request(self, flow: mitmproxy.http.HTTPFlow):
        self.num = self.num + 1
        ctx.log.info("We've seen %d flows" % self.num)


addons = [
    Tamper(),
    Counter()
]

```
Start up the proxy server to load the script, there are basically three ways to run the proxy:
* Only run the server on the backend.
```shell
mitmdump --listen-port 9090 -s tamper.py
```
* Run the server with a terminal-based GUI.
```shell
mitmproxy --listen-port 9090 -s tamper.py
```
* Run the server with a web-based GUI.
```shell
mitmweb --web-port 9091 --listen-port 9090 -s tamper.py
```

## Install The Certificate
You will get the wrong-certificate error if you directly access the HTTPS sites through proxy server. For Chrome, you can type "thisisunsafe" to bypass it or install and trust the certificate to access the proxy server permanently.
Access the proxy server with certificate:
* curl on the command line:
```shell
curl --proxy 127.0.0.1:9090 --cacert ~/.mitmproxy/mitmproxy-ca-cert.pem https://example.com/
```
* wget on the command line:
```shell
wget -e https_proxy=127.0.0.1:9090 --ca-certificate ~/.mitmproxy/mitmproxy-ca-cert.pem https://example.com/
```
Or execute the ~/.mitmproxy/mitmproxy-ca-cert.pem file to install the certificate.

## Verification
After all done, you can check the proxy result to verify it's working properly.

<img src="pic_1.png" width="60%" height="60%">

