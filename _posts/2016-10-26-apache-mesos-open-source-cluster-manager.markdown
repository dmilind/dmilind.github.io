---
title: "Apache Mesos: Open source Cluster Manager"
layout: post
date: 2016-10-26
image: /assets/images/markdown.jpg
headerImage: false
tag:
- apache mesos
- cluster manager
category: blog
author: Milind 
description: Apache Mesos: Open source Cluster Manager
---

## Summary:

Apache mesos was the project in University of California , Berkeley. In that project team came up with a solution to mange the cluster. Now what is cluster ? This could be the basic question but let me answer you to understand whole picture of Apache mesos.

What is Cluster ?
When we are talking about the data-center , then cluster means a group of servers and other resources that act like a single system to perform the operations. For an example one application can be run on 3 servers and also other tools will be required to make it happen. so these are all resources and a single entity of these resources is called cluster.

To make operation efficient on this cluster, we need a tool who will act as a manager so that tool is called Apache mesos. Mesos provides facilities of application scheduling, scaling , and fault-tolerance. Mesos have following different components.

Mesos Master Daemon :
This is a program which runs on master node. And the work of mesos master is to manage agent nodes.(agent node means slave nodes but slave is depreciated )
Mesos Agent Daemon:
This is a program runs on the agent node and runs tasks that belongs to framework

Framework:
Consider framework is an application for mesos.

Apache Zookeeper:
This is a software which is used to co-ordinate a master node.
<img class="image" src="https://dmilind.github.io/assets/images/mesos-arch.png" alt="Alt Text">

<img class="image" src="https://dmilind.github.io/assets/images/mesos-arch02.jpg" alt="Alt Text">

Here is the architecture diagram of Mesos. To maintain the quorum at least 3 server should be there and number of server is always odd number. For the production environment, quorum must be of 5. So in quorum 3 servers are present and out of them 1 will be the leader and others will be on standby. Now the question is how the leader is selected ? So this leader is selected with the help of a software which is called Apache Zookeeper.
Mesos can also be called as compute engine. So mesos will provide resource allocation. These resources consist of CPU , RAM , Storage. Now lets consider how this mesos works.

<img class="image" src="https://dmilind.github.io/assets/images/mesos.jpg" alt="Alt Text">

In this fig. We have mesos installed and configured on cluster. Here we have two agent nodes means servers. Now at one instance of time. lets consider Agent 1 is idle, So Agent 1 will pass the resource information to mesos master. This information will be passed using mesos agent which is installed on agent. For an example, Agent 1 passed information that it has 4 CPU and 4 GB free. Now mesos master got this requirement.Master has some policy module and with the help of that it is decided that all requirement should passed to framework 1. Now at framework, Application is there and jobs are there. So when framework got requirement, scheduler on framework will reply back to master that ,  it has 2 tasks to run, first task is taking 2 CPU and 1 GB RAM and other one will take 1 CPU and 2 GB RAM. Once mesos master received this information, Master will ask framework executor to run these tasks on the agent node.Still we have 1 CPU and 1 GB RAM left, allocation policy module can offer that to framework 2.

Using mesos, our resource is properly utilized and tasks are getting executed under supervision. The application which can monitor and manage the tasks is Marathon.
Marathon can be consider as a micro-service.

