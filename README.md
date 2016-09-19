
##Enable  WebHcat on Cloudera manager

####Add webchat instance from cloudera manager
+ Go to Cloudera manager and click Hive-> Instances -> Add Role Instances -> Select a host for WebHCat Server -> Finished
+ Start WebHCat server, now the WebHCat server is running and metadata related operation should work fine.
+ We need additional configuration for Hive, Pig, Mapreduce Jar and Mapreduce Streaming.
    - Go to Hive -> Configuration -> WebHCat Server Default Group ->  Advanced -> WebHCat Server Advanced Configuration Snippet(Safety Value) for webhcat-site.xml.
    -  the configuration for Hive, Pig, Mapreduce Jar and Mapreduce Streaming.
    - Hive, Pig we need specify Archive HDFS path and hive executable command path in archive file. see example:
    
```xml
    <property>
        <name>templeton.libjars</name>
        <value>/opt/cloudera/parcels/CDH-5.7.1-1.cdh5.7.1.p0.11/lib/zookeeper/zookeeper-3.4.5-cdh5.7.1.jar</value>
        <description>zookeeper jars to add to the classpath.</description>
    </property>
    <property>
        <name>templeton.hive.archive</name>
        <value>hdfs:///apps/webhcat/hive.tar.gz</value>
        <description>The path to the Hive archive file in hdfs.</description>
    </property>
    <property>
        <name>templeton.hive.path</name>
        <value>hive.tar.gz/bin/hive</value>
        <description>The path to the Hive executable.</description>
    </property>
    <property>
        <name>templeton.pig.archive</name>
        <value>hdfs:///apps/webhcat/pig.tar.gz</value>
        <description>The path to the pig archive file in hdfs </description>
    </property>

    <property>
        <name>templeton.pig.path</name>
        <value>pig.tar.gz/bin/pig</value>
        <description>The path to the pig executable.</description>
    </property>
    <property>
        <name>templeton.streaming.jar</name>
        <value>hdfs:///apps/webhcat/hadoop-streaming.jar</value>
        <description>The path to mapreduce streaming jar.</description>
    </property>
    <property>
        <name>templeton.hive.properties</name>
        <value>hive.metastore.local=false,hive.metastore.uris=thrift://metadatahostname:9083,hive.metastore.sasl.enabled=true,hive.metastore.execute.setugi=true,hive.exec.mode.local.auto=false,hive.metastore.kerberos.principal=hive/_HOST@BGDATA.COM</value>
        <description>Mainly forcus on the hive metastore properties</description>
    </property>
```
+ Add TEMPLETON_HOME into WebHCat Server Environment Advanced Configuration Snippet (Safety Valve)
    `TEMPLETON_HOME=/opt/cloudera/parcels/CDH-5.7.1-1.cdh5.7.1.p0.11/lib/hive-hcatalog`
+ Prepare and upload hive archive into HDFS
    - Step into hive directory and run. tar -hzcvf hive.tar.gz ./ , eg: `cd /opt/cloudera/parcels/CDH-5.7.1-1.cdh5.7.1.p0.11/lib/hive and run tar -hzcvf hive.tar.gz ./`
+ Prepare and upload pig archive into HDFS
    - In order to talk with HCatalog we need add 
        `HIVE_HOME=/opt/cloudera/parcels/CDH/lib/hive
         HCAT_HOME=/opt/cloudera/parcels/CDH/lib/hcatalog` 
    into pig executable file.
    - Step into pig directory and run. tar -hzcvf pig.tar.gz ./ , eg: `cd /opt/cloudera/parcels/CDH-5.7.1-1.cdh5.7.1.p0.11/lib/pig and run tar -hzcvf pig.tar.gz ./`
+ Upload all into hdfs, as /apps/wehbcat/..
    - hadoop fs -copyFromLocal hive.tar.gz /apps/webhcat/hive.tar.gz
    - hadoop fs -copyFromLocal pig.tar.gz /apps/wehbcat/pig.tar.gz
    - hadoop fs -copyFromLocal hadoop-streaming.jar /apps/wehbcat/hadoop-streaming.jar

+ We might need change HTTP user id to make sure more than 1000.
    - usermod -u 10001 HTTP
+ Done and restart server.
+ You might met permission issue to write job details into /templeton-hadoop/jobs.  Give write pemission for all user to /templeton-hadoop/jobs
