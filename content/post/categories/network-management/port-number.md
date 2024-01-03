---
title: "Port Number"
date: 2024-01-04T01:15:00+08:00
description: "Port Number 簡介"
tags: [network]
categories: [Network Management]
---

## Description

IP address 和 Port Number 通常會一起出現，兩個結合起來表示一個特定的網路服務。

***format:***

```text
# <ip address>:<prot number>
127.0.0.1:8080
```

IP address 指在 internet 上的某個網路介面（某台電腦、手機）的獨特地址

port number 則近一步表示該網路介面所提供的服務是什麼

## Port Number 定義

port number 是一個 16-bit 的數字，範圍為 0 ～ 65535

大部分的 port number 都是由 IANA（Internet Assigned Numbers Authority）所定義用途

IANA 的定義中，port number 可以分為三種：

- Well Known Ports
- Registered Ports
- Dynamic and/or Private Ports

### Well Known Ports

範圍為 0 ～ 1023

這些 port number 這些埠通常被系統進程或者由特權使用者運行的進程所使用。

example:

| Port number | Application |
| ----------- | ----------- |
| `22`        | ssh         |
| `80`        | http        |
| `443`       | https       |

### Registered Ports

範圍為 1024 ～ 49151

這些 port number 主要由軟體應用程式使用

雖然並不是永久性地分配給特定的程式，但在使用期間會由 IANA 進行管理和協調

example:

| Port number | Application |
| ----------- | ----------- |
| `3306`      | MySQL       |
| `5432`      | PostgreSQL  |

### Dynamic or Private Ports

範圍為 49152 ～ 65535

這些 port number 主要用於臨時的網路連線，或者是由程序自動選擇使用

當一個程式需要與另一個程式進行通信，並且不需要使用特定的埠號時，它們通常會選擇使用這些埠

## Postscript

port number 雖然有 IANA 的規範，並不是真的硬性規定的東西

不過在實際應用上，照著 IANA 的規範走可以減少讓其他人混亂的情況發生

像是多人在同一台開發機上進行開發，起 container 所 mapping 的 port number 就應該避免佔用到一些熱門服務的 port

也可以觀察到 chrome 在連接 https protocol 時，會預設使用 443 port number 進行連線，這些都是照著 IANA 的規範走的

## Reference

- [iana - port number registry table](<https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml>)
