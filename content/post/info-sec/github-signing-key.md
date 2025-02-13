---
title: "Github Signing Key"
date: 2025-02-13T14:13:39+08:00
description: "為何你需要設定 Github 的 Signing Key？"
tags: [github, InfoSec]
---

## Overview

只要將 git config 的 `user.email` 設定成別人的，就可以以別人的身份推 commit 到 Github，而 Signing key 就是為了防止這件事情而誕生的。

使用者本人在自己的帳號下向 Github 註冊 Signing Key（`SSH Key` or `GPG Key`），並在之後的 commit 都用這把 key 簽名，Github 會對 commit 上的簽名進行核對，只有本人簽過的 commit 才會被標注 `Verified`

下圖就是將 `user.email` 改為 Linux 之父 [Linus Torvalds](https://github.com/torvalds)的信箱，並提交隨便一份 commit，可以看到 Github 上確實顯示了這是來自 Linux 之父提交的 commit，不過 `Unverified` tag 提醒了大家需要好好審視這個 commit 來源的真實性。
![linxu_torvalds_is_here](/blog/images/linxu_torvalds_is_here.png)

## Setup Guide

1. 先去 github.com setting > SSH and GPG keys，從 `New SSH key` 新增一個 signing type 的 key
2. terminal 新增三個 git config `user.signingkey`、`gpg.format`、`commit.gpgsign`

```sh
$ git config --global user.signingkey=<your public signing key>
# 允許 git 使用 SSH Key 來簽署 commit 和 tag
$ git config --global gpg.format=ssh
# commit 指令預設進行簽名
$ git config --global commit.gpgsign=true
```

## (Optional) Verify Signature in Local

1. 新增被允許簽名的使用者到白名單  `~/.ssh/allowed_signers`

```sh
# <user email> <key tpye> <public key>
$ echo "<your email> $(cat ~/.ssh/<your public signing key>)" >> ~/.ssh/allowed_signers

$ cat ~/.ssh/allowed_signers
tommylin@gmail.com ssh-ed25519 AAAAC3Nza......
```

2. terminal 新增 git config `gpg.ssh.allowedsignersfile=<your allowed_signers file>`

```sh
git config --global gpg.ssh.allowedsignersfile=~/.ssh/allowed_signers
```

3. 新增一個 commit 測試簽名是否合法

```sh
git commit -m 'test signature'
git log --show-signature -1
```

## Reference

- [Managing commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification)
