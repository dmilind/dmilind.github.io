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

<img class="image" src="https://dmilind.github.io/assets/images/bridge.png" alt="Docker Bridge">

* Once the mapping is done, this docker bridge is able to access the data over the underline host’s network. 
* Now let's try to understand this by using Docker. On any docker host run below command.

This command shows all available docker networks in docker host. Network ID is phrased which is internally used by the docker. Name is the kind of network available for docker, Driver is the name of the driver which is used to create a network. And the scope is sort of boundary for that network. 

## Network Scope:

Network scope bounds the data transaction. This scope would be either local or global. When the scope is local that means all network connections are happening on a single docker host internally. This is good for a single container which does not want to reach outside of the local docker host. 

In a distributed micro-service architecture, one service is running in container want to fetch some data from another container, in this case networking scope need to be global. The global scope can be attained in swarm mode, where overlay network will be used. For now, lets now go into other than local scope and bridge network.
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

<img class="image" src="https://dmilind.github.io/assets/images/ipam.png" alt="ipam">

I think this is clear that how docker container will be wired to get data through Linux bridge for docker (docker0). In the bottom line, when the connection comes from the outside host. It goes to eth0, then it goes to routing table which routed the traffic to docker bridge (docker0) and then routed to veth of the container. And lastly, container finds it. 
Now let's play around the network and try to make a connection over the network. The network can be custom created also. Why do we need a custom network? This is purely dependent on the application architecture. Some time services wanted to run in custom network bridge so that containers are isolated from the default network bridge. 

Below command will create a custom network bridge named sample-test. 

