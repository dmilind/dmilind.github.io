---
title: "Jenkins Within Vagrant Box"
layout: post
date: 2019-04-28
image: /assets/images/markdown.jpg
headerImage: false
tag:
- jenkinss
- vagrant
- jenkins vagrant box
category: blog
author: Milind 
description: Apache Mesos Open source Cluster Manager
---

## Objective:

 To get same versioned jenkins application running within a team in order to mitigate the fail over, vagrant box is built up in which Jenkins instance will be running. 

## Description:

While working in a team, every developer wants to test his code in sandbox environment. We do test our code continuously, for that we use Jenkins instance. After few days we realized that just to test the code , we needed to hop on to the central server where jenkins instance was running. But this jenkins was being used by other teams also which was a big confusion and mess. Then we decided to use local jenkins instance deployed on our own laptop. After few days we again realized that jenkins version varies from developer to developer. This does not matter in most of the cases. But being working in a security company. Trust me everything matters. What is being used ? what version is running ? What extra plugins has been deployed ? On and On.
Then jenkins in vagrant has been opted out. In this blog , I tried to pen down the process that I had followed to build jenkins inside vagrant box. 

## Steps performed to get jenkins vagrant box: 

* Centos/7 box has been used as a base vagrant box. This is the first layer I can say. All packages are built upon this layer.

* Get this vagrant box on your machine. (Expecting vagrant and virtual box has been installed

```
vagrant init centos/7
```

* To get jenkins installed on this machine I used inline shell provisioner.:
  Find the Vagrantfile here: <a href='https://github.com/dmilind/vagrant/blob/master/Vagrantfile'> 
* During vagrant up, all perquisites and jenkins application will be installed in the vagrant box. 
* Now next step is to package this box to new box which will be a base box for the jenkins. If you directly try to package it to new box, it will error out about virtual box guest addition.   To solve it , I used below script to add virtual box guest addition. This is useful while mounting the shared folder on local machine. This script should be run from vagrant box. ssh to     vagrant box (centos/7), become a root user, copy below script and execute it.

```
#!/usr/bin/env bash
set -e
yum update -y 
yum install wget gcc kernel-devel -y 
cd /opt
wget http://download.virtualbox.org/virtualbox/5.2.26/VBoxGuestAdditions_5.2.26.iso 
mount VBoxGuestAdditions_5.2.26.iso -o loop /mnt 
sh /mnt/VBoxLinuxAdditions.run --nox11
unmount /mnt 
rm -f VBoxGuestAdditions_5.2.26.iso
```

* Jenkins running on this host would be from the scratch , So I configured this jenkins instance according to our needs. The vagrant box that I uploaded on vagrant cloud is for general use. 
* Now this box can be used as a base box for the our jenkins box. 
* After completing all step , exit out from the vagrant box.
* Do vagrant halt and vagrant up to find any issue is there or not. 
* If all good then halt the machines and proceed to package the box.

```
vagrant package —output jenkins.box 
```
* Add this box to your local vagrant box cache. ( set your name for the box )

```
vagrant box add milinddhoke/jenkins jenkins.box 
```
* This box is ready to run jenkins application on http://192.168.33.10 at port 8080. 

## Usage

* Get this box from vagrant cloud.
* Run this command:
```
vagrant init milinddhoke/jenkins --box-version 1.0.0
```
* Make changes in the Vagrantfile Uncomment below lines from Vagrantfile, or you can make your own preferences.

```
config.vm.network "forwarded_port", guest: 80, host: 8080
config.vm.network "private_network", ip: "192.168.33.10"
```

* Now all good to go. Run below command.

```
vagrant up 
```

Once this command get through , your jenkins application is running on http://192.168.33.10:8080

Note:
Initial log in to jenkins would be:
Username : admin
Password: jenkins 
Vagrant box Link :
<a href='https://app.vagrantup.com/milinddhoke/boxes/jenkins'>

