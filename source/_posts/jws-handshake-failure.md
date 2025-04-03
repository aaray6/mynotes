---
title: Java jws handshake_failure
date: 2020-06-30 10:45:00
tags:
---

Env: Win7, jre 1.8.0_251-b08
Browser: Chrome/Firefox/IE11

When I run jws application, I got following exception.

```java
#### Java Web Start Error:
#### 无法加载资源: https://<host>/<path>/lib/jws-tools.jar

javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
	at sun.security.ssl.Alerts.getSSLException(Unknown Source)
	at sun.security.ssl.Alerts.getSSLException(Unknown Source)
	at sun.security.ssl.SSLSocketImpl.recvAlert(Unknown Source)
	at sun.security.ssl.SSLSocketImpl.readRecord(Unknown Source)
	at sun.security.ssl.SSLSocketImpl.performInitialHandshake(Unknown Source)
	at sun.security.ssl.SSLSocketImpl.startHandshake(Unknown Source)
	at sun.security.ssl.SSLSocketImpl.startHandshake(Unknown Source)
	at sun.net.www.protocol.https.HttpsClient.afterConnect(Unknown Source)
...

com.sun.deploy.net.FailedDownloadException: 无法加载资源: https://<host>/<path>/lib/jws-tools.jar
```

The same applications works fine on Win10 and Ubuntu.

To fix this issue on Win7, go to "control panel"-> "java"

In "高级" panel, uncheck "使用与 SSL 2.0 兼容的 ClientHello 格式" option.
This option is unchecked in Win10 by default.

![screenshot](/myimages/jws_handshake_failure_java_setting.png )
