# hive-on-spark
* hive on spark tutorial.This tutorial is for some one who news to Hive and want to use the mode of hive on spark.
## English version see [offical doc](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Spark%3A+Getting+Started)
## [中文版本教程](http://35.185.190.89)

* 首先请确保你环境中拥有Spark，Hadoop，Hive，并正常运行。
* 本人使用的环境如下：

hadoop |  spark | hive
-------|--------|------
2.7.4  | 1.6    |2.1

1.  查看hive所使用的版本，到官方托管源码处查看当前版本所依赖的spark版本。
2.  下载步骤1中所依赖的spark版本。
3.  使用maven对spark源码进行编译，编译完成后会生成spark-$YOUR_SPARK_VERSION-bin-hadoop2-without-hive.tgz，将其解压

    a.  在spark2.0以前，可以使用如下命令
    
    ```
    ./make-distribution.sh --name "hadoop2-without-hive" --tgz "-Pyarn,hadoop-provided,hadoop-2.4,parquet-provided"
    ```
    
    b.  spark2.0以后，可以使用以下命令
    
    ```
    ./dev/make-distribution.sh --name "hadoop2-without-hive" --tgz "-Pyarn,hadoop-provided,hadoop-2.7,parquet-provided"
    ```
4.  配置Hive

    a.  在Hive2.2.0以前的版本，请在$HIVE_HOME/lib下创建spark-assembly-$HADOOP_VERSION.jar的软链接至步骤3中lib下的spark-assembly-xxx.jar
    
    b.  在Hive2.2.0以后的版本，若是YARN模式，需要创建软链接至以下jar包中
    
        1.  scala-library
        
        2.  spark-core
        
        3.  spark-network-common
        
    c.  在Hive2.2.0以后的版本，若是本地模式，需要在$HIVE_HOME/lib额外创建软链接至以下jar包中
    
        1.  chill-java，chill，jackson-module-paranamer,jackson-module-scala,jersey-container-servlet-core
        
        2.  jersey-server,json4s-ast,kryo-shaded,minlog,scala-xml,spark-launcher
        
        3.  spark-network-shuffle,spark-unsafe,xbean-asm5-shaded
   
5.  Hive相关配置

    ```
    set hive.execution.engine=spark;
    set spark.home=$SPARK_HOME;
    set spark.master=yarn;     //(OR <Spark Master URL>)
    set spark.eventLog.enabled=true;
    set spark.eventLog.dir=<Spark event log folder (must exist)>  // hdfs://master:9000/log
    set spark.executor.memory=1g;             
    set spark.serializer=org.apache.spark.serializer.KryoSerializer;
    ```
6.  上传Jar包至HDFS中
  
    a.  若在hive2.2.0以前，上传spark-assembly-xxx.jar至HDFS中（hdfs://xxxx:9000/spark-assembly.jar），并将下列代码添加至hive-site.xml中
    
    ```
    <property>
      <name>spark.yarn.jar</name>
      <value>hdfs://xxxx:9000/spark-assembly.jar</value>
    </property>
    ```
    
    b.  若在hive2.2.0以后，上传$SPARK_HOME中JARS下的所有包至HDFS中（hdfs:///xxxx:9000/spark-jars），并将下列代码添加至hive-site.xml中
    
    ```
    <property>
      <name>spark.yarn.jars</name>
      <value>hdfs://xxxx:9000/spark-jars/*</value>
    </property>
    ```
7.  SPARK其它配置项请参考[官方文档](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-Spark)

* **Notice**

```
报错：
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/hadoop/fs/FSDataInputStream

解决方法：
在spark.env.sh中添加如下环境变量
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

```
报错：
java.lang.NoClassDefFoundError org/apache/hive/spark/client/Job

解决方法：
在hive-site中添加如下代码
<property>
    <name>spark.driver.extraClassPath</name>
    <value>hdfs://lib/hive-exec-2.0.1.jar</value>
</property>

```
