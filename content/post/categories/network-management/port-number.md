---
title: "Port Number"
date: 2024-01-04T01:15:00+08:00
description: "Port Number 簡介"
tags: [network]
categories: [Network Management]
---

## Description

IP address 和 Port Number 通常會一起出現，兩者結合起來可以唯一標示一個網路上的服務，例如「某台主機上的某個應用程式正在提供的服務」。

### IP Address 與 Port Number 結合的格式範例

```text
# <ip address>:<port number>
127.0.0.1:8080
```

IP address 用來標示在 internet 上，某一個裝置（如電腦、手機）的網路介面。  
Port number 則是用來進一步區分該裝置上所提供的特定服務（例如 Web server、SSH 伺服器等）。

## Port Number 定義

Port number 是一個 16-bit 的整數，範圍為 0~65535。  
雖然任何程式理論上都可以使用任意 port，但大多數常見 port 的用途由 IANA（Internet Assigned Numbers Authority）來定義和規範。

IANA 的定義中，port number 可以分為三種：

- Well Known Ports
- Registered Ports
- Dynamic and/or Private Ports

### Well Known Ports

範圍為 0 ～ 1023

這些 port number 通常保留給系統進程或需要較高權限（如 root）才能啟動的伺服器服務。許多常見協定（如 HTTP、SSH）會預設使用這些 port。

以下是一些常見的 Well Known Port 範例：

| Port number | Application |
| ----------- | ----------- |
| `22`        | ssh         |
| `80`        | http        |
| `443`       | https       |

### Registered Ports

範圍為 1024 ～ 49151

這些 port 雖然不是專屬於特定應用程式，但許多應用會向 IANA 登記使用。

以下是一些常見的 Registered Port 範例：

| Port number | Application |
| ----------- | ----------- |
| `3306`      | MySQL       |
| `5432`      | PostgreSQL  |

### Dynamic or Private Ports

範圍為 49152 ～ 65535

這些 port number 主要用於臨時的網路連線，或者是由程序自動選擇使用。

## 如何查看目前本機有哪些 Port 被使用？

在 Linux 或 macOS 中，可以使用以下指令查看哪些 port 正在被程式佔用：

```bash
lsof -i -P -n | grep LISTEN
```

## Postscript

port number 雖然有 IANA 的規範，並不是真的硬性規定的東西，但是實際應用上，照著 IANA 的規範走可以減少讓其他人混亂的情況發生。

像是多人在同一台開發機上進行開發，起 container 所 mapping 的 port number 就應該避免佔用到一些熱門服務的 port，也可以觀察到 chrome 在連接 https protocol 時，會預設使用 443 port number 進行連線，這些都是照著 IANA 的規範走的。

## Reference

- [iana - port number registry table](<https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml>)
