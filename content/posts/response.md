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