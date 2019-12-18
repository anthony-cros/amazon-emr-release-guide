# Configuring Applications to Use a Specific Java Virtual Machine<a name="configuring-java8"></a>

Java 8 is the default Java virtual machine \(JVM\) for cluster instances created using Amazon EMR release version 5\.0\.0 or later\. To override this JVM setting—for example, to use Java 8 with a cluster created using Amazon EMR version 4\.8\.0—set `JAVA_HOME` for an application by supplying the setting to its environment classification, `application-env`\. For Hadoop and Hive, for example:

```
[
    {
        "Classification": "hadoop-env", 
        "Configurations": [
            {
                "Classification": "export", 
                "Configurations": [], 
                "Properties": {
                    "JAVA_HOME": "/usr/lib/jvm/java-1.8.0"
                }
            }
        ], 
        "Properties": {}
    }
]
```

For Spark, if you are writing a driver for submission in cluster mode, the driver uses Java 7\. However, setting the environment can ensure that the executors use Java 8\. To do this, we recommend setting both Hadoop and Spark classifications:

```
[
    {
        "Classification": "hadoop-env", 
        "Configurations": [
            {
                "Classification": "export", 
                "Configurations": [], 
                "Properties": {
                    "JAVA_HOME": "/usr/lib/jvm/java-1.8.0"
                }
            }
        ], 
        "Properties": {}
    }, 
    {
        "Classification": "spark-env", 
        "Configurations": [
            {
                "Classification": "export", 
                "Configurations": [], 
                "Properties": {
                    "JAVA_HOME": "/usr/lib/jvm/java-1.8.0"
                }
            }
        ], 
        "Properties": {}
    }
]
```

## Service ports<a name="configuring-java8-service-ports"></a>

The following are YARN and HDFS service ports\. These settings reflect Hadoop defaults\. Other application services are hosted at default ports unless otherwise documented\. For more information, see the application's project documentation\.


**Port Settings for YARN and HDFS**  

| Setting | Hostname/Port | 
| --- | --- | 
| fs\.default\.name | default \(hdfs://emrDeterminedIP:8020\)  | 
| dfs\.datanode\.address | default \(0\.0\.0\.0:50010\)  | 
| dfs\.datanode\.http\.address | default \(0\.0\.0\.0:50075\)  | 
| dfs\.datanode\.https\.address | default \(0\.0\.0\.0:50475\) | 
| dfs\.datanode\.ipc\.address | default \(0\.0\.0\.0:50020\) | 
| dfs\.http\.address | default \(0\.0\.0\.0:50070\)  | 
| dfs\.https\.address | default \(0\.0\.0\.0:50470\)  | 
| dfs\.secondary\.http\.address | default \(0\.0\.0\.0:50090\) | 
| yarn\.nodemanager\.address | default \($\{yarn\.nodemanager\.hostname\}:0\)  | 
| yarn\.nodemanager\.localizer\.address  | default \($\{yarn\.nodemanager\.hostname\}:8040\) | 
| yarn\.nodemanager\.webapp\.address | default \($\{yarn\.nodemanager\.hostname\}:8042\) | 
| yarn\.resourcemanager\.address | default \($\{yarn\.resourcemanager\.hostname\}:8032\) | 
| yarn\.resourcemanager\.admin\.address | default \($\{yarn\.resourcemanager\.hostname\}:8033\) | 
| yarn\.resourcemanager\.resource\-tracker\.address | default \($\{yarn\.resourcemanager\.hostname\}:8031\) | 
| yarn\.resourcemanager\.scheduler\.address | default \($\{yarn\.resourcemanager\.hostname\}:8030\) | 
| yarn\.resourcemanager\.webapp\.address | default \($\{yarn\.resourcemanager\.hostname\}:8088\) | 
| yarn\.web\-proxy\.address | default \(no\-value\)  | 
| yarn\.resourcemanager\.hostname | emrDeterminedIP | 

**Note**  
The term *emrDeterminedIP* is an IP address that is generated by the Amazon EMR control plane\. In the newer version, this convention has been removed, except for the `yarn.resourcemanager.hostname` and `fs.default.name` settings\.

## Application users<a name="configuring-java8-application-users"></a>

Applications run processes as their own user\. For example, Hive JVMs run as user `hive`, MapReduce JVMs run as `mapred`, and so on\. This is demonstrated in the following process status example:

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
hive      6452  0.2  0.7 853684 218520 ?       Sl   16:32   0:13 /usr/lib/jvm/java-openjdk/bin/java -Xmx256m -Dhive.log.dir=/var/log/hive -Dhive.log.file=hive-metastore.log -Dhive.log.threshold=INFO -Dhadoop.log.dir=/usr/lib/hadoop
hive      6557  0.2  0.6 849508 202396 ?       Sl   16:32   0:09 /usr/lib/jvm/java-openjdk/bin/java -Xmx256m -Dhive.log.dir=/var/log/hive -Dhive.log.file=hive-server2.log -Dhive.log.threshold=INFO -Dhadoop.log.dir=/usr/lib/hadoop/l
hbase     6716  0.1  1.0 1755516 336600 ?      Sl   Jun21   2:20 /usr/lib/jvm/java-openjdk/bin/java -Dproc_master -XX:OnOutOfMemoryError=kill -9 %p -Xmx1024m -ea -XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode -Dhbase.log.dir=/var/
hbase     6871  0.0  0.7 1672196 237648 ?      Sl   Jun21   0:46 /usr/lib/jvm/java-openjdk/bin/java -Dproc_thrift -XX:OnOutOfMemoryError=kill -9 %p -Xmx1024m -ea -XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode -Dhbase.log.dir=/var/
hdfs      7491  0.4  1.0 1719476 309820 ?      Sl   16:32   0:22 /usr/lib/jvm/java-openjdk/bin/java -Dproc_namenode -Xmx1000m -Dhadoop.log.dir=/var/log/hadoop-hdfs -Dhadoop.log.file=hadoop-hdfs-namenode-ip-10-71-203-213.log -Dhadoo
yarn      8524  0.1  0.6 1626164 211300 ?      Sl   16:33   0:05 /usr/lib/jvm/java-openjdk/bin/java -Dproc_proxyserver -Xmx1000m -Dhadoop.log.dir=/var/log/hadoop-yarn -Dyarn.log.dir=/var/log/hadoop-yarn -Dhadoop.log.file=yarn-yarn-
yarn      8646  1.0  1.2 1876916 385308 ?      Sl   16:33   0:46 /usr/lib/jvm/java-openjdk/bin/java -Dproc_resourcemanager -Xmx1000m -Dhadoop.log.dir=/var/log/hadoop-yarn -Dyarn.log.dir=/var/log/hadoop-yarn -Dhadoop.log.file=yarn-y
mapred    9265  0.2  0.8 1666628 260484 ?      Sl   16:33   0:12 /usr/lib/jvm/java-openjdk/bin/java -Dproc_historyserver -Xmx1000m -Dhadoop.log.dir=/usr/lib/hadoop/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/usr/lib/hadoop
```