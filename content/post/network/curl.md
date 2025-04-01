---
title: "curl 實用小技巧"
date: 2023-03-17T21:51:22+08:00
description: "工程師須知的 curl 用法"
tags: [network, cmd]
---
curl 是一個傳輸資料用的小工具，支援非常多的傳輸協議。

讓工程師可以在 Terminal 上快速存取 API 或其他網路資源。

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

## 顯示傳輸延遲時間（latency）資訊

### Usage

```shell
curl -w <format> <url>
```
`-w`（或 `--write-out`）可以讓你自定輸出格式，列出請求過程中每個階段所花費的時間。

### Example

- 透過 `-w "remote_ip: %{remote_ip}"`，直接輸出 format 資訊

```shell
curl --request GET \
     -w "remote_ip: %{remote_ip}" \
     --url "https://api.github.com/octocat"
```

- 建立 `time-format.txt`，檔案內容為顯示資訊的 format

```shell
cat << 'EOF' > time-format.txt  # 將格式寫入檔案
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

實務上，如果你發現某個 API 請求速度變慢，可以使用 `curl -w` 分析是哪個階段（DNS、TCP、TLS、Server 回應）變慢，有助於進行效能調校或報告問題。

### 名詞解釋

- `time_namelookup`：DNS 完成域名（api.github.com）解析的時間點
- `time_connect`：TCP 完成連線的時間點
- `time_appconnect`：SSL/SSH/etc 完成交握與連線的時間點
- `time_pretransfer`：所有連線都完成，準備傳輸第一個資料的時間點
- `time_redirect`：完成所有轉址的時間點
- `time_starttransfer`：curl 收到 server 回傳第一個 byte 的時間點
- `time_total`：curl 接收完全部 response 的時間點

![curl-transfer](/images/curl-transfer.png)
[picture resource](https://blog.cloudflare.com/a-question-of-timing/#timing-with-curl)

## References

- [curl - How To Use](https://curl.se/docs/manpage.html)
- [A Question of Timing](https://blog.cloudflare.com/a-question-of-timing/#timing-with-curl)
