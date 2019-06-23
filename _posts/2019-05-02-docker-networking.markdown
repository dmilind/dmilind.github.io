---
title: "Docker Networking"
layout: post
date: 2019-05-02 
image: /assets/images/markdown.jpg
headerImage: false
tag:
- docker
- containerization
- docker networking
category: blog
author: Milind 
description: Docker Networking
---

## Summary:

Docker Networking

I was understanding networking in docker, and I found it is difficult to understand very easily. Some articles were not well explained and some of them were on a high pitch. So as an initial effort, I am trying to explain how docker is wired internally with the local host. 
The server on which docker containers are running is called as Docker host. When any container is running on a docker host, an embedded application running inside may need some data from outside. To get data inside a container some networking is needed. This networking is provided by docker. The model designed by docker is called Container Networking Model. In this model, the idea is, open a bridge to underline Linux network and start getting data from it. This CNM can be implemented in several ways. Out of that let's try to understand bridge (docker0). 

```
$ docker network ls
 NETWORK ID          NAME                DRIVER              SCOPE
 634204d08917        bridge              bridge              local
 e227083495f1        docker_gwbridge     bridge              local
 cc832f453824        host                host                local
 24a7c93c8d9d        none                null                local
```

* Bridge  
         When docker is installed freshly and docker daemon is started, this network bridge is automatically created and name it is as docker0. This bridge is then mapped into underlines host’s IP routing table. 

* Once the mapping is done, this docker bridge is able to access the data over the underline host’s network. 
* Now let's try to understand this by using Docker. On any docker host run below command.

This command shows all available docker networks in docker host. Network ID is phrased which is internally used by the docker. Name is the kind of network available for docker, Driver is the name of the driver which is used to create a network. And the scope is sort of boundary for that network. 

## Network Scope: 
          Network scope bounds the data transaction. This scope would be either local or global. When the scope is local that means all network connections are happening on a single docker host internally. This is good for a single container which does not want to reach outside of the local docker host. 
In a distributed microservice architecture, one service is running in container want to fetch some data from another container, in this case networking scope need to be global. The global scope can be attained in swarm mode, where overlay network will be used. For now, lets now go into other than local scope and bridge network.
Until now, the docker bridge is made available since we started the docker engine. When the docker bridge is created, the subnet range is assigned to this bridge internally. This somewhat defaults behavior of docker. Let's confirm what subnet has been assigned to docker bridge. Run below command and check for IPAM block which is highlighted. 

```
docker@Docker:~$ docker network inspect bridge
[
    {
       "Name": "bridge",
        "Id": "634204d08917c0a8153864953d1522e41f882aea25b84949073aee792faf1114",
        "Created": "2019-04-30T22:48:22.730817465Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

```

## What is IPAM:          
           IPAM means IP Address Management. This is a tool which is used to track the IP addresses. Under Config in IPAM block, Subnet is allocated to 172.17.0.0/16. That means this docker bridge can house IP addresses in the range of 172.17.0.1 - 172.17.255.254. Under that, you can see the IP address is assigned for the Gateway. Here Gateway is the router for docker0. So docker0 ’s  IP address is the first IP which is 172.19.0.1. 
Any container spun up in this bridge will start getting an IP address from 172.19.0.2.  By default container is allowed to have egress traffic is allowed and ingress traffic is blocked. Egress means outgoing traffic from the container and ingress means incoming traffic to the container. To maintain this behavior, Each container gets its own virtual Ethernet(veth) connection to the bridge. You can understand this from the below figure. 

Consider the below image:

## fig 04

I think this is clear that how docker container will be wired to get data through Linux bridge for docker (docker0). In the bottom line, when the connection comes from the outside host. It goes to eth0, then it goes to routing table which routed the traffic to docker bridge (docker0) and then routed to veth of the container. And lastly, container finds it. 
Now let's play around the network and try to make a connection over the network. The network can be custom created also. Why do we need a custom network? This is purely dependent on the application architecture. Some time services wanted to run in custom network bridge so that containers are isolated from the default network bridge. 

```
Below command will create a custom network bridge named sample-test. 

$docker network create --driver bridge sample-test
docker@Docker:/mnt/sda1/var/lib/docker$ docker network ls

NETWORK ID          NAME                DRIVER        SCOPE     
634204d08917        bridge              bridge        local    
e227083495f1        docker_gwbridge     bridge        local
cc832f453824        host                host          local
24a7c93c8d9d        none                null          local
968da3ccc8a3        sample-net          bridge        local
```

By default the scope is local. Means all container within this network will talk withing docker host. 
Let's see what subnet and gateway are assigned to this bridge network. 

## Fig 06 

An assigned subnet is 172.19.0.0/16 and of course, gateway address would be 172.19.0.1.  A custom subnet can also be assigned to this network bridge bypassing --subnet flag in docker network create command also passing the value for that subnet. E.g 

## fig 07 

As we said all container will get IP’s from this range, let's demo it. Make sure while spinning any container “--network” flag is passed on docker run command. If this is not specified then as per docker’s default behavior all container will get spun in default network which is docker0 
Let's run-first centos container 

## fig 08 

Now you are into centos container since this is the first container spun up in sample-net network then IP address should be in the range of 172.19.0.0
Inside the container run command: 

## fig 09 

As expected right IP address has been assigned to the container. Now if you inspect sample-net network bridge, then you can see this container is mapped under sample-net network, 

## fig 10 

Let's spin another container in same network bridge: 

## fig 11 

All IPs are allocated nicely. Now let's see how connection can be made between 2 containers under the same bridge network sample-net. Log in to both containers and try to ping each other. 

## fig 12 
The connection is successful: 
How these connections are getting established?When C1 is trying to ping C2, C1 sends packets to bridge network (sample-net), since sample-net has routing information about C2, so there is no problem of sending packets to C2. The same thing is for C2. 
Now try to spin a container in default docker network bridge (docker0).

## fig 13

Also, this container is assigned IP from docker0 range. 
Now let's try to ping C3 from C1 and vice versa: 

## fig 14 

Hmm, so the connection is not successful even though both containers are running on the same host. I am sure the same output will be obtained from other hosts while pinging C3.This is the beauty of Linux namespace concept. 
But this might be the problem when spinning containers in different network bridges. Some time one container from one network bridge wants to connect to another container from other network bridge. In such cases, a subset of network bridge should be defined. 
For example, Create a new network bridge and call it as sample-net2
Now spin out the first container 

## fig 15 
This container is attached to one network sample-net 
Time to spin the new container. This container should be connected to both networks sample-net as well as sample-net2 

## fig 16 

Now try to ping each other which is a success. 
Cool …. 
What are other network bridges

## fig 17 
Different Container Network Model implementations.
Host :         
           If you want to use underline host’s network bridge, you can attach your container to this network bridge. But this is highly insecure.  
Null :         
          Some time if you want to isolate your container from any data traffic, then attach your container to this network bridge. 
Overlay :           
          This network bridge is used when the scope is global. This will be used in docker swarm where the container will talk to each other over network pipeline. I will try to explain this in another blog. 

