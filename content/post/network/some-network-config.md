---
title: "hosts + resolv 紀錄"
date: 2022-05-20T14:38:44+08:00
description: "hosts + resolv 的介紹"
tags: [network]
---

## 前言

介紹一下 `/etc/hosts` 和 `/etc/resolv.conf` 兩個設定檔，順便做個紀錄

## /etc/hosts

### 敘述

搜尋 domain name 時，會先來 `/etc/hosts` 檢查，如果有找到 domain name 對應的 IP 就會直接使用，不會再去訪問 DNS Server。

### 測試

因為會優先檢查，所以如果在表單寫了 `127.0.0.1 www.kkbox.com` 的 mapping，本機端就永遠無法連接到真實的 kkbox.com 了，而是會先被導到 `127.0.0.1`

```shell
# 測試 ping www.kkbox.com 得到 ip 位址為 210.61.183.32
$ ping www.kkbox.com

PING 15169.www.kkcube.com (210.61.183.32) 56(84) bytes of data.
64 bytes from ip-210-61-183-32.kkcube.com (210.61.183.32): icmp_seq=1 ttl=37 time=35.3 ms
64 bytes from ip-210-61-183-32.kkcube.com (210.61.183.32): icmp_seq=2 ttl=37 time=35.6 ms

# 讓 www.kkbox.com mapping 到 127.0.0.1
$ echo "127.0.0.1 www.kkbox.com" >> /etc/hosts
$ ping www.kkbox.com

PING www.kkbox.com (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.219 ms
```

## /etc/resolv.conf

[Ubuntu manpages](https://manpages.ubuntu.com/manpages/xenial/en/man5/resolv.conf.5.html)

### 敘述

搜尋 domain name 時會先來這邊查看設定的 DNS 網站有哪些，在透過這些 DNS 網站進行解析

### 參數介紹

* nameserver
指定解析 domain name 時需要使用的 DNS
* search
如果輸入的 domain name 不完整的時候，會補上 search 的內容再次搜尋
* domain
作為 search 的 default 內容，如果設定 search 後就沒用了

### 測試

```shell
# news 為不合法的 hostname
$ ping news
ping: news: No address associated with hostname

# 在 resolv.conf 新增 search 條件
$ echo "search google.com" >> /etc/resolv.conf

# hostname 不完整時，會透過 search 自動補齊
$ ping news
PING news.google.com (142.251.43.14) 56(84) bytes of data.
64 bytes from tsa03s08-in-f14.1e100.net (142.251.43.14): icmp_seq=1 ttl=37 time=8.76 ms
64 bytes from tsa03s08-in-f14.1e100.net (142.251.43.14): icmp_seq=2 ttl=37 time=9.95 ms
```

### 注意事項

* mac 上修改 `/etc/resolv.conf` 後並不會有任何改變，原因待查證
