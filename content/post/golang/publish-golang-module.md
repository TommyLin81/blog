---
title: "Publish Golang Module"
date: 2022-08-06T01:01:57+08:00
description: "簡單記錄一下，把一個 public 的 golang 專案做成 publish library 的步驟"
tags: [golang]
---

## 前言

在 golang 要把專案變成 publish library 非常簡單

先不考慮 library 長久開發上需要特別規劃的版本與 dependency 等細節

簡單的實作一個可以用的被 `go get` 的 library

## Publish library

### 要能被使用前需要做到的事情

- 專案要使用 go module（use go.mod）
- repo 要有 Redistributable license
- repo 要有版本號 tag（[version numbering](https://go.dev/doc/modules/version-numbers)）

### 步驟

附上實作的 [repo](https://github.com/TommyLin81/hello-golang)，步驟寫在 readme 上面
