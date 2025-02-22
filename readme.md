# [Tommy's Blog](https://blog.tommylin.tech/)

## Document

Using [Hugo](https://gohugo.io/getting-started/installing/) + [hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod/tree/master) theme.

Recording learned technologies and sharing book reviews.

## Requirement

1. [Git](https://git-scm.com/downloads)
2. [Hugo](https://gohugo.io/getting-started/installing/)

## Usage

### How to Build

**get project：**

```shell
git clone https://github.com/TommyLin81/blog.git

cd blog
```

**get submodule：**

```shell
git submodule update --init --recursive
```

**(optional) update submodule to latest version:**

```shell
git submodule sync --recursive
git submodule update --init --recursive --remote
```

**run local sites：**

```shell
hugo server -D
```

**default local sites：**

<http://localhost:1313/blog/>

### How to Add a New Content

**add new Content：**

```shell
# hugo new <path>/<page_name>
hugo new post/new-first-post.md

hugo new post/docker/new-first-post.md
```
