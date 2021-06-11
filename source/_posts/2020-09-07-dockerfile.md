---
layout: post
title:  "Dockerfile"
date:   2020-09-09 00:00:00 +0900
categories: docker
---

### docker run oracle example

```
    docker run -d -p 59160:22 -p 59161:1521 jaspeen/oracle-xe-11g
```

## Dockerfile

### ENTRYPOINT

This keyword is used to execute some scripts when you start docker container. That is this keword related to the command 'docker run', 'docker start'
It can use only one time in Dockerfile.

### FROM [IMAGENAME:IMAGETAG]

it could define the image name and image tag. If you use this keyword then the dockerfile would import the image written with this. 
'lastest' would be default value if you don't specify image tag. 

```
FROM ubuntu:14.04
```

### BUILD CONTEXT

It's usually a directory, which have files is needed when you do docker build.

### ENV

You can set the environment variables using this keyword. and you can also get this like '${ENV_NAME}' or $ENV_NAME

```
/* SET */
ENV SCOUTER_SERVER=$scouter_ip

/* GET */
#{SCOUNTER_SERVER}
```

### VOLUME

This keyword can set the shareable directory with a host when you create container. and It could be array type.

```
VOLUME /home/volume
```

### ARG

It's similar with the parameters of programming languages. You can access some value is came from outside of docker container. it should be set with '--build-arg' option when you build dockerfile. 

```
docker build --build-arg my_arg=/home
```

### USER

### COPY

### AD

### CMD