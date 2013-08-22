---
layout: post
title: "Copying a DynamoDB table across AWS regions"
description: ""
category: 
tags: [Hive, dynamodb, EMR]
---
{% include JB/setup %}

The EMR Hive SerDe now supports region attributes on dynamodb tables. This is
pretty cool since it allows you to do really neat stuff. Now using simple hive
scripts you can :
* Query/Join/Aggregate data across tables in different regions  - without
having to copy over everything locally. 
* Filter and copy data from one table in one region to another. 
* You can also set the read/write throughput attributes on your tables to
ensure you cluster doesnt eat too much of your dynamo capacity. 

In effect this allows you to have pseudo-replication for your dynamodb tables
across regions - This is way better than just az independence and in case any one
AWS region goes down you are not totally lost as you have some sort of back up
in another region for applications to switch over to. Note I call it
pseudo-replication since deletes are not replicated in the destination
table. This isn't that big of a problem since ideally you should structure
your dynamo tables to have soft deletes and not hard deletes. If cost is that
big of a factor then you should try and ensure that your tables are
partitioned by time, and backup relatively old tables to S3. 

Here is a toy example that copys from one table in the eu-west-1 region
to the us-west-2 region and then copys to both us-west-2 and us-east-1 using a
single statement. 

     DROP TABLE IF EXISTS nddb_euwest1_test; 
     DROP TABLE IF EXISTS nddb_uswest2_test; 

     --Primary table in eu-west-1 region
     CREATE EXTERNAL TABLE nddb_euwest1_test( uuid_hash STRING, day STRING, name STRING,
     value STRING)
     	   STORED BY 'org.apache.hadoop.hive.dynamodb.DynamoDBStorageHandler' 
     	   TBLPROPERTIES ( 
      	   "dynamodb.table.name" = "testtable",
      	   "dynamodb.region"   = "eu-west-1", 
      	   "dynamodb.throughput.read.percent" = ".1000",
      	   "dynamodb.throughput.write.percent" = ".1000", 
      	   "dynamodb.column.mapping" =
      	   "uuid_hash:uuid_hash,day:day,name:name,value:value"); 
 
     --Secondary copy in us-west-2
     CREATE EXTERNAL TABLE nddb_uswest2_test( uuid_hash STRING, day STRING, name
     	    STRING, value STRING)
       	       STORED BY 'org.apache.hadoop.hive.dynamodb.DynamoDBStorageHandler' 
       	       TBLPROPERTIES ( 
       	       "dynamodb.table.name" = "kgpai.uswest.test",
       	       "dynamodb.region"   = "us-west-2", 
       	       "dynamodb.throughput.read.percent" = ".4000",
	       "dynamodb.throughput.write.percent" = ".4000",
       	       "dynamodb.column.mapping" =
       	       "uuid_hash:uuid_hash,day:day,name:name,value:value"); 

     --Make a schemaless copy 
     insert into table nddb_uswest2_test select * from nddb_euwest1_test;
 

     -- Multi inserts allow you to copy to multiple regions at once. 
     CREATE EXTERNAL TABLE nddb_useast1_test( uuid_hash STRING, day STRING, name
     	    STRING, value STRING)
       	       STORED BY 'org.apache.hadoop.hive.dynamodb.DynamoDBStorageHandler' 
       	       TBLPROPERTIES ( 
       	       "dynamodb.table.name" = "kgpai.useast.test",
       	       "dynamodb.region"   = "us-east-1", 
       	       "dynamodb.throughput.read.percent" = ".4000",
	       "dynamodb.throughput.write.percent" = ".4000",
       	       "dynamodb.column.mapping" =
       	       "uuid_hash:uuid_hash,day:day,name:name,value:value"); 
 
      -- Copy across multiple regions at once
      FROM nddb_euwest1_test 
      INSERT INTO nddb_useast1_test select * 
      INSERT INTO nddb_uswest2_test select * ; 


Although I have only shown schemaless backups above, you can also do schemaful
backups with the capability to filter out rows etc. The cluster configuration that has
served me quite well has had  an  m1.large as the master and 3 m1.larges again
as the core instances. In my adhoc testing I have easily been able to push it
to  10,000 tps and I reckon it very likely can do more. 




  
