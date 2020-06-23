### Issue
An invalid HTTP header results in a tomcat's html error response.

```shell script
curl  http://localhost:8080/person
{"name":"John","age":50}


curl -H '(request): test'  http://localhost:8080/person
<!doctype html><html lang="en"><head><title>HTTP Status 400 – Bad Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 – Bad Request</h1></body></html>
```

On the server, we get this stack trace
```
2020-06-23 19:16:01.478  INFO 49412 --- [nio-8080-exec-1] o.apache.coyote.http11.Http11Processor   : Error parsing HTTP request header
 Note: further occurrences of HTTP request parsing errors will be logged at DEBUG level.

java.lang.IllegalArgumentException: The HTTP header line [(request): test] does not conform to RFC 7230 and has been ignored.
	at org.apache.coyote.http11.Http11InputBuffer.skipLine(Http11InputBuffer.java:1020) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.coyote.http11.Http11InputBuffer.parseHeader(Http11InputBuffer.java:872) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.coyote.http11.Http11InputBuffer.parseHeaders(Http11InputBuffer.java:594) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:283) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1590) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128) ~[na:na]
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628) ~[na:na]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) ~[tomcat-embed-core-9.0.36.jar:9.0.36]
	at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]
```

Since our API "promises" a JSON response, the clients cannot handle html in the response stream.

This used to work fine with `spring-boot-starter-parent:2.0.9.RELEASE` (where it pulled in `org.apache.tomcat.embed:tomcat-embed-core:jar:8.5.39` which did not have this issue)

It seems to have been broken in later versions of spring boot like `spring-boot-starter-parent:2.3.1.RELEASE` (which pulls in `org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.36` which has this issue)