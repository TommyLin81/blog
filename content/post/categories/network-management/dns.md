---
title: "DNS 入門"
date: 2023-06-12T09:47:26+08:00
description: "Domain Name Server 的入門介紹"
tags: [network]
categories: [Network Management]
---

## Description

一般在瀏覽器上輸入的 url（`https://google.com`）都是由英文組成，以人類方便記憶的形式呈現；

但是實際上封包傳輸時，需要知道的是 Public IP 位址。

本章節要介紹的 DNS 就是負責將 domain name（`google.com`）轉換成對應 IP Address 的服務。

## Chat Flow

![curl-transfer](/blog/images/dns.png)
[picture resource](https://www.cloudflare.com/zh-tw/learning/dns/what-is-dns/)

0. 在瀏覽器輸入 [`https://google.com`](http://google.com) 後，Linux 系統會先去 `/etc/hosts` 文件查看是否有對應的 IP 位址。
1. 如果 `/etc/hosts` 沒有找到對應的結果，瀏覽器會對設定好的 DNS Server 發出 DNS protocol request，該 DNS Server 通常為 Recursive Resolver Server，會一直協助這次的 DNS 解析直到取得結果。
2. Resolver Server 的 cache 如果沒有資料，就會去詢問 Root Server [`google.com`](http://google.com) 的資訊
3. Root Server 解析到 `.com` 這段資訊後，會回傳有紀錄 `.com` 所有資訊的 TLD Server 的 IP 位址
4. Resolver Server 馬上問 TLD Server [`google.com`](http://google.com) 的資訊
5. TLD 解析解析完 `google` 這段，並將存有 [`google.com`](http://google.com) 資訊的 Name Server 的 IP 位址
6. Resolver Server 馬上問 Name Server [`google.com`](http://google.com) 的資訊
7. Name Server 會根據內部儲存的資訊回覆不同的結果，可能是 [`google.com`](http://google.com) 的 IP 位址，也可能是另一個需要轉址的 domain，這邊假設 Name Server 儲存的是 type 為 A 的 IP 位址資訊 `142.251.42.238`
8. Resolver Server 將 `142.251.42.238` 回傳給瀏覽器
9. 瀏覽器發出 [`https://142.251.42.238`](http://142.251.42.238) 的 http 請求
10. google 的 Server 回應網站畫面給瀏覽器

## Reference

- <https://www.cloudflare.com/zh-tw/learning/dns/what-is-dns>
- <https://www.youtube.com/watch?v=HnUDtycXSNE>
- <https://www.rfc-editor.org/rfc/rfc1034.txt>
