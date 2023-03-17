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

### Usage

```shell
curl [options...] <url>
```

### Example

```shell
curl --request GET \
     --url "https://api.github.com/octocat"
```

## 透過 write out format 取得傳輸階段的 latency

- 先建立 format file

```txt
// .time-format.txt

\n----------\n
time_namelookup:  %{time_namelookup}s\n
time_connect:  %{time_connect}s\n
time_appconnect:  %{time_appconnect}s\n
time_pretransfer:  %{time_pretransfer}s\n
time_redirect:  %{time_redirect}s\n
time_starttransfer:  %{time_starttransfer}s\n
----------\n
time_total:  %{time_total}s\n
```

- 透過 curl 取得資源

```shell
curl -X "GET" \
     -w "@time-format.txt" \
     --header "Authorization: Bearer ${GITHUB_TOKEN}" \
     "https://api.github.com/user"

{
     "login": "TommyLin81",
     ...
}
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

## Reference

<https://curl.se/docs/manpage.html>
