---
title: Docker notes
date: 2018-02-27 21:59:50
tags:
---
# Docker notes

## check node:latest system version

```console
docker run -it --rm \
> api \
> bash
root@8f56b2965b96:/usr/src/app# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
root@8f56b2965b96:/usr/src/app# which g++
/usr/bin/g++
root@8f56b2965b96:/usr/src/app# 
```

## Docker — 从入门到实践

very useful doc [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/)

## some commands

```console
docker build -t cxxhost .
docker run -d -v /home/quxr/dev/mycxxtest:/usr/src/app cxxhost
docker run -it --rm -v /home/quxr/dev/mycxxtest/mycode:/usr/src/app cxxhost bash
docker run -it --rm cxxhost bash

docker run -v /home/quxr/dev/mycxxtest:/usr/src/app cxxhost

docker-compose stop cxxhost
docker-compose rm -f cxxhost

docker-compose up --no-deps -d cxxhost

docker run -it --rm -v ~/dev/dizao/server/sacs_parser_builder/sacs_parser:/usr/src/app sacs_parser_builder bash

docker logs -f dizao_api_1
docker-compose restart api
docker-compose stop api
docker kill d768cabb9bcc
docker ps
docker-compose rm -f api
docker image ls
docker system df
docker image prune
docker image ls
docker image ls -a

```

## Dockerfile sample

```console
FROM node:latest
# Create app directory
RUN mkdir -p /usr/src/app \
    && apt-get update \
    && apt-get install -y cmake

WORKDIR /usr/src/app

VOLUME /usr/src/app
CMD [ "/usr/src/app/make.sh" ]
```

## dizao

1. build
2. compose
3. run
4. install node.js
5. npminstall
6. node addxxxx.js
7. node addxxxx.js

## Dockerfile for gradle and springboot

### gradle sample

[Dockerfile](https://github.com/aaray6/mytest/blob/master/mygradeltest/Dockerfile)

### gradle springboot sample

[Springboot with gradle image](https://github.com/aaray6/mytest/tree/master/mysprintboottest2)

[Springboot with jdk image, base gradlew](https://github.com/aaray6/mytest/tree/master/mysprintboottest)

[official gradle docker image](https://hub.docker.com/_/gradle/)

[official gradle docker image Dockerfile](https://github.com/keeganwitt/docker-gradle/blob/1fcbfdaa2566e3cf3fb055fbd1342f2aa462bb85/jdk8-alpine/Dockerfile)