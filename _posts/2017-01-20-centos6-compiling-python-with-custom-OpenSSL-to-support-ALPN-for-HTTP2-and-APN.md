---
title: Allow to send HTTP/2 requests to APNS on CentOS6 and Python3.5.
description: "Describes how to compile python with custom openssl package to allow issuing HTTP/2 requests to Apple Push Notifications Service"
layout: post
author: andrii
tags: [akozlovskyi, python, eh6, openssl, apn, http2, devops]
categories: [DevOps]
modified: 2017-01-20
image:
    feature: posts/cent_os6_openssl/title.jpg
    
---

Trying to send Apple push notifications (`APN`) on `CentOS6` using python hyper library got the ```ConnectionResetError```
And it looked like `hyper` is trying to use `HTTP1.1` instead of `HTTP/2.`

It turns out `hyper` requires an `ALPN` (Application-Layer Protocol Negotiation) to negotiate `HTTP/2`.
The latest openssl version available for eh6 is `OpenSSL 1.0.1e`, but `ALPN` was introduced > 1.0.2.
In order not to brake dependencies, we need to compile and install new openssl library into our custom location.
Also we need to compile Python specifying custom openssl library.

Let'd do it... 

<!-- more -->

## Install OpenSSL

Download and unpack sources:

```
cd /usr/local/src/
wget https://openssl.org/source/openssl-1.0.2j.tar.gz && tar -xzf openssl-1.0.2j.tar.gz
```

Configure and install

```
cd openssl-1.0.2j
./config shared --prefix=/usr/local --openssldir=/usr/local/openssl
make & make install
```

## Install Python:

Download and unpack sources:

```
cd /usr/local/src/
wget https://www.python.org/ftp/python/3.5.3/Python-3.5.3.tgz && tar -xzf Python-3.5.3.tgz
```

Configure and install

```
cd Python-3.5.3
export LD_LIBRARY_PATH="/usr/local/lib/:/usr/local/lib64/" 
export CPPFLAGS="-I/usr/local/include -I/usr/local/include/openssl"
export LDFLAGS="-L/usr/local/lib/ -L/usr/local/lib64/"
./configure --prefix=/usr/local/
make & make install
```

**Important**: ensure all paths are correct
At this point a good sign is not to have `pip` related errors.

Now let's check ssl version.

```
/usr/local/bin/python3
Python 3.5.3 (default, Jan 20 2017, 15:47:07) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-17)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import ssl
>>> ssl.OPENSSL_VERSION
'OpenSSL 1.0.2j  26 Sep 2016'
```

If you see something like:
```
ImportError: No module named _ssl
```

You can try:
1. Ensure Environment variables were set correctly
2. Ensure app path are correct.
3. According to different sources: You may or may not need to install `openssl-devel` package before compiling Python. (I didn't)
4. Recompile an reinstall python

Now you can install `pyopenssl`, `hyper` using `pip3`, or create a virtual env and install there

```
/usr/local/bin/python3 /usr/local/bin/pip3 install pyopenssl hyper
```

Now after some coding you actually can try to send `APN`.

**Notes**:
Prior to `pyopenssl` you may need `ibffi` `libffi-devel` packages.

**Additional**:
There is a way to compile NGINX to serve as HTTP/2 server under CentOS6, check this script: [script](https://gist.github.com/kennwhite/6b6250e635c45c92a118a7a5cdc052c6)

On stack overflow for Python 3.4: [How do I compile python 3.4 withcustom openssl](http://stackoverflow.com/questions/23548188/how-do-i-compile-python-3-4-with-custom-openssl)
