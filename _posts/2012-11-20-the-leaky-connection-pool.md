---
layout: post
title: "The Leaky Connection Pool"
description: ""
category: posts
tags: [ s3, aws, java, connection ]
---
{% include JB/setup %}

Down the rabbit hole
-------------------- 
This is a simple, note worthy issue I noticed at work a few days ago - There also wasn't much information about this particular problem out on the internet (I did not look too hard, so maybe I am wrong :) ). 

Anways, So a few days ago, logging into one of our ec2 boxes , I started noticing a whole host of errors like so in our logs. (Where I work, ec2 boxes are 'special', in the sense that they do not come out of the box with automatic metric and logscans and other goodness.)


       Caused by: com.amazonaws.AmazonClientException: Unable to execute HTTP request: Timeout waiting for connection
       at com.amazonaws.http.AmazonHttpClient.executeHelper(AmazonHttpClient.java:299)
       at com.amazonaws.http.AmazonHttpClient.execute(AmazonHttpClient.java:170)
       at com.amazonaws.services.s3.AmazonS3Client.invoke(AmazonS3Client.java:2648)
       at com.amazonaws.services.s3.AmazonS3Client.getObject(AmazonS3Client.java:831)
       at com.amazonaws.services.s3.AmazonS3Client.getObject(AmazonS3Client.java:737)
       at biz.neustar.migration.dao.s3.S3TestScriptDAO.getScriptBody(S3TestScriptDAO.java:95)


At first glance it seems like a problem with S3 - except I dont ever recall there being a problem with S3 ( save this one case , but thats a different blog post). 
So I quickly ran netstat, to see whats going on: 

     $ netstat -antp
   	tcp        0      0 :::35904                    :::*                        LISTEN      23239/java
   	tcp        0      0 :::7780                     :::*                        LISTEN      23898/java          
   	tcp        0      0 :::50313                    :::*                        LISTEN      8323/java    
   	tcp        0      0 :::8080                     :::*                        LISTEN      5287/java         
   	tcp        0      0 :::22                       :::*                        LISTEN      2761/sshd
   	tcp        0      0 :::6743                     :::*                        LISTEN      23239/java        
   	tcp        0      0 :::8443                     :::*                        LISTEN      5287/java         
   	tcp        0      0 :::6780                     :::*                        LISTEN      23239/java        
   	tcp        1      0 ::ffff:127.0.0.1:51552      ::ffff:127.0.0.1:2009       CLOSE_WAIT  23239/java        
   	tcp        0      0 ::ffff:10.61.96.216:43522   ::ffff:176.32.100.128:443   ESTABLISHED 21456/java        
   	tcp        0      0 ::ffff:10.61.96.216:49241   ::ffff:176.32.98.215:443    ESTABLISHED 21456/java     
   	tcp        0      0 ::ffff:10.61.96.216:49242   ::ffff:176.32.98.215:443    ESTABLISHED 21456/java  
   	tcp        1      0 ::ffff:127.0.0.1:43956      ::ffff:127.0.0.1:2009       CLOSE_WAIT  23898/java      
   	tcp        1      0 ::ffff:127.0.0.1:42521      ::ffff:127.0.0.1:2009       CLOSE_WAIT  23898/java      
   	tcp     2486      0 ::ffff:10.61.96.216:60812   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:60205   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:58438   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:58761   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:59360   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:57951   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp        0      0 ::ffff:127.0.0.1:40711      ::ffff:127.0.0.1:6969       ESTABLISHED 23239/java      
   	tcp     2486      0 ::ffff:10.61.96.216:56149   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:54684   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java      
   	tcp     2486      0 ::ffff:10.61.96.216:54685   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java     
   	tcp     2486      0 ::ffff:10.61.96.216:54674   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java     
   	tcp     2486      0 ::ffff:10.61.96.216:54675   ::ffff:169.254.169.254:80   CLOSE_WAIT  21456/java

So essentially I was seeing a chock full of CLOSE_WAIT connections to S3. It was apparent that this was causing problems, but I wasn't sure what the **CLOSE_WAIT**
state is, and _why_ exactly that is a problem, and further how to root cause this in our code base. Obviously this was a problem with the way the s3 client was being used - but it wasn't clear what anti pattern was causing this problem. Anyway's here is the tcp connection state diagram. 

![tcp_state](http://publib.boulder.ibm.com/infocenter/zos/v1r11/topic/com.ibm.zos.r11.halu101/dwgl0004.gif "Tcp State Diagram")

The **CLOSE_WAIT** state is a passive close state, and a client's socket will continue to be in this state until it actively closes the connection. Note , that this is completely on the client side and unlike the other states in the TCP state diagram, this state doesn't have a timeout - which means it is going to be open indefinitely. 
So what was causing this problem ? Turns out the S3 [getObject()](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3Client.html#getObject(com.amazonaws.services.s3.model.GetObjectRequest) ) call keeps the http connection open until it is closed by the client. The anti-pattern here was a very simple: 

 ByteStreams.copy(object.getObjectContent(), Files.newOutputStreamSupplier(temporaryScriptFile));

The silent call to object.getObjectContent() wasn't closing the open connection. The best way to approach these problems is to actually create a decorator against the s3 client api and use that to access getObjectContent(). That is something I will cover in a different blog post though. 
