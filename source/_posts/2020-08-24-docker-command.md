---
layout: post
title:  "Docker Commands"
date:   2020-08-24 00:00:00 +0900
categories: docker
---

### Dockerfile vs Image vs Container

I thought that they are really different things before, but It's not true. Actually they are like the concept of the program and process


It's the summaries about the docker, which are gotten when i've struggled for getting the jvm option the java on the docker. 

### The useful command of Docker.

#### docker run

This command is usually used when you want to create the docker image and runs it. It's a lot of the options so 
It's important to know about these options for using this command well.

'-d' option is for executing your docker image on the background. 'd' spelling might means 'daemon'. 
Having you done this, You could see the process of the launching your image.

```
> docker run -d
```

'-p' option is for setting the port of your docker container and host system. It is usually used like the below command
and first one is for the host system and the later is for the your container. they are connected each other.
so The data are shoot to the host port would be transferred to the container port was connected. 
38080 is for the host system and 38081 is for the container on the view of the below command.
 
```
> docker run -p 38080:38081
```

'-e' option is for setting 'env' values. this values are often needed when you would like to get some data of outside of the container.
for example host's ip, host's name or the thing that we could not get in the container easily. and This spelling might means the 'environment variable'.

```
> docker run -e host.ip=123.123.123.123
```

You can set the image name using '--name' option when you use the 'docker run' command. It will give the alias for your container
It might be needed well because the CONTAINER_ID are too difficult to remember. so Whenever you write your command on the docker CLI,
you could see yourself who would use the alias of the docker image. If you don't use this option, then your image alias will create automatically using the name pool.

```
> docker run -name kidongyun
``` 

'-h' option is for giving the host name value to your container. Actually I don't know the default value but
I recommend use this option if you need to get the hostname of your host system in the docker container.

```
> docker run -h kidongyun
```

'-v' option can connect the some files from host to the containers. in other words, You can mount the files. If you done this,
Your container could get the host directories not the directories are copied from the host. That is you can access the modified things
when you change your source code in host.

```
> docker run -v /home/docker/kidong/identidock/app:/app kidongyun
```

```
> docker inspect kidongyun
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/docker/kidong/identidock/app",
                "Destination": "/app",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
```

#### docker ps

This command would be used a lot of the time, This command give to use the information related to the docker container.
for example the container name, id, the time when it's started, the execution status of the container and so on.
They are usually useful information to us. so Don't forget this command.

```
> docker ps
```

Having you used '-a' option, It will give the all information about the containers included are not worked.
I've used this option when i need to remove the containers are not operated in my case.

```
> docker ps -a
```

#### docker stop

You can stop your docker container using this command. after this, you might not be able to see this container on 'docker ps' command
for seeing this, you should attach the option '-a' when you use the 'docker ps' command.

```
> docker stop kidong yun
```

#### docker rm / docker rmi

If you want to eliminate the docker container, then use this command. It's do similar with the linux command so You could get this easily.
and The second command is for removing the docker image not container.

```
> docker rm kidongyun
> docker rmi kidongyun
```

#### docker inspect [CONTAINER_ID] or [CONTAINER_NAME]

Having you wanted to know about the detail of your docker container, This command will give them you want.
It has really a lot of the information like the status of network, environment variables, the information associated with host system
and the everything, which i think. 

```
> docker inspect kidongyun
```

#### docker exec -it [CONTAINER_ID] /bin/bash

We can open the shell of the docker container using this command. '-it' options actually could be separated with '-i' and '-t'
but We usually use this like that. This options mean that I would like to interact with the my container and please open the tty
between host and the container. and We execute the /bin/bash program on the container and we could communicate using this shell named 'bash'

#### docker build --tag [IMAGE_ID]:[IMAGE_TAG] [DIRECTORY_PATH]

You can create the images using this command. This command is needed the directory path included the DockerFile with the information 
related to the setting about the image. and You could also create the images differently as the tag name if you want.
in my company, This tag use as the release number of some business.

#### docker network ls

Having you wanted to see the specification of your docker network information, Let follow the below command.

```
    > docker network ls
```

### The sort of network type.

The communication way between host system and the containers is do similar with the concept of real network thing.
when you install the docker program, Your system would be installed new NIC(Network Interface Card might be virtual thing).
This NIC is called docker0 and you can see about these information at the below command.

```
    > route

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth0
link-local      0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
180.70.96.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0

```

#### bridge

It's the default way of the docker network. This way always use the docker0 interface. It's the new virtual network area.
So They've been in the independent place. It means you cannot access them at anywhere, anytime. the docker0 network usually 
has the 172.17.0.0/16 network. and whenever you add the new host, their ip is increase from the 172.17.0.1. Every container has
the their own network area. so It's also impossible to communicate with each containers. in other words, The bridge network type is
that give to each containers independent network area.

The below command could get the detail of the bridge network setting specification.

```
    > docker network inspect bridge
```

#### host

This way share the network area the host and each containers are used this type. You have to use the option '--net=host'
like the below.

```
    > docker run --net=host httpd web01
```

You can check using the below command whether your starting is right or not.

```
    > docker exec web01 ip addr show
```

#### 3. container

This way share the network area with another container.

#### 4. none

Having you picked this option, This container would have the isolated network, which is not with interface.

```
> docker run --name web04 --net=none -d httpd
```

[docker@tourDNDCvm71 ~]$ route