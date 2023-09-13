---
title: "Glassfish Usage"
date: 2023-09-12T14:57:44+08:00
draft: true
---

## 1. Install

https://glassfish.org/download.html

## 2. Usage

### 2.1 配置

找到文件 `glassfish/conf/asenv.conf`(win 下是 asenv.bat), 可以设定 glassfish 的 JDK 版本

```bash
# win
set AS_JAVA={JDK_LOCATION}
# linux
AS_JAVA={JDK_LOCATION}
```

### 2.2 Domain Manage

默认存在一个 `domain1` 的 domain 资源

```bash

# 启动默认的 domain 资源
./bin/asadmin start-domain

# 启动置顶的 domain
./bin/asadmin start-domain domain1

# 同样的操作还有 stop-domain create-domain

```

## 3. Resource

### 3.1 JDBC Connect Pool

首先创建 JDBC Connect Pool, MySQL datasource 可以适用 `com.mysql.cj.jdbc.MysqlDataSource`

再创建 JDBC Resource, 



