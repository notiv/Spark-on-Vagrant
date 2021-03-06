Spark on Vagrant
================================

# Introduction

Vagrant project to spin up a cluster of 4 virtual machines with Hadoop v2.6.0 and Spark v1.4.1 and a series of other data science related tools (iPython with various kernels etc.). Based on the excellent work of [Jee Vang](https://github.com/vangj/vagrant-hadoop-2.4.1-spark-1.0.1) and [Nick Tai](https://github.com/CorcovadoMing/vagrant-hadoop-2.4.1-spark-1.0.1).

1. node1 : HDFS NameNode + Spark Master + Anaconda
2. node2 : YARN ResourceManager + JobHistoryServer + ProxyServer
3. node3 : HDFS DataNode + YARN NodeManager + Spark Slave
4. node4 : HDFS DataNode + YARN NodeManager + Spark Slave

# Getting Started

1. [Download and install VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. [Download and install Vagrant](http://www.vagrantup.com/downloads.html).
3. Git clone this project, and change directory (cd) into this project (directory).
4. Run ```vagrant up``` to create the VM.
5. Run ```vagrant ssh``` to get into your VM.
6. Run ```vagrant destroy``` when you want to destroy and get rid of the VM.

Some gotcha's.

1. Make sure you download Vagrant v1.4.3 or higher.
2. Make sure when you clone this project, you preserve the Unix/OSX end-of-line (EOL) characters. The scripts will fail with Windows EOL characters.
3. Make sure you have 4Gb of free memory for the VM. You may change the Vagrantfile to specify smaller memory requirements.
4. This project has NOT been tested with the VMWare provider for Vagrant.
5. You may change the script (common.sh) to point to a different location for Hadoop and Spark to be downloaded from. Here is a list of mirrors for Hadoop: http://www.apache.org/dyn/closer.cgi/hadoop/common/.

# Advanced Stuff

If you have the resources (CPU + Disk Space + Memory), you may modify Vagrantfile to have even more HDFS DataNodes, YARN NodeManagers, and Spark slaves. Just find the line that says "numNodes = 4" in Vagrantfile and increase that number. The scripts should dynamically provision the additional slaves for you.

# Make the VMs setup faster
You can make the VM setup even faster if you pre-download the Hadoop, Spark, Oracle JDK and Anaconda into the /resources directory.

1. /resources/hadoop-2.6.0.tar.gz
2. /resources/spark-1.4.1-bin-hadoop2.6.tgz
3. /resources/jdk-7u51-linux-x64.gz
4. /resources/Anaconda3-2.3.0-Linux-x86_64.sh

The setup script will automatically detect if these files (with precisely the same names) exist and use them instead. If you are using slightly different versions, you will have to modify the script accordingly.

# Make sure YARN and Spark jobs can run
I typically run the following tests after post-provisioning on node1 (as root user). 

### Test YARN
Run the following command to make sure you can run a MapReduce job.

```
yarn jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 100
```

### Test Spark on YARN
You can test if Spark can run on YARN by issuing the following command. Try NOT to run this command on the slave nodes.

```
$SPARK_HOME/bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --num-executors 10 \
    --executor-cores 2 \
    $SPARK_HOME/lib/spark-examples*.jar \
    100
```

### Test code directly on Spark	
```
$SPARK_HOME/bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master spark://node1:7077 \
    --num-executors 10 \
    --executor-cores 2 \
    $SPARK_HOME/lib/spark-examples*.jar \
    100
```
Note: If you get the following warning:
```
Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources"
```
then reduce the number of executor cores to 1.

### Test Spark using Shell
Start the Spark shell using the following command. Try NOT to run this command on the slave nodes.

```
$SPARK_HOME/bin/spark-shell --master spark://node1:7077
```

Then go here https://spark.apache.org/docs/latest/quick-start.html to start the tutorial. Start by using the following command:
```
val textFile = sc.textFile("file:///vagrant/README.md")
```

You might also want to dive into the learn-scala folder as that is a companion Scala project to learn Spark.

### Test IPython Notebook
Start the Ipython Notebook using the following command:
```
ipython notebook --ip=0.0.0.0
```
Since there is no graphical environment in the virtual machine, the notebook is only accesible from the host. In a host browser enter the following URL http://10.211.55.101:8888

### Start the IRKernel
In order to make the R Kernel (IRKernel) available in the iPython notebooks, run the following command in the linux command line:
```
R -q -e "IRkernel::installspec()"
```

# Web UI
You can check the following URLs to monitor the Hadoop daemons or the IPython Notebook.

1. [NameNode] (http://10.211.55.101:50070/dfshealth.html)
2. [ResourceManager] (http://10.211.55.102:8088/cluster)
3. [JobHistory] (http://10.211.55.102:19888/jobhistory)
4. [Spark] (http://10.211.55.101:8080)
5. [Spark History] (http://10.211.55.101:18080)
6. [IPython Notebook](http://10.211.55.101:8888)

# Vagrant boxes
A list of available Vagrant boxes is shown at http://www.vagrantbox.es. 

# Vagrant box location
The Vagrant box is downloaded to the ~/.vagrant.d/boxes directory. On Windows, this is C:/Users/{your-username}/.vagrant.d/boxes.

# References
This project was kludge together with great pointers from all around the internet. All references made inside the files themselves.

# Copyright Stuff
Copyright 2014 Jee Vang

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
