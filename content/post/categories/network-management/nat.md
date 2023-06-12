---
title: "NAT"
date: 2023-06-12T09:30:00+08:00
description: "Network Address Translator 的簡易介紹"
tags: [network]
categories: [Network Management]
---

## Description

NAT（Network Address Translator）是為了解決 IPv4 不夠用所延伸出的應用方式，

NAT 服務會建立一張對應表，將同一組 Public IP 的不同 port number 分配給內部網路的所有 Private IP，

當持有 Private IP 的內部網路端發出訪問 Internet 的請求時，該請求就會透過 NAT 服務將請求來源的 Private IP 轉為對應的 Public IP + prot number 並接續進行請求。

## NAT 分類

NAT 主要分為三類

- 靜態 NAT
- 動態 NAT
- PAT（Port Address Translation）

### 靜態 NAT

將 Private IP 位址對應到一個固定的 Public IP 位址。這種形式的 NAT 經常被用在需要與外部網路進行固定通信的服務上。

### 動態 NAT

將 Private IP 位址對應到一個 Public IP 位址池中的某個位址。當 Private IP 位址需要與外部網路通信時，NAT 設備從 Public IP 位址池中選擇一個尚未使用的 IP 位址進行對應。

### PAT（Port Address Translation）

也被稱為 NAPT (Network Address Port Translation) 或者是 NAT Overload，這是最常見的形式。

PAT 不僅僅轉換 IP 位址，還會轉換 TCP 或 UDP 的 port number；

這意味著多個 Private IP 位址可以對應到同一個 Public IP 位址，只是端口號不同。

## 範例

### step.1 請求發起

當內部設備（家庭或公司網路內的使用者）想要訪問外部網路時它會創建一個包含來源 IP 地址（設備的 Private IP）和 port number 的封包。

### step.2 轉換

這個封包到達執行 PAT 的設備（路由器或防火牆）時，PAT 設備會把封包的來源 IP 地址更改為 Public IP 地址，並同步更改 port number，

新的 port number 是由 PAT 設備從可用端口池中選擇的；PAT 設備會在 NAT 表中新增一筆資料，記錄這個 Private IP + port number 對應到的 Public IP + port number。

### step.3 請求傳遞

PAT 設備會將修改後的數據包發送到互聯網。

### step.4 響應返回

當響應返回到達 PAT 設備時，PAT 設備會查找 NAT 表，找到與這個 Public IP + port number 相對應的 Private IP + prot number。

並將響應封包的目標 IP 地址和目標端口號更改為這個內部 IP 地址和端口號，最後發送封包給該內部設備。
