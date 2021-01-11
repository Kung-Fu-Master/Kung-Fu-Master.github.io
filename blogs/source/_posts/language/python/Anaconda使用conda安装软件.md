---
title: Aanconda conda配置proxy下载软件
tags:
categories:
- language
- python
---

## conda 下载软件

配置公司proxy:
```
conda config --set proxy_servers.http http://<proxy>:port
conda config --set proxy_servers.https http://<proxy>:port
```
<!-- more -->

### pandoc 安装

#### windows msi安装包进行安装
The simplest way to get the latest pandoc release is to use the installer.
[Download the latest installer](https://github.com/jgm/pandoc/releases)

#### 使用conda进行安装
pandoc 可以使用 conda 安装，关于 conda 的安装和使用可以参考前一篇文档 conda_pip_info。 使用如下命令安装 pandoc
```
$ conda install pandoc
```
会等待一段时间solving environment, 之后就提示下载了.

安装完毕后，还需要安装 pandoc-xnos 插件，主要功能是图片、表格、公式等编号的索引，此软件包采用 python 编写，使用 pip 命令安装。

```
$ pip install pandoc-xnos
```
由于 pandoc 生成 PDF 文件需要使用 latex 工具，因此还需要安装 texlive 软件，推荐在 linux 平台直接使用 yum 或者 apt 命令安装。

