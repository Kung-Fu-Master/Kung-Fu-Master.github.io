---
title: istio secure naming
tags: istio
categories:
- microService
- istio
---

## secure naming
reference: https://istio.io/latest/docs/concepts/security/#secure-naming

<!-- more -->

Sidecar and perimeter proxies work as [Policy Enforcement Points](https://www.jerichosystems.com/technology/glossaryterms/policy_enforcement_point.html) (PEPs) to secure communication between clients and servers.
Sidecar 和边缘代理作为 Policy Enforcement Points(PEPs) 以保护客户端和服务器之间的通信安全.

服务器身份（Server identities）被编码在证书里，但服务名称（service names）通过服务发现或 DNS 被检索。安全命名信息将服务器身份映射到服务名称。身份 A 到服务名称 B 的映射表示“授权 A 运行服务 B“。控制平面监视 apiserver，生成安全命名映射，并将其安全地分发到 PEPs。 以下示例说明了为什么安全命名对身份验证至关重要。

假设运行服务 datastore 的合法服务器仅使用 infra-team 身份。恶意用户拥有 test-team 身份的证书和密钥。恶意用户打算模拟服务以检查从客户端发送的数据。恶意用户使用证书和 test-team 身份的密钥部署伪造服务器。假设恶意用户成功攻击了发现服务或 DNS，以将 datastore 服务名称映射到伪造服务器。

当客户端调用 datastore 服务时，它从服务器的证书中提取 test-team 身份，并用安全命名信息检查 test-team 是否被允许运行 datastore。客户端检测到 test-team 不允许运行 datastore 服务，认证失败。

安全命名能够防止 HTTPS 流量受到一般性网络劫持，除了 DNS 欺骗外，它还可以保护 TCP 流量免受一般网络劫持。如果攻击者劫持了 DNS 并修改了目的地的 IP 地址，它将无法用于 TCP 通信。这是因为 TCP 流量不包含主机名信息，我们只能依靠 IP 地址进行路由，而且甚至在客户端 Envoy 收到流量之前，也可能发生 DNS 劫持。

## 双向 TLS 认证
reference: https://istio.io/latest/zh/docs/concepts/security/#mutual-TLS-authentication

Istio 通过客户端和服务器端 PEPs 建立服务到服务的通信通道，PEPs 被实现为Envoy 代理。当一个工作负载使用双向 TLS 认证向另一个工作负载发送请求时，该请求的处理方式如下：

1. Istio 将出站流量从客户端重新路由到客户端的本地 sidecar Envoy。
2. 客户端 Envoy 与服务器端 Envoy 开始双向 TLS 握手。在握手期间，客户端 Envoy 还做了安全命名检查，以验证服务器证书中显示的服务帐户是否被授权运行目标服务。
3. 客户端 Envoy 和服务器端 Envoy 建立了一个双向的 TLS 连接，Istio 将流量从客户端 Envoy 转发到服务器端 Envoy。
4. 授权后，服务器端 Envoy 通过本地 TCP 连接将流量转发到服务器服务。

