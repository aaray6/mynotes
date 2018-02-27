---
title: docker
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
```
