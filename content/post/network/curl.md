---
title: "curl 實用小技巧"
date: 2023-03-17T21:51:22+08:00
description: "工程師須知的 curl 用法"
tags: [network, cmd]
---
## Description

curl 是一個傳輸資料用的小工具，支援非常多的傳輸協議。

方便工程師在 terminal 下就可以進行資源的取得！

對於一個注重效能的工程師來說，curl 也可以顯示傳輸中的每個過程所花費的時間 :D

## 基本操作

### usage

```shell
curl [options...] <url>
```

### example

```shell
curl --request GET \
     --url "https://api.github.com/octocat"
```

## 透過 write out format 取得傳輸階段的 latency

### usage

```shell
curl -w <format> <url>
```

### example 1

- 透過 `-w "remote_ip: %{remote_ip}"`，直接輸出 format 資訊

```shell
curl --request GET \
     -w "remote_ip: %{remote_ip}" \
     --url "https://api.github.com/octocat"
```

### example 2

- 建立 `time-format.txt`，檔案內容為顯示資訊的 format

```shell
cat > time-format.txt << 'EOF'
\n----------\n
time_namelookup:  %{time_namelookup}s\n
time_connect:  %{time_connect}s\n
time_appconnect:  %{time_appconnect}s\n
time_pretransfer:  %{time_pretransfer}s\n
time_redirect:  %{time_redirect}s\n
time_starttransfer:  %{time_starttransfer}s\n
----------\n
time_total:  %{time_total}s\n
EOF
```

- 透過 `curl -w "@time-format.txt"` 套用指定檔案內的 format

```shell
curl -X "GET" \
     -w "@time-format.txt" \
     --header "Authorization: Bearer ${GITHUB_TOKEN}" \
     "https://api.github.com/user"

// ...
----------
time_namelookup:  0.007930s
time_connect:  0.092922s
time_appconnect:  0.201676s
time_pretransfer:  0.201888s
time_redirect:  0.000000s
time_starttransfer:  0.526371s
----------
time_total:  0.526832s

```

### 名詞解釋

- `time_namelookup`：DNS 完成域名（api.github.com）解析的時間點
- `time_connect`：TCP 完成連線的時間點
- `time_appconnect`：SSL/SSH/etc 完成交握與連線的時間點
- `time_pretransfer`：所有連線都完成，準備傳輸第一個資料的時間點
- `time_redirect`：完成所有轉址的時間點
- `time_starttransfer`：curl 收到 server 回傳第一個 byte 的時間點
- `time_total`：curl 接收完全部 response 的時間點

![curl-transfer](/images/curl-transfer.png)
[picture source](https://blog.cloudflare.com/a-question-of-timing/#timing-with-curl)

</details>

## Reference

<https://curl.se/docs/manpage.html>
