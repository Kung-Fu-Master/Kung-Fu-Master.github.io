---
title: ECDHE-RSA-AES256-GCM-SHA384 解释
tags: security
categories:
- technologies
- security
---

## ECDHE-RSA-AES256-GCM-SHA384 解释

“密钥交换算法 + 签名算法 + 对称加密算法 + 摘要算法”

“握手时使用 ECDHE 算法进行密钥交换，用 RSA 签名和身份认证，握手后的通信使用 AES 对称算法，密钥长度 256 位，分组模式是 GCM，摘要算法 SHA384 用于消息认证和产生随机数。”

<!-- more -->

