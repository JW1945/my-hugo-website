+++
author = "Frenchie"
title = "Ambassador"
date = "2023-01-29"
description = "Ambassador HTB"
tags = [
    "Gaming",
    "Hacking",
]
showthedate = false
+++

## Ambassador HackTheBox
### Lesson Learnt

While doing manual LFI at grafana endpoint, I tried to exfill sqlite3 db from /var/lib/grafana/grafana.db

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

Use requests.content instead of requests.text