---
title: "DNS 介紹"
date: 2025-07-28T18:42:24+08:00
description: "說明網址如何被 DNS 轉換為對應的 IP 位址，並使用 dig 工具來查詢 DNS 和排除 DNS 問題！"
tags: [network]
categories: [Network Management]
---

## Introduction

DNS（Domain Name System）是一個將網址轉換成 IP 位址的系統；就像是一本電話簿的功能，使用者只需要記住對象的名字（網址），而不用記住相對複雜的電話號碼（IP）。

當使用者想要存取 `google.com` 這個網址時，系統會透過一系列的 DNS 查詢，最終找到網址對應的 IP，有了 IP 就可以將封包發送到正確的主機上了！

## DNS Lookup

![curl-transfer](/images/dns.png)
[image source](https://www.cloudflare.com/learning/dns/what-is-dns/)

### Local Resolution

0. 使用者在瀏覽器輸入 `https://google.com`，程式會呼叫 DNS Client 開始解析網址，先從本地端檢查快取、檢查 `/etc/hosts` 設定、檢查是否為 multicast DNS
1. 若皆無對應的 IP，則向外部 DNS Resolver 發送 DNS 查詢請求，DNS Resolver 的選擇與當前連線的網路狀態有關，macOS 環境可以透過 `scutil --dns` 查看

### Recursive Resolution

2. DNS Resolver 若無快取資料，則開始迭代查詢；首先會從全球 13 個 Root Nameserver IP 選出一個，並向其詢問 `.com` 資訊存放於哪些 TLD Nameserver
3. Root Nameserver 回覆 TLD Nameserver 的位址
4. DNS Resolver 再向其中一台 TLD Nameserver 詢問 `google.com` 的 Authoritive Server
5. TLD Nameserver 回覆持有 `google.com` 所有 DNS 記錄（DNS record）的 Authoritive Server 位址
6. DNS Resolver 再向其中一台 Authoritive Server 取得所需的 DNS 記錄

### Resolution Result

7. Authoritive Server 回傳 A 記錄（A record）上的 IP 資訊 `142.250.77.14`
8. DNS Resolver 將該 IP 傳給瀏覽器
9. 瀏覽器對該 IP 發出 HTTP 請求
10. 取得網頁內容

## Common DNS Record Types

| Type  | Description                                                                                                                                                               |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A     | IPv4 位址                                                                                                                                                                 |
| AAAA  | IPv6 位址                                                                                                                                                                 |
| CNAME | 指向另一個網址                                                                                                                                                            |
| MX    | 指向電子信箱的網址，其中指向的網址必須要為 A 記錄或是 AAAA 記錄，否則失效                                                                                                 |
| TXT   | 任意文字，常用於驗證用途                                                                                                                                                  |
| SOA   | 權威記錄起始（Start of Authority），紀錄當前 DNS 區域（DNS zone）的主要名稱伺服器（Primary Nameserver）、管理者信箱、DNS 記錄的最新版本，作為名稱伺服器之間同步資訊的依據 |
| NS    | 名稱伺服器（Nameserver），通常會有多組                                                                                                                                    |
| SRV   | 服務記錄，將服務名稱、協議組合而成的網址指向另一個指定埠號的網址                                                                                                          |
| PTR   | 指標紀錄（Pointer），反向記錄 IPv4 或 IPv6 指向的網址，通常用於垃圾郵件檢查                                                                                               |

## DNS Cache and TTL

### Cache

DNS 查詢結果會被快取在多個層級當中

- 瀏覽器快取：瀏覽器內建的 DNS 快取
- 作業系統快取：OS 層級的 DNS 快取
- DNS Resolver 快取：ISP 或公共 DNS 服務的快取
- 各級名稱伺服器快取：Root、TLD、Authoritative Server 的快取

### TTL（Time To Live）

每個 DNS 記錄都可以設定 TTL，決定這個設定在快取中保留的時間，需要依照不同服務的屬性和需求去權衡 TTL 的長度

- 短 TTL 適用於需要頻繁更新的 DNS 記錄，但是會增加查詢的次數，提高負載的上限
- 長 TTL 適用於穩定不變的 DNS 記錄，但是在記錄需要進行改變時，會花費較長的時間等待快取過期

## dig - A DNS Troubleshooting Tool

dig 可以快速進行 DNS 查詢結果的驗證，以及實現 DNS 迭代查詢的過程

### Commen Commands

- 查詢指定網址的 A 記錄

```bash
dig google.com
```

- 查詢指定網址的 A 記錄，顯示最簡短資訊

```bash
dig google.com +short
```

- 查詢指定網址的所有 DNS 記錄

```bash
dig google.com ANY
```

- 使用指定的 DNS Resolver 進行查詢

```bash
dig @8.8.8.8 google.com
```

- 使用 dig 進行迭代查詢，並列出查詢的過程和結果

```bash
dig google.com +trace
```

### Tracing of Delegation Path

dig 可以透過指令模擬 DNS Resolver 進行迭代查詢，並將查詢過程列出來

接著我們就使用以下指令來分解整個迭代查詢的過程

```bash
dig @1.1.1.1 google.com +trace
```

返回的結果如下，接下來讓我們一個步驟一個步驟分開來看

```bash
; <<>> DiG 9.10.6 <<>> @1.1.1.1 google.com +trace
; (1 server found)
;; global options: +cmd
.			511807	IN	NS	a.root-servers.net.
.			511807	IN	NS	b.root-servers.net.
.			511807	IN	NS	c.root-servers.net.
.			511807	IN	NS	d.root-servers.net.
.			511807	IN	NS	e.root-servers.net.
.			511807	IN	NS	f.root-servers.net.
.			511807	IN	NS	g.root-servers.net.
.			511807	IN	NS	h.root-servers.net.
.			511807	IN	NS	i.root-servers.net.
.			511807	IN	NS	j.root-servers.net.
.			511807	IN	NS	k.root-servers.net.
.			511807	IN	NS	l.root-servers.net.
.			511807	IN	NS	m.root-servers.net.
.			511807	IN	RRSIG	NS 8 0 518400 20250810050000 20250728040000 46441 . GMmmL9anoO58q0ZiFYjwpnN/kKHeUa2WyieU7mf+STO0TfKFJn5AAy8V YTh5+SjpZTx84d7+BEfncw9QRJDWxQ1sDiR0zj4rc38iRBvObHadbohG mMTtb56MPktffKKy6gBu2cMIHMIM50NwjX8sOSl6R2XH4/OTGe4304lb aSllI0a9RE2GCl84mtzkoLNHMyHCfgmFHxxe3aOZ1R1yJoaqPllFWJwO XX82UxHtjuUjG6zdm9d/v1rqsljQwiZdkufJmwvgGzXnVNSGy2R5y8F/ MljK2zEyJqlYuevi6b+GUAsqihw8cKKXYmFsyCmlmB4Aed5dbTpDgVr9 1BFryg==
;; Received 525 bytes from 1.1.1.1#53(1.1.1.1) in 30 ms

com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	19718 13 2 8ACBB0CD28F41250A80A491389424D341522D946B0DA0C0291F2D3D7 71D7805A
com.			86400	IN	RRSIG	DS 8 1 86400 20250810050000 20250728040000 46441 . aIoQnSkjY/gg+HYZHvmmc89Y7Guf4h8IASBu+/4NFs4GvPU2tIUm83ag YF9iyWiC7PeMmnlWcNQZAXdj4AYMA8HbqWWHpmYjivexwRVl97fR5ixi zueanQb6UnhCoZGwSFJBck5bz+rwkerkFvQWhTT8e1K1JroUayMY+9P/ zEM9U4ABa9jGpTQVxYRVYZgyrYcZZBeTLzQajCq+GMZFkQZ5me73yu1/ RP61msu3SDkcnvHjo2MYnW8teFTd1KkWDEKFWtci89sX6wpak7kGmz0P OgP/jPkB9QtEIW6EVoxnqbU9ccJVoho858fxxg1rGi6OlxpM0lIva2FD QuvuSw==
;; Received 1170 bytes from 170.247.170.2#53(b.root-servers.net) in 200 ms

google.com.		172800	IN	NS	ns2.google.com.
google.com.		172800	IN	NS	ns1.google.com.
google.com.		172800	IN	NS	ns3.google.com.
google.com.		172800	IN	NS	ns4.google.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN NSEC3 1 1 0 - CK0Q3UDG8CEKKAE7RUKPGCT1DVSSH8LL  NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN RRSIG NSEC3 13 2 900 20250803022222 20250727011222 20545 com. pKSGB6htdnXf5cs5mVnNLTjSDXYnU9z+UqLLYiI8h1BJ3Z6h7FG0/Fp7 oMt6lu4iETmSJHAViT8hZ/DGruNg0g==
S84BOR4DK28HNHPLC218O483VOOOD5D8.com. 900 IN NSEC3 1 1 0 - S84BR9CIB2A20L3ETR1M2415ENPP99L8  NS DS RRSIG
S84BOR4DK28HNHPLC218O483VOOOD5D8.com. 900 IN RRSIG NSEC3 13 2 900 20250804012038 20250728001038 20545 com. fnIjm2HNoTq7wtFOrglxDYQZFBFxGoTIdQya88ryzO4wI4gn+p36M71D fGOsZoaVd+ntw/K7530z3v4Uubwt2Q==
;; Received 644 bytes from 2001:503:83eb::30#53(c.gtld-servers.net) in 79 ms

google.com.		300	IN	A	142.250.198.78
;; Received 55 bytes from 216.239.32.10#53(ns1.google.com) in 65 ms
```

第一步可以看到 dig 向我們設定的 Cloudflare DNS Resolver（`1.1.1.1`）索取 Root Nameserver IP 列表

Root Nameserver IP 固定有 13 個 IP，一般的 DNS Resolver 都會將這 13 個 IP 內建在服務中，所以可以看到 [DNS Lookup](#dns-lookup) 的流程並未有這段過程

```bash
.			511807	IN	NS	a.root-servers.net.
.			511807	IN	NS	b.root-servers.net.
.			511807	IN	NS	c.root-servers.net.
.			511807	IN	NS	d.root-servers.net.
.			511807	IN	NS	e.root-servers.net.
.			511807	IN	NS	f.root-servers.net.
.			511807	IN	NS	g.root-servers.net.
.			511807	IN	NS	h.root-servers.net.
.			511807	IN	NS	i.root-servers.net.
.			511807	IN	NS	j.root-servers.net.
.			511807	IN	NS	k.root-servers.net.
.			511807	IN	NS	l.root-servers.net.
.			511807	IN	NS	m.root-servers.net.
.			511807	IN	RRSIG	NS 8 0 518400 20250810050000 20250728040000 46441 . GMmmL9anoO58q0ZiFYjwpnN/kKHeUa2WyieU7mf+STO0TfKFJn5AAy8V YTh5+SjpZTx84d7+BEfncw9QRJDWxQ1sDiR0zj4rc38iRBvObHadbohG mMTtb56MPktffKKy6gBu2cMIHMIM50NwjX8sOSl6R2XH4/OTGe4304lb aSllI0a9RE2GCl84mtzkoLNHMyHCfgmFHxxe3aOZ1R1yJoaqPllFWJwO XX82UxHtjuUjG6zdm9d/v1rqsljQwiZdkufJmwvgGzXnVNSGy2R5y8F/ MljK2zEyJqlYuevi6b+GUAsqihw8cKKXYmFsyCmlmB4Aed5dbTpDgVr9 1BFryg==
;; Received 525 bytes from 1.1.1.1#53(1.1.1.1) in 30 ms
```

第二步對應到 [DNS Lookup](#dns-lookup) 的 `2.`、`3.` 步，可以看到 dig 從 13 組 Root Nameserver 中挑選了 `b.root-servers.net` 進行頂網域（top-level domain）`com.` 的查詢，並且取得了 13 組含有相關資訊的 TDL Nameserver

```bash
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	19718 13 2 8ACBB0CD28F41250A80A491389424D341522D946B0DA0C0291F2D3D7 71D7805A
com.			86400	IN	RRSIG	DS 8 1 86400 20250810050000 20250728040000 46441 . aIoQnSkjY/gg+HYZHvmmc89Y7Guf4h8IASBu+/4NFs4GvPU2tIUm83ag YF9iyWiC7PeMmnlWcNQZAXdj4AYMA8HbqWWHpmYjivexwRVl97fR5ixi zueanQb6UnhCoZGwSFJBck5bz+rwkerkFvQWhTT8e1K1JroUayMY+9P/ zEM9U4ABa9jGpTQVxYRVYZgyrYcZZBeTLzQajCq+GMZFkQZ5me73yu1/ RP61msu3SDkcnvHjo2MYnW8teFTd1KkWDEKFWtci89sX6wpak7kGmz0P OgP/jPkB9QtEIW6EVoxnqbU9ccJVoho858fxxg1rGi6OlxpM0lIva2FD QuvuSw==
;; Received 1170 bytes from 170.247.170.2#53(b.root-servers.net) in 200 ms
```

第三步對應到 [DNS Lookup](#dns-lookup) 的 `4.`、`5.` 步，可以看到 dig 從 13 組 TLD Nameserver 中挑選了 `c.gtld-servers.net` 進行 `google.com.` 的查詢，並且取得了 4 組名稱伺服器；因為這些名稱伺服器都包含了 `google.com.` 的所有 DNS 記錄，所以我們也稱其為 `google.com.` 的權威伺服器

```bash
google.com.		172800	IN	NS	ns2.google.com.
google.com.		172800	IN	NS	ns1.google.com.
google.com.		172800	IN	NS	ns3.google.com.
google.com.		172800	IN	NS	ns4.google.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN NSEC3 1 1 0 - CK0Q3UDG8CEKKAE7RUKPGCT1DVSSH8LL  NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN RRSIG NSEC3 13 2 900 20250803022222 20250727011222 20545 com. pKSGB6htdnXf5cs5mVnNLTjSDXYnU9z+UqLLYiI8h1BJ3Z6h7FG0/Fp7 oMt6lu4iETmSJHAViT8hZ/DGruNg0g==
S84BOR4DK28HNHPLC218O483VOOOD5D8.com. 900 IN NSEC3 1 1 0 - S84BR9CIB2A20L3ETR1M2415ENPP99L8  NS DS RRSIG
S84BOR4DK28HNHPLC218O483VOOOD5D8.com. 900 IN RRSIG NSEC3 13 2 900 20250804012038 20250728001038 20545 com. fnIjm2HNoTq7wtFOrglxDYQZFBFxGoTIdQya88ryzO4wI4gn+p36M71D fGOsZoaVd+ntw/K7530z3v4Uubwt2Q==
;; Received 644 bytes from 2001:503:83eb::30#53(c.gtld-servers.net) in 79 ms
```

第四步對應到 [DNS Lookup](#dns-lookup) 的 `6.`、`7.` 步，可以看到 dig 從 4 組權威伺服器中挑選了 `ns1.google.com` 進行了 A 記錄的查詢，最後取得 A 記錄儲存的 IP 位址 `142.250.198.78`

```bash
google.com.		300	IN	A	142.250.198.78
;; Received 55 bytes from 216.239.32.10#53(ns1.google.com) in 65 ms
```

## Summary

DNS 是網路基礎中，重要的組成部分；本篇文章介紹了 DNS 的基礎知識與實務應用。

在實際應用中，DNS 的功能五花八門，除了可以依照地理位置提供距離最近的服務 IP 位址，更是一個適合進行底層應用服務抽換的抽象層！

掌握好 dig 工具的使用，配合對 DNS 機制的理解，相信未來不管是提出解決方案，還是排除網路問題，都可以有更好的思考方向！

## References

- [What is DNS? | How DNS works | Cloudflare](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [DNS Records Explained](https://www.youtube.com/watch?v=HnUDtycXSNE)
[rfc-editor.org/rfc/rfc1034.txt](https://www.rfc-editor.org/rfc/rfc1034.txt)
- [DNS 線上教學研究計畫](https://dns-learning.twnic.tw/index.html)
- [自訂網域很難嗎？DNS 的限制與實踐](https://blog.kenwsc.com/custom-domain-and-dns/)
- [dig Command in Linux with Examples - GeeksforGeeks](https://www.geeksforgeeks.org/linux-unix/dig-command-in-linux-with-examples/)
- [DNS Records Explained](https://www.youtube.com/watch?v=HnUDtycXSNE)
