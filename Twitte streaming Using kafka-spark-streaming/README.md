To do this, we are going to set up an environment that includes 
* a single-node Kafka cluster
* a single-node Hadoop cluster
* Hive and Spark

---
## 1. VM Setup

 [Video](https://www.youtube.com/watch?v=x5MhydijWmc&ab_channel=ProgrammingKnowledge)
---
## 2. Install Kafka 

``` bash
 wget http://apache.claz.org/kafka/2.2.0/kafka_2.12-2.2.0.tgz
```
After that you will need to unpack it. At this point, I also like to rename the Kafka to something a little more concise.

``` bash
tar -xvf kafka_2.12-2.2.0.tgz
mv kafka_2.12-2.2.0.tgz kafka
```

Before continuing with Kafka, we'll need to install Java.

``` bash
sudo apt install openjdk-8-jdk -y
```

Test the Java installation by checking the version. 

``` bash
java -version
```

``` bash
pip3 install kakfa-python 
```

Confirm this installation with 

``` bash
pip3 list | grep kafka
```

---
## 3. Install Hadoop



``` bash
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz
tar -xvf hadoop-2.8.5.tar.gz
mv hadoop-2.8.5.tar.gz hadoop
cd hadoop
~/hadoop$ pwd
/home/<USER>/hadoop
```

Edit `.bashrc` found in your home directory, and add the following.

``` bash
export HADOOP_HOME=/home/<USER>/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

``` bash
source .bashrc
```

Next, we will need to edit/add some configuration files. From the Hadoop home folder (the one named `hadoop` that is in your home directory), `cd` into `etc/hadoop`.

Add the following to `hadoop-env.sh`

``` bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-"/home/<USER>/hadoop/etc/hadoop"}
```

Replace the file `core-site.xml` with the following:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

Replcae the file `hdfs-site.xml` with the following:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.permission</name>
        <value>false</value>
    </property>
</configuration>
```

As mentioned earlier, we are just setting up a single-node Hadoop cluster. This isn'y very realistic, but it works for this example. For this to work, we need to allow our machine to SSH into itself.

First, install SSH with 

``` bash
sudo apt install openssh-server openssh-client -y
```

Then, set up password-less authentication

``` bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

Then, try SSH-ing into the machine (type `exit` to quit the SSH session and return)

``` bash
ssh localhost
```

With Hadoop configured and SSH setup, we can start the Hadoop cluster and test the installation.

``` bash
hdfs namenode -format
start-dfs.sh
```
Finally, test that Hadoop was correctly installed by checking the Hadoop Distributed File System (HDFS):

``` bash
hadoop fs -ls /
```
---
## 4. Install Hive

``` bash
wget http://archive.apache.org/dist/hive/hive-2.3.5/apache-hive-2.3.5-bin.tar.gz
tar -xvf apache-hive-2.3.5-bin.tar.gz
mv apache-hive-2.3.5-bin.tar.gz hive
```
Add the following to your `.bashrc` and run it with `source`
``` bash
export HIVE_HOME=/home/<USER>/hive
export PATH=$PATH:$HIVE_HOME/bin
```

Give it a quick test with 
``` bash
hive --version
```

Add the following directories and permissions to HDFS

``` bash
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -mkdir -p /tmp
hadoop fs -chmod g+w /user/hive/warehouse
hadoop fs -chmod g+w /tmp
```

Inside `~/hive/conf/`, create/edit `hive-env.sh` and add the following

``` bash
export HADOOP_HOME=/home/<USER>/hadoop
export HADOOP_HEAPSIZE=512
export HIVE_CONF_DIR=/home/<USER>/hive/conf
```

While still in `~/hive/conf`, create/edit `hive-site.xml` and add the following

``` xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:derby:;databaseName=/home/davis/hive/metastore_db;create=true</value>
        <description>JDBC connect string for a JDBC metastore.</description>
    </property>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
        <description>location of default database for the warehouse</description>
    </property>
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://localhost:9083</value>
        <description>Thrift URI for the remote metastore.</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.apache.derby.jdbc.EmbeddedDriver</value>
        <description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.PersistenceManagerFactoryClass</name>
        <value>org.datanucleus.api.jdo.JDOPersistenceManagerFactory</value>
        <description>class implementing the jdo persistence</description>
    </property>
    <property>
        <name>hive.server2.enable.doAs</name>
        <value>false</value>
    </property>
</configuration>
```

(optional) Since Hive and Kafka are running on the same system, you'll get a warning message about some SLF4J logging file. From your Hive home you can just rename the file

``` bash
~/hive$ mv lib/log4j-slf4j-impl-2.6.2.jar lib/log4j-slf4j-impl-2.6.2.jar.bak
```

Now we need to create a database schema for Hive to work with using `schematool`

``` bash
~$ schematool -initSchema -dbType derby
```

We are now ready to enter the Hive shell and create the database for holding tweets. First, we need to start the Hive Metastore server with the following command.

``` bash
~$ hive --services metastore
```

This should give some output that indicates that the metastore server is running. You'll need to keep this running, so open up a new terminal tab to continue with the next steps.

Now, enter the Hive shell with the `hive` command

``` bash
~$ hive

 ...

hive> 
```

Make the database for storing our Twitter data:

``` bash
hive> CREATE TABLE tweets (text STRING, words INT, length INT)
    > ROW FORMAT DELIMITED FIELDS TERMINATED BY '\\|'
    > STORED AS TEXTFILE;
```

You can use `SHOW TABLES;` to double check that the table was created.

---
## 5. Install Spark

```bash
tar -xvf Downloads/spark-2.4.3-bin-hadoop2.7.tgz
mv spark-2.4.3-bin-hadoop2.7.tgz spark
```
ensuring Scala is installed on our system.

```bash
sudo apt install scala -y
```
Test with 
```bash
scala -version
```

Instead of writing Scala code, we will write our Spark transformer in Python, so we will need `pyspark`.

``` bash
pip3 install pyspark
```

Check with
```bash
pip3 list | grep spark
```

Now we need to add the Spark /bin files to the path, so open up `.bashrc` and add the following
```bash
export PATH=$PATH:/home/<USER>/spark/bin
export PYSPARK_PYTHON=python3
```
By setting the `PYSPARK_PYTHON` variable, we can use PySpark with Python3, the version of Python we have been using so far.
After running `source .bashrc`, try entering the PySpark shell
```bash
pyspark
 ...
Using Python version ....
SparkSession available as 'spark'.
>>> 
```
One last thing! We need a JAR file that wasn't included in PySpark that will allow us to connect to Kafka. Download the JAR file with the artifact ID `spark-streaming-kafka-0-8-assembly_2.11` from [search.maven.org](https://search.maven.org/search?q=a:spark-streaming-kafka-0-8-assembly_2.11).

---

