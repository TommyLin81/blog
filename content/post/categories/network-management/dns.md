---
title: "DNS 入門"
date: 2023-06-12T09:47:26+08:00
description: "Domain Name Server 的入門介紹"
tags: [network]
categories: [Network Management]
---

## Overview

DNS（Domain Name System）是負責將網址（domain name）轉換成 IP 位址的系統。
當使用者輸入 `google.com` 時，DNS 會查詢對應的 IP，讓瀏覽器能與正確的伺服器建立連線。

## DNS 查詢流程（DNS Query Flow）

![curl-transfer](/images/dns.png)
[picture resource](https://www.cloudflare.com/zh-tw/learning/dns/what-is-dns/)

### 本機查詢（Local Resolution）

0. 使用者在瀏覽器輸入 `https://google.com`，系統首先查詢 `/etc/hosts` 檔案中是否有對應紀錄
1. 若無命中則發送 DNS 查詢給 DNS Server（通常是 ISP 提供的）

### 遞迴查詢（Recursive Resolution）

2. DNS Server（Recursive Resolver）若無快取資料，會向 Root Server 詢問 `.com`
3. Root Server 回覆 TLD Server 的位址
4. Resolver 再向 TLD Server 詢問 `google.com`
5. TLD Server 回覆 Name Server 的位址
6. Resolver 向該 Name Server 請求 `google.com` 的記錄

### 回傳結果與載入網頁（Resolution Result）

7. Name Server 回傳 A 記錄（如 `142.251.42.238`）
8. Resolver 將該 IP 傳給瀏覽器
9. 瀏覽器對該 IP 發出 HTTP 請求
10. 顯示網站內容

## 常見 DNS 記錄類型（DNS Record Types）

| 記錄類型 | 說明                     |
| -------- | ------------------------ |
| A        | IPv4 位址                |
| AAAA     | IPv6 位址                |
| CNAME    | 別名指向另一個域名       |
| MX       | 郵件交換伺服器           |
| TXT      | 任意文字，常用於驗證用途 |
| NS       | Name Server 的指定紀錄   |

## References

- [What is DNS? | How DNS works | Cloudflare](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [DNS Records Explained](https://www.youtube.com/watch?v=HnUDtycXSNE)
[rfc-editor.org/rfc/rfc1034.txt](https://www.rfc-editor.org/rfc/rfc1034.txt)

