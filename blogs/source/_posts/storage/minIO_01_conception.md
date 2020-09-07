---
title: MinIO 01 conception
tags: storage
categories:
- storage
---

## **官网**
official website:[https://min.io/](https://min.io/)


## **存储形式**

文件对象上传到 MinIO ，会在对应的数据存储磁盘中，以 Bucket 名称为目录，文件名称为下一级目录，文件名称下是 part.1 和 xl.json，前者是编码数据块及检验块，后者是元数据文件.  

## **纠删码EC（Erasure Code）**

MinIO 使用纠删码机制来保证高可靠性，使用 highwayhash 来处理数据损坏（ Bit Rot Protection ）。关于纠删码，简单来说就是可以通过数学计算，把丢失的数据进行还原，它可以将n份原始数据，增加m份数据，并能通过n+m份中的任意n份数据，还原为原始数据。即如果有任意小于等于m份的数据失效，仍然能通过剩下的数据还原出来。举个最简单例子就是有两个数据(d1, d2)，用一个校验和y（d1 + d2 = y）即可保证即使丢失其中一个，依然可以还原数据。如丢失 d1 ，则使用 y - d2 = d1 还原，同理，d2 丢失或者y丢失，均可通过计算得出.  

EC 的具体应用实现中， RS（Reed-Solomen）是 EC 的一种更简单快捷的实现，可以通过矩阵运算，还原数据。MinIO 将对象拆分成N/2数据和N/2 校验块 。具体的数学矩阵运算及证明，可以参考文章《Erasure-Code-擦除码-1-原理篇》及《EC纠删码原理》.  



