---
title: "CFSSL 工具使用"
date: 2023-08-09T17:10:25+08:00
draft: false
categories: ['奇妙的世界']
tags: ['command', 'tool']
---

需要用到的工具：

- cfssl，生成私钥，公钥，证书等信息，输出的为json格式
- cfssljson，把上面的 json 内容输出格式为 pem 的文件
- cfssl-certinfo，查看证书信息

可到 [Github](https://github.com/cloudflare/cfssl) 直接下载可执行文件


## 1. 生成 CA 证书

ca 证书为“根证书”，可信链条上的根节点，一般内置在浏览器或操作系统中；为方便使用 cfssl 创建证书，我们先创建一下配置文件 ca-config.json，用来在证书签名时方便选择场景应用：

```json
{
    "signing": {
        "default": {
            "expiry": "43800h"
        },
        "profiles": {
            "server": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

随后，创建 CA 证书的详细信息文件，也是个 json：ca.json

```json
{
    "CN": "SelfSignedCa",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Guangdong",
            "L": "Guangzhou",
            "O": "Hello.Inc",
            "OU": "System"
        }
    ]
}
```

| 字段 | 字段说明 |
|------|---------|
| CN   | common name, 一般代表网站地址 |
| C    | country, 国家代码，如 CN, US |
| ST   | state，州，省 |
| L    | locality, 城市 |
| O    | organization 组织名称 |
| OU   | 其他信息 |


使用以下命令创建 CA 证书

```bash
# cfssl工具生成的结果都为 json 格式，需要使用 cfssljson 进行转义为证书文件，-bare 指定证书文件名前缀
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

```

将会生成三个文件

1. ca.pem, ca 证书
2. ca-key.pem ca的私钥
3. ca.csr ca证书签名请求，生成 ca.pem 的中间产物，无用，但可以用来在证书过期后进行重签名

## 2. 使用 CA 签名其他证书

为证书生成配置文件，如为 etcd 使用的证书配置：etcd-csr.json
```json
{
    "CN": "etcd",
    // usage for website , please add hosts
    // hosts: ["www.xxx.com"]
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Guangdong",
            "L": "Guangzhou",
            "O": "Hello.Inc",
            "OU": "System"
        }
    ]
}
```
使用命令 
```bash
# 将会生成三个文件 etcd.pem, etcd-key.pem, etcd.csr
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer etcd-csr.json | cfssljson -bare etcd
```

## 3. 查看
使用 cfssl-certinfo 查看证书 `cfssl-certinfo -cert etcd.pem` 

