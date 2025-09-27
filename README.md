## Description:

The idea is to develop a robust, distributed data processing infrastructure via a two-node CloudLab topology. The core deliverable is the provisioning and configuration of a high-availability Apache Hadoop 2.7.2 cluster, integrating HDFS for durable data storage and YARN for scalable resource management. Key architectural tasks include implementing passwordless SSH for secure, automated node communication, fine-tuning HDFS replication for optimal performance, and deploying a Fair Scheduler to ensure equitable resource allocation across multiple queues, minimizing latency and maximizing cluster utilization. This project validates a production-ready blueprint for internal big data environments.

NOTE: This is the guide for Windows system and thus MobaXterm must be installed.

Prerequisites

    CloudLab account

    Access to Linux/MacOS terminal

    SSH key configured on CloudLab


## Hadoop Cluster Setup on CloudLab

This guide documents the full process of setting up a Hadoop 2.7.2 cluster with YARN on [CloudLab](!https://www.cloudlab.us/).

    Master Node: ctl

    Worker Nodes: cp-1, cp-2

The cluster will run HDFS and YARN with passwordless SSH, OpenJDK 8, and custom configuration files.

## SSH Key Generation & Passwordless Setup

1. Generate SSH Key (on localhost):

```
 ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2. Upload this SSH key in Cloudlab:

   Localhost: cat ~/.ssh/id_rsa.pub

   Copy the key

   Open Cloudlab

   Manage SSH keys

   Add key and save

## Setting passwordless nodes (localhost)

```
scp ~/.ssh/id_rsa* ctl_domain:~/.ssh/
scp ~/.ssh/id_rsa* cp-1_domain:~/.ssh/
scp ~/.ssh/id_rsa* cp-2_domain:~/.ssh/
```

## On Each Node (ctl, cp-1, cp-2)

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/id_rsa* ~/.ssh/authorized_keys
rm -rf ~/.ssh/known_hosts
echo 'StrictHostKeyChecking no' >> ~/.ssh/config
```

After this step, you should be able to ssh ctl, ssh cp-1, ssh cp-2 without entering a password.

## Install Java JDK

Run the following on all nodes (ctl, cp-1, cp-2):

```
sudo apt-get install -y openjdk-8-jdk
java -version
```

openjdk version "1.8.0_XXX"

## Download & Setup Hadoop

1. On Master Node (ctl), & worker nodes (cp-1, cp-2)

NOTE: Complete steps 1,2,3,4,5 in this section on one node and then move to the next node to repeat the 5 steps

```
wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.2/hadoop-2.7.2.tar.gz
tar -xvzf hadoop-2.7.2.tar.gz
mv hadoop-2.7.2 hadoop
```

2. Update environment variables:

```
echo export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64 >> ~/.bashrc
echo export HADOOP_HOME=~/hadoop >> ~/.bashrc
echo export HADOOP_PREFIX=~/hadoop >> ~/.bashrc
echo export HADOOP_YARN_HOME=~/hadoop >> ~/.bashrc
echo export HADOOP_CONF_DIR=~/hadoop/etc/hadoop >> ~/.bashrc
echo export YARN_CONF_DIR=~/hadoop/etc/hadoop >> ~/.bashrc
source ~/.bashrc
```

3. Edit Hadoop environment file:

```
nano ~/hadoop/etc/hadoop/hadoop-env.sh
```

4. In the first line of the file add:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64
```

5. Create an HDFS folder

```
sudo mkdir /dev/hdfs
sudo chmod 777 /dev/hdfs
```

## Configure Hadoop

You must configure the following files on all nodes (ctl, cp-1, cp-2) inside ~/hadoop/etc/hadoop/.

1. cd ~/hadoop/etc/hadoop
2. nano core-site.xml

```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://ctl:9000/</value>
  </property>
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/dev/hdfs</value>
  </property>
</configuration>
```
3. nano hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.blocksize</name>
    <value>268435456</value>
  </property>
  <property>
    <name>dfs.namenode.handler.count</name>
    <value>100</value>
  </property>
</configuration>
```

4. nano yarn-site.xml

yarn-site.xml

```
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>ctl</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
  </property>
  <property>
    <name>yarn.scheduler.fair.allocation.file</name>
    <value>~/hadoop/etc/fair-scheduler.xml</value>
  </property>
  <property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>32</value>
  </property>
  <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>65536</value>
  </property>
  <property>
    <name>yarn.nodemanager.hostname</name>
    <value>[nodemanager hostname]</value>
  </property>
  <property>
    <name>yarn.nodemanager.bind-host</name>
    <value>[nodemanager hostname]</value>
  </property>
</configuration>
```

NOTE: (change [nodemanager hostname] to the node you’re on: ctl, cp-1, or cp-2)

5. nano slaves

```
cp-1-0
cp-2-0
```

6. cd ..
   
7. touch fair_scheduler.xml
   
8. nano fair_scheduler.xml

nano fair_scheduler.xml

```
<allocations>
  <defaultQueueSchedulingPolicy>drf</defaultQueueSchedulingPolicy>
  <queue name="queue0">
    <weight>1</weight>
    <allowPreemptionFrom>false</allowPreemptionFrom>
    <schedulingPolicy>fifo</schedulingPolicy>
  </queue>
  <queue name="queue1">
    <weight>1</weight>
    <allowPreemptionFrom>true</allowPreemptionFrom>
  </queue>
</allocations>
```

## Start Services

On Master Node (ctl) -- format HDFS only once

```
hadoop/bin/hdfs namenode -format hdfs
```

Start DFS:

```
hadoop/sbin/start-dfs.sh
```

Start YARN:

```
hadoop/sbin/start-yarn.sh
```

## Verify Setup

On Master Node (ctl) -- Check running processes:

```
jps
```

## Check Status

Goto: http://<ctl_domain>:50070

NOTE: all domains must be from cloudlab







  










