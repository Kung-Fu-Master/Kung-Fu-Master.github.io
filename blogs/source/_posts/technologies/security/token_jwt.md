---
title: Token jwt
tags: security
categories:
- technologies
- security
---

# Reference
 * [csdn blog](https://www.cnblogs.com/MyCode1990/p/13096423.html)
 * [简书](https://www.jianshu.com/p/4941a269a9d8)

# **Json Web Token(JWT)简介**
JWT属于Token验证方式的一种方法.
> JSON Web Token（JWT）是一个开放标准（RFC 7519），它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。此信息可以验证和信任，因为它是数字签名的。JWTs可以使用密钥（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名.

# **传统服务端验证客户端身份的方法**
HTTP 是一种没有状态的协议，也就是它并不知道是谁是访问应用。这里我们把用户看成是客户端，客户端使用用户名还有密码通过了身份验证，不过下回这个客户端再发送请求时候，还得再验证一下。

解决的方法就是，当用户请求登录的时候，如果没有问题，我们在服务端生成一条记录，这个记录里可以说明一下登录的用户是谁，然后把这条记录的 ID 号发送给客户端，客户端收到以后把这个 ID 号存储在 Cookie 里，下次这个用户再向服务端发送请求的时候，可以带着这个 Cookie ，这样服务端会验证一个这个 Cookie 里的信息，看看能不能在服务端这里找到对应的记录，如果可以，说明用户已经通过了身份验证，就把用户请求的数据返回给客户端。

上面说的就是 Session，我们需要在服务端存储为登录的用户生成的 Session ，这些 Session 可能会存储在内存，磁盘，或者数据库里。我们可能需要在服务端定期的去清理过期的 Session 。

# **基于 Token 的身份验证方法**
Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生。  
Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。  
使用基于 Token 的身份验证方法，在服务端不需要存储用户的登录记录。大概的流程是这样的：
 1. 客户端使用用户名跟密码请求登录
 2. 服务端收到请求，去验证用户名与密码
 3. 验证成功后，服务端会签发一个 Token，再把这个 Token 发送给客户端
 4. 客户端收到 Token 以后可以把它存储起来，比如放在 Cookie 里或者 Local Storage 里
 5. 客户端每次向服务端请求资源的时候需要带着服务端签发的 Token
 6. 服务端收到请求，然后去验证客户端请求里面带着的 Token，如果验证成功，就向客户端返回请求的数据

# **为什么要使用JSON Web Token**
 * Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮.
 * JSON比XML不那么冗长，当它被编码时，它的大小也更小，使得JWT比SAML更紧凑。这使得JWT成为在HTML和HTTP环境中传递的一个很好的选择。
 * 安全方面，使用HMAC算法，SWT只能由共享密钥对称签名。但是，JWT和SAML令牌可以使用X.509证书形式的公钥/私钥对进行签名。与签名JSON的简单性相比，使用XML数字签名来签名XML而不引入隐藏的安全漏洞是非常困难的。
 * JSON解析器在大多数编程语言中都很常见，因为它们直接映射到对象。相反，XML没有自然的文档到对象的映射。这使得使用JWT比使用SAML断言更容易。
 * 在使用方面，JWT是在互联网上使用的。这突出了JSON Web令牌在多个平台（尤其是移动平台）上客户端处理的方便性。
 * 比较编码JWT和编码SAML的长度.

# **JWT(Json Web Token 一种Token验证方法)**

实施 Token 验证的方法挺多的，还有一些标准方法，比如 JWT，读作：jot ，表示：JSON Web Tokens 。JWT 标准的 Token 有三个部分：

 * header（头部）
 * payload（荷载,数据）
 * signature（签名）
中间用点分隔开，并且都会使用 Base64 编码，所以真正的 Token 看起来像这样：

	eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc


## **Header 头部**
 * 令牌的类型
 * 正在使用的签名算法(HMAC, SHA256, RSA等)。

每个 JWT token 里面都有一个 header，也就是头部数据。里面包含了使用的算法，这个 JWT 是不是带签名的或者加密的。主要就是说明一下怎么处理这个 JWT token 。  
头部里包含的东西可能会根据 JWT 的类型有所变化，比如一个加密的 JWT 里面要包含使用的加密的算法。唯一在头部里面要包含的是 alg 这个属性，如果是加密的 JWT，这个属性的值就是使用的签名或者解密用的算法。如果是未加密的 JWT，这个属性的值要设置成 none。  
示例：

	{
	  "typ": "JWT",
	  "alg": "HS256"
	}
意思是这个 JWT 用的算法是 HS256。上面的内容得用 base64url 的形式编码一下，所以就变成这样：

	eyJhbGciOiJIUzI1NiJ9

## **Payload 有效负载**
有效负载包含了"声明(claims)", 有三种类型的claims：
 * registered claims 已注册的 (不是强制的，而是推荐,iss（发行者）、exp（到期时间）、sub（主题）、aud（受众）和其他)
 * public claims 公开的 (可以由使用JWT的用户随意定义, 为了避免冲突，应该在[IANA JSON Web令牌注册表](https://www.iana.org/assignments/jwt/jwt.xhtml)中定义它们，或者将它们定义为包含防冲突命名空间的URI)
 * private claims 私有的

Payload 里面是 Token 的具体内容，这些内容里面有一些是标准字段，你也可以添加其它需要的内容。下面是标准字段：
 * iss：Issuer，发行者
 * sub：Subject，主题
 * aud：Audience，观众
 * exp：Expiration time，过期时间
 * nbf：Not before
 * iat：Issued at，发行时间
 * jti：JWT ID

比如下面这个 Payload ，用到了 iss 发行人，还有 exp 过期时间这两个标准字段。另外还有两个自定义的字段，一个是 name ，还有一个是 admin 。

	{
	  "sub": "1234567890",
	  "name": "John Doe",
	  "iss": "ninghao.net",
	  "exp": "1438955445",
	  "admin": true
	}
使用 base64url 编码以后就变成了这个样子：

	 eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ
**请注意，对于已签名的令牌，此信息虽然受保护不受篡改，但任何人都可以读取。除非经过加密，否则不要将机密信息放在JWT的有效负载或头部**.

## **Signature 签名**
要创建签名部分，您必须已经有了 经过编码的**头部**、经过编码的**负载**、一个**秘钥**、在头部中指定的**算法**，这样就可以进行签名了.  
Signature这部分内容有三个部分，先是用 Base64 编码的 header.payload ，再用加密算法加密一下，加密的时候要放进去一个 Secret ，这个相当于是一个密码，这个密码秘密地存储在服务端。  

 * header
 * payload
 * secret


	const encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload); 
	HMACSHA256(encodedString, 'secret');
处理完成以后看起来像这样：

	 SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
最后这个在服务端生成并且要发送给客户端的 Token 看起来像这样：

	 eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
输出的内容是三个由点分隔的Base64 URL字符串。它可以在HTML和HTTP环境中轻松传递，它比XML的标准（如SAML）更加紧凑.  
客户端收到这个 Token 以后把它存储下来，下回向服务端发送请求的时候就带着这个 Token 。服务端收到这个 Token ，然后进行验证，通过以后就会返回给客户端想要的资源。

## **签名的作用:**  
(1) 签名用于验证消息在传输过程中没有被更改。
(2) 使用私钥签名的令牌，还可以验证JWT的发送者是它所说的发送者。

# **签发 Json Web Token(JWT)**
 * JWT 对 “信息” 进行签名，产生一个令牌。
 * 签名的令牌可以验证其中包含的内容的完整性（防篡改）。
 * 也可对“信息”加密，加密的令牌则对其他方隐藏这些内容。
 * 当令牌使用公钥/私钥对签名时，签名还证明只有持有私钥的一方才是签名方。可以非对称加密方式证明了


## **HS256 算法签发Json Web Token(JWT)**
这种算法需要一个密钥（密码）.  
在项目里随便添加一个 .js 文件，比如 index.js，在文件里添加下面这些代码：  

	const jwt = require('jsonwebtoken')
	// Token 数据
	const payload = {
	name: 'wanghao',
	admin: true
	}
	// 密钥
	const secret = 'ILOVENINGHAO'
	// 签发 Token
	const token = jwt.sign(payload, secret, { expiresIn: '1day' })
	// 输出签发的 Token
	console.log(token)
非常简单，就是用了刚刚为项目安装的 jsonwebtoken 里面提供的 jwt.sign 功能，去签发一个 token。这个 sign 方法需要三个参数：
 * playload：签发的 token 里面要包含的一些数据。
 * secret：签发 token 用的密钥，在验证 token 的时候同样需要用到这个密钥。
 * options：一些其它的选项。

在命令行下面，用 node 命令，执行一下项目里的 index.js 这个文件（node index.js），会输出应用签发的 token：

	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlLCJpYXQiOjE1MjkwMzM5MDYsImV4cCI6MTUyOTEyMDMwNn0.DctA2QlUCrM6wLWkIO78wBVN0NLpjoIq4T5B_2WJ-PU
上面的 Token 内容并没有加密，所以如果用一些 JWT 解码功能，可以看到 Token 里面包含的内容，内容由三个部分组成，像这样：

	// header
	{
	  "alg": "HS256", 
	  "typ": "JWT"
	}
	// payload
	{
	  "admin": true, 
	  "iat": 1529033906, 
	  "name": "wanghao", 
	  "exp": 1529120306
	}
	// signature
	DctA2QlUCrM6wLWkIO78wBVN0NLpjoIq4T5B_2WJ-PU
> 假设用户通过了某种身份验证，你就可以使用上面的签发 Token 的功能为用户签发一个 Token。一般在客户端那里会把它保存在 Cookie 或 LocalStorage 里面。
> 用户下次向我们的应用请求受保护的资源的时候，可以在请求里带着我们给它签发的这个 Token，后端应用收到请求，检查签名，如果验证通过确定这个 Token 是我们自己签发的，那就可以为用户响应回他需要的资源。

## **RS256 算法签发Json Web Token(JWT)**
默认签发还有验证 Token 的时候用的是 HS256 算法，这种算法需要一个密钥（密码）。我们还可以使用 RS256 算法签发与验证 JWT。这种方法可以让我们分离开签发与验证，签发时需要用一个密钥，验证时使用公钥，也就是有公钥的地方只能做验证，但不能签发 JWT。

在项目下面创建一个新的目录，里面可以存储即将生成的密钥与公钥文件。

	$ cd ~/desktop/jwt-demo
	$ mkdir config
	$ cd config
**密钥:**
先生成一个密钥文件：

	 ssh-keygen -t rsa -b 2048 -f private.key
**公钥:**
基于上面生成的密钥，再去创建一个对应的公钥：

	 openssl rsa -in private.key -pubout -outform PEM -out public.key
**签发 JWT（RS256 算法）**
用 RS256 算法签发 JWT 的时候，需要从文件系统上读取创建的密钥文件里的内容。

	 const fs = require('fs')
	 // 获取签发 JWT 时需要用的密钥
	 const privateKey = fs.readFileSync('./config/private.key')
	 签发仍然使用 jwt.sign 方法，只不过在选项参数里特别说明一下使用的算法是 RS256：
	 // 签发 Token
	 const tokenRS256 = jwt.sign(payload, privateKey, { algorithm: 'RS256' })
	 // 输出签发的 Token
	 console.log('RS256 算法：', tokenRS256)

# **验证Json Web Token(JWT)**

## **HS256 算法验证Json Web TOken(JWT)**
需要一个签发token时候用的密钥（密码）来验证,这种算法需要一个密钥（密码）.  
验证 JWT 的用效性，确定一下用户的 JWT 是我们自己签发的，首先要得到用户的这个 JWT Token，然后用 jwt.verify 这个方法去做一下验证。这个方法是 Node.js 的 jsonwebtoken 这个包里提供的，在其它的应用框架或者系统里，你可能会找到类似的方法来验证 JWT。  
打开项目的 index.js 文件，里面添加几行代码：

	// 验证 Token
	jwt.verify(token, 'bad secret', (error, decoded) => {
	  if (error) {
	    console.log(error.message)
	    return
	  }
	  console.log(decoded)
	})
把要验证的 Token 数据，还有签发这个 Token 的时候用的那个密钥告诉 verify 这个方法，在一个回调里面有两个参数，error 表示错误，decoded 是解码之后的 Token 数据。

执行：

	$ node ~/desktop/jwt-demo/index.js
输出：

	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlLCJpYXQiOjE1MjkwMzQ3MzMsImV4cCI6MTUyOTEyMTEzM30.swXojmu7VimFu3BoIgAxxpmm2J05dvD0HT3yu10vuqU
	invalid signature
注意输出了一个 invalid signature ，表示 Token 里的签名不对，这是因为我们组长 verify 方法提供的密钥并不是签发 Token 的时候用的那个密钥。这样修改一下：

	jwt.verify(token, secret, (error, decoded) => { ...
再次运行，会输出类似的数据：

	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlLCJpYXQiOjE1MjkwMzUzODYsImV4cCI6MTUyOTEyMTc4Nn0.mkNrt4TfcfmP22xd3C_GQn8qnUmlB39dKT9SpIBTBGI
	{ name: 'wanghao', admin: true, iat: 1529035386, exp: 1529121786 }

## **RS256 算法验证Json Web Token(JWT)**
验证 JWT（RS256 算法）
验证使用 RS256 算法签发的 JWT，需要在文件系统上读取公钥文件里的内容。然后用 jwt 的 verify 方法去做验证。

	// 获取验证 JWT 时需要用的公钥
	const publicKey = fs.readFileSync('./config/public.key')
	
	// 验证 Token
	jwt.verify(tokenRS256, publicKey, (error, decoded) => {
	  if (error) {
	    console.log(error.message)
	    return
	  }
	  console.log(decoded)
	})

# **使用场景**
![](token_scene1.png)

1. 应用程序或客户端，向授权服务器请求授权。
2. 当授权被通过时，授权服务器将向应用程序返回一个访问令牌token。
3. 应用程序使用访问令牌访问受保护的资源。
**请注意，使用签名的令牌，令牌中包含的所有信息都将公开给用户或其他方（虽然他们无法更改它，但可以阅读）。这意味着您不应将机密信息放入令牌中.**



