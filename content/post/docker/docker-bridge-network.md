---
title: "Docker Dridge Network 紀錄"
date: 2022-05-14T18:43:01+08:00
description: "介紹 docker network 使用上的一些發現"
tags: [docker]
---

## docker network

[docs.docker/network](https://docs.docker.com/network/)

### 概述

docker 內的每個 container 都是一個封閉的環境，彼此之間不能互相溝通與干涉。

如果需要模擬多個服務之間互動行為，可以透過 docker network 來讓 container 之間透過網路進行連接。

docker 提供了多種不同 driver 的 network，讓 container 能完成在不同情境上的應用

container 啟動時預設使用 bridge network。

### 小常識

docker 啟動時會產生三個 network，分別是 `bridge`、`host`、`none`

其中 `bridge` 就是 container 預設使用的 bridge network

```shell
$ docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
67fc3098224b   bridge     bridge    local
6b6200a452a4   host       host      local
1f931a78dc58   none       null      local
```

## bridge network

[docs.docker/network/bridge](https://docs.docker.com/network/bridge/)

### 概述

bridge network 有兩種，default bridge 和 custom bridge

## default bridge network

default bridge network 是啟動 docker 時就會產生的 network 之一，名稱為 `bridge`，

在啟動容器時沒有特別指定 network，都會預設使用 default bridge network 配置。

透過 `docker inspect` 查看 `bridge` 的設定，`bridge` 預設使用 `172.17.0.0/16` 網段，

使用 default bridge 的 container 內部的虛擬網路卡（eth0）會分配到網段中其中一個未被使用的 ip。

```shell
$ docker inspect bridge
[
    {
        "Name": "bridge",
        "Id": "67fc3098224b578e2687ee1372ab04ed4c89cc87788a5b1fb198eae767766424",
        "Created": "2022-05-12T01:23:18.260789834Z",
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
        // ...
    }
]
```

### common connect

所有使用 default bridge 的 container 都可以透過彼此得 ip 位址互相溝通，

也可以透過 `--link` 指令來建立 hostname。

`--link {container name}` 會將被連結的 container_name 和 hostname 都寫到被啟動的 container 的 `/etc/hosts` 內

```shell
# 建立 ubuntu_1 和 ubuntu_2 container，其中 ubuntu_2 設定 --link ubuntu_1
$ docker run -itd \
    --name ubuntu_1 \
    --hostname ubuntu_1.ubuntu.com \
     ubuntu

$ docker run -itd \
    --name ubuntu_2 \
    --hostname ubuntu_2.ubuntu.com \
    --link ubuntu_1 \
    ubuntu

# 使用 docker inspect 可以看到兩個 container 都是使用 default bridge network
$ docker inspect bridge
[
    {
        "Name": "bridge",
        "Id": "67fc3098224b578e2687ee1372ab04ed4c89cc87788a5b1fb198eae767766424",
        "Created": "2022-05-12T01:23:18.260789834Z",
        "Scope": "local",
        "Driver": "bridge",

        //..

        "Containers": {
            "71be1bc0d8b8f32339de5bb723b9985b7842cd638ba000090372bbfd1bf4d908": {
                "Name": "ubuntu_2",
                "EndpointID": "c95c89294fc3f2640fda551766831185529d23aee6d35d0681bf7d0244e6e974",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16", <=== ubuntu_2
                "IPv6Address": ""
            },
            "7763111acfbda138138fa4f3bc495bf5d43da6f2d4eebb7f540a1aca39bbce5f": {
                "Name": "ubuntu_1",
                "EndpointID": "72ef265089774e2d3c02793c909cf9e434214cfbb38ac0716a16b28d4e4d693a",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16", <=== ubuntu_1
                "IPv6Address": ""
            }
        },

        //..
    }
]
```

確認 ubuntu_1 container 的行為

```shell
# 進入 ubuntu_1 
$ docker exec -it ubuntu_1 bash

# 安裝 ping
root@ubuntu_1:/$ apt-get update && apt-get install -y iputils-ping

# 查看 /etc/hosts，發現沒有 ubuntu_2 的資料，因為 --link 指令是單向設定的
root@ubuntu_1:/$ cat /etc/hosts
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2 ubuntu_1.ubuntu.com ubuntu_1

# 在 default bridge network 內的容器可以透過 ip 互相溝通
root@ubuntu_1:/$ ping 172.17.0.3
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.200 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.312 ms

# 無法透過 ubuntu_2 的 hostname 進行溝通
root@ubuntu_1:/$ ping ubuntu_2.ubuntu.com
ping: ubuntu_2.ubuntu.com: Name or service not known
```

確認 ubuntu_2 container 的行為

```shell
# 進入 ubuntu_2
$ docker exec -it ubuntu_2 bash 

# 安裝 ping
root@ubuntu_2:/$ apt-get update && apt-get install -y iputils-ping

# 查看 /etc/hosts，可以看到 --link 幫忙設定了 ubuntu_1 的 hostname
root@ubuntu_2:/$ cat /etc/hosts
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2 ubuntu_1 ubuntu_1.ubuntu.com
172.17.0.3 ubuntu_2.ubuntu.com ubuntu_2
```

### DNS

docker container 在 default bridge network 下的 DNS 設定會與主機的相同。

也可以透過指令 `--dns` 來進行相關設定。

```shell
$ docker run -it --rm --dns "8.8.8.8" --name ubuntu_10 ubuntu

root@7246b29814ea:/$ cat /etc/resolv.conf
nameserver 8.8.8.8
```

**note：**  

mac 上無法將 host 的 DNS Server 寫到 container 上，需要再另外確認狀況

## custom bridge network

官方推薦使用 custom bridge network，有兩種方式可以啟用；

1. 使用者自行建立 custom bridge network 來使用
2. docker-compose 預設採用 custom bridge network

與 default bridge network 不同的是，在相同 bridge network 內，

所有容器皆可可透過 hostname 來進行溝通。

### common connect

[docs.docker/network-tutorial-standalone/use-user-defined](https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks)
[docs.docker/commandline/network_create](https://docs.docker.com/engine/reference/commandline/network_create/)

可以透過 `docker network create` 指令來產生 custom bridge network。

```shell
# docker network create [OPTIONS] {network_name}
$ docker network create --driver bridge tommylin_network_1

$ docker network ls
NETWORK ID     NAME                 DRIVER    SCOPE
67fc3098224b   bridge               bridge    local
6b6200a452a4   host                 host      local
1f931a78dc58   none                 null      local
6a779b6aa701   tommylin_network_1   bridge    local
```

設定上可以進行多種客製化

```shell
$ docker network create \
    -d bridge \
    --subnet "172.99.0.0/16" \
    --gateway "172.99.0.254" \
    tommylin_network_2

419635b2789e93342769629c99ba2a2b563412a735d4d427b35e566ca3bb37a5

$ docker inspect tommylin_network_2
[
    {
        "Name": "tommylin_network_2",
        "Id": "419635b2789e93342769629c99ba2a2b563412a735d4d427b35e566ca3bb37a5",
        "Created": "2022-05-19T01:55:52.895488922Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.99.0.0/16",
                    "Gateway": "172.99.0.254"
                }
            ]
        },

        // ..
    }
]
```

* `-d --driver`：設定 network 的驅動類型，預設為 bridge

接著使用 `docker run --network {network_name}` 指令，

在啟動 container 時進行指定要使用的 network。

也可以透過 `docker network connect {network_name} {container_name}` 指令，

改變已經啟動的 container 使用的 network。

```shell
# 啟動 container
$ docker run --network tommylin_network_1 -itd ubuntu_3 ubuntu

# 將 ubuntu_1 ubuntu_2 改為 custom bridge network
$ docker network connect tommylin_network_1 ubuntu_1
$ docker network connect tommylin_network_1 ubuntu_2

$ docker inspect tommylin_network_1
[
    {
        "Name": "tommylin_network_1",
        // ...
        "Containers": {
            "4826c335e4c4fe4651b4c63877e5f642e5fb3bf1710ad84a0a09649dc61f3697": {
                "Name": "ubuntu_2",
                "EndpointID": "f43f0c6a1ede5888ea6c353a28246d724e6cef962b16e9fd0a559259b81dc0d6",
                "MacAddress": "02:42:ac:15:00:04",
                "IPv4Address": "172.21.0.4/16",
                "IPv6Address": ""
            },
            "9025cf8a4a712bbdde04ea590790d597761ed7004d0d54d5206b7103b75aa700": {
                "Name": "ubuntu_1",
                "EndpointID": "b96bb94ca46c1443d76ec479eec2957820552d109c496ed6eedd6b6d942e708c",
                "MacAddress": "02:42:ac:15:00:03",
                "IPv4Address": "172.21.0.3/16",
                "IPv6Address": ""
            },
            "f87d476b05da0bface270d74d38e60abddbe1269af59342ecdc4c4384c4a741c": {
                "Name": "ubuntu_3",
                "EndpointID": "9e6fdfd511b1841295973d30c93dff8c51a0ae31567b8ad20a4362c918e72ff0",
                "MacAddress": "02:42:ac:15:00:02",
                "IPv4Address": "172.21.0.2/16",
                "IPv6Address": ""
            }
        },
        // ...
    }
]
```

檢查 ubuntu_3 container 的行為

```shell
# 進入 ubuntu_3
$ docker exec -it ubuntu_3 bash

# 安裝 ping
root@f87d476b05da:/$ apt-get update && apt-get install -y iputils-ping

# 查看 /etc/hosts，沒有什麼額外的設定
root@f87d476b05da:/$ cat /etc/hosts
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.21.0.2 f87d476b05da

# 可以 ping 到 ubuntu_2 是因為 custom bridge network 透過自訂的 DNS Server 進行轉發
root@f87d476b05da:/$ ping ubuntu_2
PING ubuntu_2 (172.21.0.4) 56(84) bytes of data.
64 bytes from ubuntu_2.tommylin_network_1 (172.21.0.4): icmp_seq=1 ttl=64 time=1.45 ms
64 bytes from ubuntu_2.tommylin_network_1 (172.21.0.4): icmp_seq=2 ttl=64 time=0.227 ms

root@f87d476b05da:/$ ping ubuntu_2.ubuntu.com
PING ubuntu_2.ubuntu.com (172.21.0.4) 56(84) bytes of data.
64 bytes from ubuntu_2.tommylin_network_1 (172.21.0.4): icmp_seq=1 ttl=64 time=0.485 ms
64 bytes from ubuntu_2.tommylin_network_1 (172.21.0.4): icmp_seq=2 ttl=64 time=0.317 ms

```

### docker compose connection

使用 docker-compose 的情況下，預設會自動產生 custom bridge network，

並且將同一個 docker-compose.yml 內的所有 container 放入同一個 custom bridge network。  

network 建立的規則為 `{root_directory_name}_{network_name}`

**test/docker-compose.yml ：**

```yaml
version: "3"

services:
  ubuntu_1:
    image: ubuntu
    command: ["sleep","infinity"]
  ubuntu_2:
    image: ubuntu
    command: ["sleep","infinity"]
  ubuntu_3:
    image: ubuntu
    command: ["sleep","infinity"]
```

檢查 network

```shell
$ docker-compose up -d

# 無設定的 network name 叫做 default
$ docker network ls
NETWORK ID     NAME                 DRIVER    SCOPE
67fc3098224b   bridge               bridge    local
6b6200a452a4   host                 host      local
1f931a78dc58   none                 null      local
6655397e6617   test_default         bridge    local
```

幫 test/docker-compose.yml 加上 network 的設定

```yaml
version: "3"

services:
  ubuntu_1:
    image: ubuntu
    command: ["sleep","infinity"]
    networks:
      - ubuntu_network
  ubuntu_2:
    image: ubuntu
    command: ["sleep","infinity"]
    networks:
      - ubuntu_network
  ubuntu_3:
    image: ubuntu
    command: ["sleep","infinity"]
    networks:
      - ubuntu_network

networks:
  ubuntu_network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.88.0.0/16
          gateway: 172.88.0.1
```

### DNS

custom bridge network 無法自行設定 DNS，因為已經預設上會使用 Embedded DNS Server（127.0.0.11）

**note：**

理論上應該可以對 Embedded DNS Server 額外註冊 external DNS，未來再來研究。
