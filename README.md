## Description:

The idea is to develop a robust, distributed data processing infrastructure via a two-node CloudLab topology. The core deliverable is the provisioning and configuration of a high-availability Apache Hadoop 2.7.2 cluster, integrating HDFS for durable data storage and YARN for scalable resource management. Key architectural tasks include implementing passwordless SSH for secure, automated node communication, fine-tuning HDFS replication for optimal performance, and deploying a Fair Scheduler to ensure equitable resource allocation across multiple queues, minimizing latency and maximizing cluster utilization. This project validates a production-ready blueprint for internal big data environments.

NOTE: This is the guide for Windows system and thus MobaXterm must be installed.

## Creating a cluster on Cloudlab

  Step 1: Create an account on Cloudlab and sign-up to either an existing project or start a new project.
  
  (Note: For my university, the existing project root is AMS560-SBU --> submit request --> sign up)

  Step 2: In Experiments --> Start Experiment 
  
            2.1. Select a profile:

                * Choose OpenStack
                * Choose Pike/Zed & use 2 nodes

            






