---
title: sqlplus
date: 2020-05-13 20:14:27
tags:
---

To use sqlplus to test, do following

```console
export $ORACLE_HOME=/opt/oracleclient
export PATH=$PATH:$ORACLE_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
sqlplus user/pass@SID
```