```
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

```
docker@Docker:/mnt/sda1/var/lib/docker$ docker network inspect sample-net
[
    {
        "Name": "sample-net",
        "Id": "968da3ccc8a398a833879e071707bcced8b1804680c0a99ecbaca9aad9d2d193",
        "Created": "2019-05-01T19:47:40.070354154Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default"
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
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
        "Options": {},
        "Labels": {
    }
]
```

An assigned subnet is 172.19.0.0/16 and of course, gateway address would be 172.19.0.1.  A custom subnet can also be assigned to this network bridge bypassing --subnet flag in docker network create command also passing the value for that subnet. E.g 

```
docker network create --driver bridge --subnet “10.10.0.0/16” \ sample-net2
```

As we said all container will get IP’s from this range, let's demo it. Make sure while spinning any container “--network” flag is passed on docker run command. If this is not specified then as per docker’s default behavior all container will get spun in default network which is docker0 
Let's run-first centos container 

```
docker@Docker:/mnt/sda1/var/lib/docker$ docker container run -it --name C1 --network sample-net centos /bin/sh
```

Now you are into centos container since this is the first container spun up in sample-net network then IP address should be in the range of 172.19.0.0
Inside the container run command: 

```
sh-4.2# cat /etc/hosts

127.0.0.1 localhost

::1 localhost ip6-localhost ip6-loopback

fe00::0 ip6-localnet

ff00::0 ip6-mcastprefix

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

172.19.0.2 459083dfd765
```

As expected right IP address has been assigned to the container. Now if you inspect sample-net network bridge, then you can see this container is mapped under sample-net network, 

```
docker@Docker:~$ docker network inspect sample-net

[

    {

        "Name": "sample-net",

        "Id": "968da3ccc8a398a833879e071707bcced8b1804680c0a99ecbaca9aad9d2d193",

        "Created": "2019-05-01T19:47:40.070354154Z",

        "Scope": "local",

        "Driver": "bridge",

        "EnableIPv6": false,

        "IPAM": {

            "Driver": "default",

            "Options": {},

            "Config": [

                {

                    "Subnet": "172.19.0.0/16",

                    "Gateway": "172.19.0.1"

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

        "Containers": {

            "459083dfd76555cd238bb1cbd8f3bda235f662f7937427cea0fbb23e271c6daa": {

                "Name": "C1",

                "EndpointID": "26197f5319144702dc23aed6b6fa0dd1dd997cd8a5d033d62105d273de41d534",

                "MacAddress": "02:42:ac:13:00:02",

                "IPv4Address": "172.19.0.2/16",

                "IPv6Address": ""

            }

        },

        "Options": {},

        "Labels": {}

    }

]
```

Let's spin another container in same network bridge: 

```
docker container run -it --name C2 --network sample-net \ centos /bin/sh 

sh-4.2# cat /etc/hosts

127.0.0.1 localhost

::1 localhost ip6-localhost ip6-loopback

fe00::0 ip6-localnet

ff00::0 ip6-mcastprefix

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

172.19.0.3 ca4f195c6697
```

All IPs are allocated nicely. Now let's see how connection can be made between 2 containers under the same bridge network sample-net. Log in to both containers and try to ping each other. 

```
From C1:

sh-4.2# ping C2

PING C2 (172.19.0.3) 56(84) bytes of data.

64 bytes from C2.sample-net (172.19.0.3): icmp_seq=1 ttl=64 time=0.042 ms

64 bytes from C2.sample-net (172.19.0.3): icmp_seq=2 ttl=64 time=0.053 ms

64 bytes from C2.sample-net (172.19.0.3): icmp_seq=3 ttl=64 time=0.081 ms

^C

--- C2 ping statistics ---

3 packets transmitted, 3 received, 0% packet loss, time 2033ms

rtt min/avg/max/mdev = 0.042/0.058/0.081/0.018 ms 
From C2: 
sh-4.2# ping C1
PING C1 (172.19.0.2) 56(84) bytes of data.
64 bytes from C1.sample-net (172.19.0.2): icmp_seq=1 ttl=64 time=0.063 ms
64 bytes from C1.sample-net (172.19.0.2): icmp_seq=2 ttl=64 time=0.058 ms
64 bytes from C1.sample-net (172.19.0.2): icmp_seq=3 ttl=64 time=0.064 ms
64 bytes from C1.sample-net (172.19.0.2): icmp_seq=4 ttl=64 time=0.147 ms
64 bytes from C1.sample-net (172.19.0.2): icmp_seq=5 ttl=64 time=0.057 ms
64 bytes from C1.sample-net (172.19.0.2): icmp_seq=6 ttl=64 time=0.057 ms
^C
--- C1 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5062ms
rtt min/avg/max/mdev = 0.057/0.074/0.147/0.033 ms
```
The connection is successful: 
How these connections are getting established?When C1 is trying to ping C2, C1 sends packets to bridge network (sample-net), since sample-net has routing information about C2, so there is no problem of sending packets to C2. The same thing is for C2. 
Now try to spin a container in default docker network bridge (docker0).

```
docker@Docker:~$ docker container run -it --name C3 centos /bin/sh sh-4.2# cat /etc/hosts

127.0.0.1 localhost

::1 localhost ip6-localhost ip6-loopback

fe00::0 ip6-localnet

ff00::0 ip6-mcastprefix

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

172.17.0.2 d141687fb10d
```

Also, this container is assigned IP from docker0 range. 
Now let's try to ping C3 from C1 and vice versa: 

```
sh-4.2# ping C4
ping: C4: Name or service not known

Okay: service now is known, let's try with IP 


sh-4.2# ping 172.17.0.2

PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.

^C

--- 172.17.0.2 ping statistics ---

7 packets transmitted, 0 received, 100% packet loss, time 6127ms 
```

Hmm, so the connection is not successful even though both containers are running on the same host. I am sure the same output will be obtained from other hosts while pinging C3.This is the beauty of Linux namespace concept. 
But this might be the problem when spinning containers in different network bridges. Some time one container from one network bridge wants to connect to another container from other network bridge. In such cases, a subset of network bridge should be defined. 
For example, Create a new network bridge and call it as sample-net2
Now spin out the first container 

```
docker@Docker:/mnt/sda1/var/lib/docker$ docker container run -it --name C1-sample-net sample-net centos /bin/sh 
``` 
This container is attached to one network sample-net 
Time to spin the new container. This container should be connected to both networks sample-net as well as sample-net2 

```
docker@Docker:~$ docker run -it --name C2-sample-net2 --net sample-net2 --net sample-net centos /bin/sh
``` 

Now try to ping each other which is a success. 
Cool …. 
What are other network bridges

```
docker@Docker:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE

634204d08917           bridge                   bridge                   local

e227083495f1            docker_gwbridge bridge                   local

cc832f453824            host                       host                      local

24a7c93c8d9d           none                      null                       local

968da3ccc8a3           sample-net             bridge                   local

457184e60e78          sample-net2           bridge                   local

```

Different Container Network Model implementations.
* Host:         
        If you want to use underline host’s network bridge, you can attach your container to this network bridge. But this is highly insecure.  
* Null:         
          Some time if you want to isolate your container from any data traffic, then attach your container to this network bridge. 
* Overlay:           
          This network bridge is used when the scope is global. This will be used in docker swarm where the container will talk to each other over network pipeline. I will try to explain this in another blog. 

