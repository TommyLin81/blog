## [Tommy's Blog](https://tommylin.kkbox.io/blog/)

### Overview

[TOC]

### Document

紀錄工作上遇到覺得不錯的東西

### Requirement

1. [Git](https://git-scm.com/downloads)
2. [Hugo](https://gohugo.io/getting-started/installing/)

### How to Build

**get project：**

```shell
git clone git@gitlab.kkinternal.com:tommylin/blog.git

cd blog
```

**get submodule：**

```shell
git submodule update --init --recursive
```

**run local sites：**

```shell
hugo server -D
```

**default local sites：**

<http://localhost:1313/blog/>

### How to Add New Content

**add new Content：**

```shell
# hugo new <path>/<page_name>
hugo new post/new-first-post.md

hugo new post/docker/new-first-post.md
```
