---
title: BurpSuite Jython Extension for Passive Scanner

date: 2018-05-8 19:01:10 

categories: Python

tags: Python 

description: A simple BurpSuite Jython extension to capture https traffic, which is used in the passive scanning.
---
## BurpSuite Jython Extension

In order to glean the vulnerability detection information and avoid scanning the massive useless URLs of the websites, I prefer using passive scanner during my daily work.

I'd like to use BurpSuite to analyze URLs and store them in the database for the next passive scanning process via socket tunnel. But here comes the problem, BurpSuite cannot transfer https flow to another place, so I wrote a simple Jython script as the disposal.

Tutorial about writing a Jython extension can be found everywhere. Here I put the main code 

```python
def processHttpMessage(self, toolFlag, messageIsRequest, messageInfo):
    
        # only process requests
        if not messageIsRequest:
        
            # create a new log entry with the message details
            self._lock.acquire()
            row = self._log.size()
            req = self._helpers.analyzeRequest(messageInfo)
            LE = LogEntry(toolFlag, self._callbacks.saveBuffersToTempFiles(messageInfo), req.getUrl())
            self._log.add(LE)
            try:
                params = req.getParameters()
                pas = []
                pnames = []
                for pa in params:
                    pnames.append(pa.getName())
                    pas.append(pa.getName()+"="+pa.getValue())
                #pas = pas.join('&')
                
                values = {"docs":[{"TIME":time.time(),"URL":LE._url.toString(),"PNames":'&'.join(pnames),"Method":req.getMethod(),"HOST":LE._url.getHost(),"PATH":LE._url.getPath(),"PARAM":'&'.join(pas),"REQ":LE._requestResponse.getRequest().tostring(),"RESP":LE._requestResponse.getResponse().tostring(),"USER":"test"}]}
                
                #data = 'docs=[{"URL":"'+LE._url.toString()+'","REQ":"'+urllib.quote(LE._requestResponse.getRequest().tostring())+'","RESP":"'+urllib.quote(LE._requestResponse.getResponse().tostring())+'"}]'
                
                # data = "docs="+urllib.quote(json.dumps(values['docs'], sort_keys=True))
                send_data = {}
                req_url = LE._url.toString()
                req_headers, req_body = split_req(LE._requestResponse.getRequest().tostring())
                o = urlparse(req_url)
                req_host = o.netloc
                req_headers['Host'] = req_host
                req_method = req.getMethod()
                data = extract_request(req_url, req_headers, req_method, req_body)
                send_data['req_url'] = req_url
                send_data['req_headers'], send_data['req_body'] = req_headers, req_body
                send_data['req_host'] = req_host
                send_data['req_method'] = req_method
                send_data['data'] = data
                print(send_data)

                target_url = "http://192.168.10.57:8888"
                res = urllib2.Request(target_url, str(send_data))
                threading.Thread(target=urllib2.urlopen, args=(res,)).start()

            except:
                pass
            self.fireTableRowsInserted(row, row)
            self._lock.release()
        return
        
def extract_request(url, headers, method, body):
    try:
        a = re.search('://.*?/(.*)', url)
    except:
        pass
    url = ('/' + a.group(1)) if a is not None else '/'
    requests = "%s %s\r\n" % (method, url)
    for key, value in headers.items():
        requests += "%s: %s\r\n" % (key, value)
    requests += "\r\n%s" % body
    return requests 
```

the function extract_request(url, headers, method, body) is used to transform requests and the processHttpMessage(self, toolFlag, messageIsRequest, messageInfo) function is used to send the BurpSuite URLs to another server which is started locally. Sure, there's no need to do such complicated work, but I can't find some modules(redis) I need in Jython, maybe you have better ideas I don't know...Start a server

I put my server code below:

```py
from http.server import BaseHTTPRequestHandler, HTTPServer
import logging
from lib.redisopt import content_deal


class S(BaseHTTPRequestHandler):
    def _set_response(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()

    def do_GET(self):
        self.wfile.write("GET request for {}".format(self.headers).encode('utf-8'))

    def do_POST(self):
        content_length = int(self.headers['Content-Length']) # <--- Gets the size of data
        post_data = self.rfile.read(content_length) # <--- Gets the data itself
        receive_data = eval(post_data.decode('utf-8'))
        req_url = receive_data['req_url']
        req_headers, req_body = receive_data['req_headers'], receive_data['req_body']
        req_host = receive_data['req_host']
        req_method = receive_data['req_method']
        data = receive_data['data']
        content_deal(req_headers, req_host, req_method, postdata=req_body, uri=req_url, packet=data)

        self.wfile.write("POST request for {}".format(self.headers).encode('utf-8'))


def run(server_class=HTTPServer, handler_class=S, port=8888):
    logging.basicConfig(level=logging.INFO)
    server_address = ('0.0.0.0', port)
    httpd = server_class(server_address, handler_class)
    logging.info('Starting httpd...\n')
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        pass
    httpd.server_close()
    logging.info('Stopping httpd...\n')


if __name__ == '__main__':
    from sys import argv

    if len(argv) == 2:
        run(port=int(argv[1]))
    else:
        run()
```

The function content_deal() is declared in the passive scanner system which written in python.

one last thing we should pay attention to is that if we decide to use BurpSuite to transfer the https data, we must import the BurpSuite certificate. Of course, in addition to using BurpSuite, we also have many other choices, such as sslsplit, mitmproxy…and we need to import the right certificates by the same.
