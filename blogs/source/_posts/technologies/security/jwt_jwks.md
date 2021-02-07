---
title: jwt jwks
tags: security
categories:
- technologies
- security
---

referencd:
https://docs.aws.amazon.com/zh_cn/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html
https://blog.unosquare.com/why-and-how-to-improve-jwt-security-with-jwks-key-rotation-in-java

<!-- more -->

## jwt

一个 JSON Web 令牌 (JWT) 包含三个部分：

1. 标头

2. 负载

3. 签名

```
11111111111.22222222222.33333333333
```
这些部分编码为base64url字符串,并用点 **`(.)`** 字符分隔。如果您的JWT不符合此结构, 则视为无效, 不接受.  

## jwks
这是一个样本 jwks.json 文件:
```
{
	"keys": [{
		"kid": "1234example=",
		"alg": "RS256",
		"kty": "RSA",
		"e": "AQAB",
		"n": "1234567890",
		"use": "sig"
	}, {
		"kid": "5678example=",
		"alg": "RS256",
		"kty": "RSA",
		"e": "AQAB",
		"n": "987654321",
		"use": "sig"
	}]
}
```

**1. 密钥ID kid(Key ID)**
的 kid 是一个提示,指示用于保护令牌的JSONWeb签名(JWS)的密钥。

**2. 算法alg(encryption algorithm)**
的 alg 标头参数表示用于保护ID令牌的密码算法。用户池使用 RS256 加密算法，这是一种采用 SHA-256 的 RSA 签名。有关 RSA 的更多信息，请参阅 RSA 密码术。

更多算法参考[specification](https://tools.ietf.org/html/rfc7518#section-3.3)

| "alg" Param Value | Digital Signature Algorithm |
| :-----: | :-----: |
| RS256   | RSASSA-PKCS1-v1_5 using SHA-256 |
| RS384   | RSASSA-PKCS1-v1_5 using SHA-384 |
| RS512   | RSASSA-PKCS1-v1_5 using SHA-512 |

**3. 密钥类型kty(Key Type)**
kty 参数标识与密钥结合使用的加密算法系列，例如，在本示例中为“RSA”。

**4. RSA指数(e)**
 e 参数包含RSA公钥的指数值。它表示为采用 Base64urlUInt 编码的值。
 e Parameter is used to define the RSA public exponent.

**5. RSA模量(n)**
 n 参数包含RSA公钥的模量值。它表示为采用 Base64urlUInt 编码的值.  
 n Parameter is used to define the modulus for both the public and private keys. Its length, usually expressed in bits, is the key length.

**6. 使用use(Public Key Use)**
的 use 参数描述公钥的预期用途。对于本示例, use 价值 sig 表示签名。



