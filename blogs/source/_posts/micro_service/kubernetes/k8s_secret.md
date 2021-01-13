---
title: k8s secret
tags: istio
categories:
- microService
- kubernetes
---

## Secret
Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 secret 中比放在 Pod 的定义或者 容器镜像 中来说更加安全和灵活。 参阅 [Secret 设计文档](https://git.k8s.io/community/contributors/design-proposals/auth/secrets.md) 获取更多详细信息。

<!-- more -->

Secret 是一种包含少量敏感信息例如密码、令牌或密钥的对象。 这样的信息可能会被放在 Pod 规约中或者镜像中。 用户可以创建 Secret，同时系统也创建了一些 Secret.

Kubernetes 会验证 Secret 作为卷来源时所给的对象引用确实指向一个类型为 Secret 的对象。因此，Secret 需要先于任何依赖于它的 Pod 创建。

Secret API 对象处于某名字空间 中。它们只能由同一命名空间中的 Pod 引用。

每个 Secret 的 `大小限制为 1MB`。这是为了防止创建非常大的 Secret 导致 API 服务器 和 kubelet 的内存耗尽。然而，创建过多较小的 Secret 也可能耗尽内存。 更全面得限制 Secret 内存用量的功能还在计划中。

kubelet 仅支持从 API 服务器获得的 Pod 使用 Secret。 这包括使用 kubectl 创建的所有 Pod，以及间接通过副本控制器创建的 Pod。 它不包括通过 kubelet --manifest-url 标志，--config 标志或其 REST API 创建的 Pod（这些不是创建 Pod 的常用方法）。

以环境变量形式在 Pod 中使用 Secret 之前必须先创建 Secret，除非该环境变量被标记为可选的。 Pod 中引用不存在的 Secret 时将无法启动。

使用 secretKeyRef 时，如果引用了指定 Secret 不存在的键，对应的 Pod 也无法启动。

对于通过 `envFrom` 填充环境变量的 Secret，如果 Secret 中包含的键名无法作为合法的环境变量名称，对应的键会被跳过，该 Pod 将被允许启动。 不过这时会产生一个事件，其原因为 `nvalidVariableNames`，其消息中包含被跳过的无效键的列表。 下面的示例显示一个 Pod，它引用了包含 2 个无效键 1badkey 和 2alsobad。

```
kubectl get events
输出类似于：
LASTSEEN   FIRSTSEEN   COUNT     NAME            KIND      SUBOBJECT                         TYPE      REASON
0s         0s          1         dapi-test-pod   Pod                                         Warning   InvalidEnvironmentVariableNames   kubelet, 127.0.0.1      Keys [1badkey, 2alsobad] from the EnvFrom secret default/mysecret were skipped since they are considered invalid environment variable names.
```

## Secret 与 Pod 生命周期的关系

通过 API 创建 Pod 时，不会检查引用的 Secret 是否存在。一旦 Pod 被调度，kubelet 就会尝试获取该 Secret 的值。如果获取不到该 Secret，或者暂时无法与 API 服务器建立连接， kubelet 将会定期重试。kubelet 将会报告关于 Pod 的事件，并解释它无法启动的原因。 一旦获取到 Secret，kubelet 将创建并挂载一个包含它的卷。在 Pod 的所有卷被挂载之前， Pod 中的容器不会启动。

## Secret 概览

要使用 Secret，Pod 需要引用 Secret。
Pod 可以用三种方式之一来使用 Secret：

- 作为挂载到一个或多个容器上的 [卷](https://kubernetes.io/zh/docs/concepts/storage/volumes/)中的[文件](https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)
  中的[文件](#在Pod中使用Secret文件)
- 作为[容器的环境变量](#以环境变量的形式使用Secrets)
- 由 [kubelet在为Pod拉取镜像时使用](#使用imagePullSecret)

Secret 对象的名称必须是合法的 [DNS 子域名](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names). 在为创建 Secret 编写配置文件时，你可以设置 `data` 与/或 `stringData` 字段。 `data` 和 `stringData` 字段都是可选的。 `data` 字段中所有键值都必须是 base64 编码的字符串。如果不希望执行这种 base64 字符串的转换操作，你可以选择设置 stringData 字段，其中可以使用任何字符串作为其取值。

## Secret 的类型
在创建 Secret 对象时，你可以使用 [Secret](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#secret-v1-core) 资源的 `type` 字段，或者与其等价的 `kubectl` 命令行参数（如果有的话）为其设置类型。 Secret 的类型用来帮助编写程序处理 Secret 数据。

Kubernetes 提供若干种内置的类型，用于一些常见的使用场景。 针对这些类型，Kubernetes 所执行的合法性检查操作以及对其所实施的限制各不相同。

| 内置类型 | 用法 |
| :------ | :----- |
| Opaque | <div style="width: 300pt">用户定义的任意数据</div> |
| kubernetes.io/service-account-token | 服务账号令牌 |
| kubernetes.io/dockercfg | ~/.dockercfg 文件的序列化形式 |
| kubernetes.io/dockerconfigjson | ~/.docker/config.json 文件的序列化形式 |
| kubernetes.io/basic-auth | 用于基本身份认证的凭据 |
| kubernetes.io/ssh-auth | 用于 SSH 身份认证的凭据 |
| kubernetes.io/tls | 用于 TLS 客户端或者服务器端的数据 |
| bootstrap.kubernetes.io/token | 启动引导令牌数据 |

通过为 Secret 对象的 type 字段设置一个非空的字符串值，你也可以定义并使用自己 Secret 类型。如果 type 值为空字符串，则被视为 Opaque 类型。 Kubernetes 并不对类型的名称作任何限制。不过，如果你要使用内置类型之一， 则你必须满足为该类型所定义的所有要求。

### Opaque Secret

当 Secret 配置文件中未作显式设定时，默认的 Secret 类型是 Opaque。 当你使用 kubectl 来创建一个 Secret 时，你会使用 generic 子命令来标明 要创建的是一个 Opaque 类型 Secret。 例如，下面的命令会创建一个空的 Opaque 类型 Secret 对象：

```
kubectl create secret generic empty-secret
kubectl get secret empty-secret
```

输出类似于

```
NAME           TYPE     DATA   AGE
empty-secret   Opaque   0      2m6s
```

DATA 列显示 Secret 中保存的数据条目个数。 在这个例子种，0 意味着我们刚刚创建了一个空的 Secret。


### 服务账号令牌 Secret
Kubernetes 在创建 Pod 时会自动创建一个服务账号 Secret 并自动修改你的 Pod 以使用该 Secret。该服务账号令牌 Secret 中包含了访问 Kubernetes API 所需要的凭据。

如果需要，可以禁止或者重载这种自动创建并使用 API 凭据的操作。 不过，如果你仅仅是希望能够安全地访问 API 服务器，这是建议的工作方式。

参考 [ServiceAccount](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-service-account/) 文档了解服务账号的工作原理。你也可以查看 [Pod](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#pod-v1-core) 资源中的 automountServiceAccountToken 和 serviceAccountName 字段文档，了解 从 Pod 中引用服务账号。


### Docker 配置 Secret

你可以使用下面两种 `type` 值之一来创建 Secret，用以存放访问 Docker 仓库 来下载镜像的凭据。

 * `kubernetes.io/dockercfg`
 * `kubernetes.io/dockerconfigjson`

下面是一个 kubernetes.io/dockercfg 类型 Secret 的示例：

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
        "<base64 encoded ~/.dockercfg file>"
```

> **说明：**
> 如果你不希望执行 base64 编码转换，可以使用 stringData 字段代替。

当你使用清单文件来创建这两类 Secret 时，API 服务器会检查 data 字段中是否 存在所期望的主键，并且验证其中所提供的键值是否是合法的 JSON 数据。 不过，API 服务器不会检查 JSON 数据本身是否是一个合法的 Docker 配置文件内容。

```
kubectl create secret docker-registry secret-tiger-docker \
  --docker-username=tiger \
  --docker-password=pass113 \
  --docker-email=tiger@acme.com
```

上面的命令创建一个类型为 kubernetes.io/dockerconfigjson 的 Secret。 如果你对 data 字段中的 .dockerconfigjson 内容进行转储，你会得到下面的 JSON 内容，而这一内容是一个合法的 Docker 配置文件。

```
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "tiger",
      "password": "pass113",
      "email": "tiger@acme.com",
      "auth": "dGlnZXI6cGFzczExMw=="
    }
  }
}
```

### 基本身份认证 Secret 
`kubernetes.io/basic-auth` 类型用来存放用于基本身份认证所需的凭据信息。 使用这种 Secret 类型时，Secret 的 data 字段必须包含以下两个键：

 * `username`: 用于身份认证的用户名；
 * `password`: 用于身份认证的密码或令牌。
以上两个键的键值都是 base64 编码的字符串。 当然你也可以在创建 Secret 时使用 `stringData` 字段来提供明文形式的内容。 下面的 YAML 是基本身份认证 Secret 的一个示例清单：

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin
  password: t0p-Secret
```

提供基本身份认证类型的 Secret 仅仅是出于用户方便性考虑。 你也可以使用 Opaque 类型来保存用于基本身份认证的凭据。 不过，使用内置的 Secret 类型的有助于对凭据格式进行归一化处理，并且 API 服务器确实会检查 Secret 配置中是否提供了所需要的主键。

### SSH 身份认证 Secret

Kubernetes 所提供的内置类型 `kubernetes.io/ssh-auth` 用来存放 `SSH` 身份认证中 所需要的凭据。使用这种 Secret 类型时，你就必须在其 `data` （或 `stringData`） 字段中提供一个 ssh-privatekey 键值对，作为要使用的 SSH 凭据。

下面的 YAML 是一个 SSH 身份认证 Secret 的配置示例：

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # 此例中的实际数据被截断
  ssh-privatekey: |
          MIIEpQIBAAKCAQEAulqb/Y ...
```

提供 SSH 身份认证类型的 Secret 仅仅是出于用户方便性考虑。 你也可以使用 Opaque 类型来保存用于 SSH 身份认证的凭据。 不过，使用内置的 Secret 类型的有助于对凭据格式进行归一化处理，并且 API 服务器确实会检查 Secret 配置中是否提供了所需要的主键。

> **注意**： SSH 私钥自身无法建立 SSH 客户端与服务器端之间的可信连接。 需要其它方式来建立这种信任关系，以缓解“中间人（Man In The Middle）” 攻击，例如向 ConfigMap 中添加一个 known_hosts 文件。


### TLS Secret

Kubernetes 提供一种内置的 `kubernetes.io/tls` Secret 类型，用来存放证书 及其相关密钥（通常用在 TLS 场合）。 此类数据主要提供给 Ingress 资源，用以终结 TLS 链接，不过也可以用于其他 资源或者负载。当使用此类型的 Secret 时，Secret 配置中的 `data` （或 `stringData`）字段必须包含 `tls.key` 和 `tls.crt` 主键，尽管 API 服务器 实际上并不会对每个键的取值作进一步的合法性检查。

下面的 YAML 包含一个 TLS Secret 的配置示例：

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # 此例中的数据被截断
  tls.crt: |
        MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  tls.key: |
        MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
```

提供 TLS 类型的 Secret 仅仅是出于用户方便性考虑。 你也可以使用 Opaque 类型来保存用于 TLS 服务器与/或客户端的凭据。 不过，使用内置的 Secret 类型的有助于对凭据格式进行归一化处理，并且 API 服务器确实会检查 Secret 配置中是否提供了所需要的主键。

当使用 kubectl 来创建 TLS Secret 时，你可以像下面的例子一样使用 tls 子命令：

```
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert/file \
  --key=path/to/key/file
```

这里的公钥/私钥对都必须事先已存在。用于 `--cert` 的公钥证书必须是 `.PEM` 编码的 （Base64 编码的 DER 格式），且与 `--key` 所给定的私钥匹配。 私钥必须是通常所说的 PEM 私钥格式，且未加密。对这两个文件而言，PEM 格式数据 的第一行和最后一行（例如，证书所对应的 `--------BEGIN CERTIFICATE-----` 和 `-------END CERTIFICATE----`）都不会包含在其中。

### 启动引导令牌 Secret

通过将 Secret 的 `type` 设置为 `bootstrap.kubernetes.io/token` 可以创建 启动引导令牌类型的 Secret。这种类型的 Secret 被设计用来支持节点的启动引导过程。 其中包含用来为周知的 ConfigMap 签名的令牌。

启动引导令牌 Secret 通常创建于 `kube-system` 名字空间内，并以 `bootstrap-token-<令牌 ID>` 的形式命名；其中 `<令牌 ID>` 是一个由 6 个字符组成 的字符串，用作令牌的标识。


## 创建Secret
有几种不同的方式来创建 Secret：

 * [使用 kubectl 命令创建 Secret](https://kubernetes.io/zh/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
 * [使用配置文件来创建 Secret](https://kubernetes.io/zh/docs/tasks/configmap-secret/managing-secret-using-config-file/)
 * [使用 kustomize 来创建 Secret](https://kubernetes.io/zh/docs/tasks/configmap-secret/managing-secret-using-kustomize/)

## 编辑 Secret
你可以通过下面的命令编辑现有的 Secret：

```
kubectl edit secrets mysecret
```
这一命令会打开默认的编辑器，允许你更新 data 字段中包含的 base64 编码的 Secret 值

## 使用 Secret

Secret 可以作为数据卷被挂载，或作为[环境变量](https://kubernetes.io/zh/docs/concepts/containers/container-environment/) 暴露出来以供 Pod 中的容器使用。它们也可以被系统的其他部分使用，而不直接暴露在 Pod 内。 例如，它们可以保存凭据，系统的其他部分将用它来代表你与外部系统进行交互。

### 在Pod中使用Secret文件

在 Pod 中使用存放在卷中的 Secret：

1. 创建一个 Secret 或者使用已有的 Secret。多个 Pod 可以引用同一个 Secret。
2. 修改你的 Pod 定义，在 `spec.volumes[]` 下增加一个卷。可以给这个卷随意命名， 它的 `spec.volumes[].secret.secretName` 必须是 Secret 对象的名字。
3. 将 `spec.containers[].volumeMounts[]` 加到需要用到该 Secret 的容器中。 指定 `spec.containers[].volumeMounts[].readOnly = true` 和 `spec.containers[].volumeMounts[].mountPath` 为你想要该 Secret 出现的尚未使用的目录。
4. 修改你的镜像并且／或者命令行，让程序从该目录下寻找文件。 Secret 的 data 映射中的每一个键都对应 `mountPath` 下的一个文件名。

这是一个在 Pod 中使用存放在挂载卷中 Secret 的例子：

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

您想要用的每个 Secret 都需要在 `spec.volumes` 中引用。

如果 Pod 中有多个容器，每个容器都需要自己的 `volumeMounts` 配置块， 但是每个 Secret 只需要一个 `spec.volumes`。

您可以打包多个文件到一个 Secret 中，或者使用的多个 Secret，怎样方便就怎样来。


### 将 Secret 键名映射到特定路径

我们还可以控制 Secret 键名在存储卷中映射的的路径。 你可以使用 spec.volumes[].secret.items 字段修改每个键对应的目标路径：

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```

将会发生什么呢：

 * `username` Secret 存储在 `/etc/foo/my-group/my-username` 文件中而不是 `/etc/foo/username` 中。
 * `password` Secret 没有被映射
如果使用了 `spec.volumes[].secret.items`，只有在 `items` 中指定的键会被映射。 要使用 Secret 中所有键，就必须将它们都列在 `items` 字段中。 所有列出的键名必须存在于相应的 Secret 中。否则，不会创建卷

### Secret 文件权限
你还可以指定 Secret 将拥有的权限模式位。如果不指定，默认使用 0644。 你可以为整个 Secret 卷指定默认模式；如果需要，可以为每个密钥设定重载值。

例如，您可以指定如下默认模式：

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 256
```

之后，Secret 将被挂载到 /etc/foo 目录，而所有通过该 Secret 卷挂载 所创建的文件的权限都是 0400。

请注意，JSON 规范不支持八进制符号，因此使用 256 值作为 0400 权限。 如果你使用 YAML 而不是 JSON，则可以使用八进制符号以更自然的方式指定权限。

注意，如果你通过 `kubectl exec` 进入到 Pod 中，你需要沿着符号链接来找到 所期望的文件模式。例如，下面命令检查 Secret 文件的访问模式：

```
kubectl exec mypod -it sh

cd /etc/foo
ls -l
```

输出类似于：

```
total 0
lrwxrwxrwx 1 root root 15 May 18 00:18 password -> ..data/password
lrwxrwxrwx 1 root root 15 May 18 00:18 username -> ..data/username
```

### 挂载的 Secret 会被自动更新

当已经存储于卷中被使用的 Secret 被更新时，被映射的键也将终将被更新。 组件 kubelet 在周期性同步时检查被挂载的 Secret 是不是最新的。 但是，它会使用其本地缓存的数值作为 Secret 的当前值

> **Note:** 说明： 使用 Secret 作为[子路径卷](https://kubernetes.io/zh/docs/concepts/storage/volumes/#using-subpath)挂载的容器 不会收到 Secret 更新。


### 以环境变量的形式使用Secrets

将 Secret 作为 Pod 中的[环境变量](https://kubernetes.io/zh/docs/concepts/containers/container-environment/)使用：

1. 创建一个 Secret 或者使用一个已存在的 Secret。多个 Pod 可以引用同一个 Secret。
2. 修改 Pod 定义，为每个要使用 Secret 的容器添加对应 Secret 键的环境变量。 使用 Secret 键的环境变量应在 `env[x].valueFrom.secretKeyRef` 中指定 要包含的 Secret 名称和键名。
3. 更改镜像并／或者命令行，以便程序在指定的环境变量中查找值。

这是一个使用来自环境变量中的 Secret 值的 Pod 示例：

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```

#### 使用来自环境变量的 Secret 值
在一个以环境变量形式使用 Secret 的容器中，Secret 键表现为常规的环境变量，其中 包含 Secret 数据的 base-64 解码值。这是从上面的示例在容器内执行的命令的结果：

```
echo $SECRET_USERNAME
```

**`Secret 更新之后对应的环境变量不会被更新`**

如果某个容器已经在通过环境变量使用某 Secret，对该 Secret 的更新不会被 容器马上看见，除非容器被重启。有一些第三方的解决方案能够在 Secret 发生 变化时触发容器重启。

## 不可更改的 Secret

FEATURE STATE: Kubernetes v1.19 [beta]

Kubernetes 的 alpha 特性 不可变的 Secret 和 ConfigMap 提供了一种可选配置， 可以设置各个 Secret 和 ConfigMap 为不可变的。对于大量使用 Secret 的集群（至少有成千上万各不相同的 Secret 供 Pod 挂载）， 禁止变更它们的数据有下列好处：

防止意外（或非预期的）更新导致应用程序中断
通过将 Secret 标记为不可变来关闭 kube-apiserver 对其的监视，从而显著降低 kube-apiserver 的负载，提升集群性能。

这个特性通过 ImmutableEmphemeralVolumes 特性门控 来控制，从 v1.19 开始默认启用。 你可以通过将 Secret 的 immutable 字段设置为 true 创建不可更改的 Secret。 例如：

```
apiVersion: v1
kind: Secret
metadata:
  ...
data:
  ...
immutable: true
```

> **说明：**
> 一旦一个 Secret 或 ConfigMap 被标记为不可更改，撤销此操作或者更改 data 字段的内容都是 不 可能的。 只能删除并重新创建这个 Secret。现有的 Pod 将维持对已删除 Secret 的挂载点 - 建议重新创建这些 Pod。


## 使用imagePullSecret

`imagePullSecrets` 字段中包含一个列表，列举对同一名字空间中的 Secret 的引用。 你可以使用 `imagePullSecrets` 将包含 Docker（或其他）镜像仓库密码的 Secret 传递给 kubelet。kubelet 使用此信息来替你的 Pod 拉取私有镜像。 关于 `imagePullSecrets` 字段的更多信息，请参考 [PodSpec API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#podspec-v1-core) 文档。


## 使用案例

### 案例：以环境变量的形式使用 Secret

创建一个 Secret 定义：

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

生成 Secret 对象：

```
kubectl apply -f mysecret.yaml
```
使用 `envFrom` 将 Secret 的所有数据定义为容器的环境变量。 Secret 中的键名称为 Pod 中的环境变量名称：

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: mysecret
  restartPolicy: Never
```

### 案例：包含 SSH 密钥的 Pod
创建一个包含 SSH 密钥的 Secret：

```
kubectl create secret generic ssh-key-secret \
  --from-file=ssh-privatekey=/path/to/.ssh/id_rsa \
  --from-file=ssh-publickey=/path/to/.ssh/id_rsa.pub
```
输出类似于：

```
secret "ssh-key-secret" created
```
你也可以创建一个带有包含 SSH 密钥的 secretGenerator 字段的 kustomization.yaml 文件。

> **`注意：`** 发送自己的 SSH 密钥之前要仔细思考：集群的其他用户可能有权访问该密钥。 你可以使用一个服务帐户，分享给 Kubernetes 集群中合适的用户，这些用户是你要分享的。 如果服务账号遭到侵犯，可以将其收回。

现在我们可以创建一个 Pod，令其引用包含 SSH 密钥的 Secret，并通过存储卷来使用它：

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  labels:
    name: secret-test
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key-secret
  containers:
  - name: ssh-test-container
    image: mySshImage
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

容器中的命令运行时，密钥的片段可以在以下目录找到：

```
/etc/secret-volume/ssh-publickey
/etc/secret-volume/ssh-privatekey
```
然后容器可以自由使用 Secret 数据建立一个 SSH 连接

### 案例：包含生产/测试凭据的 Pod 

下面的例子展示的是两个 Pod。 一个 Pod 使用包含生产环境凭据的 Secret，另一个 Pod 使用包含测试环境凭据的 Secret。

你可以创建一个带有 `secretGenerator` 字段的 `kustomization.yaml` 文件，或者执行 `kubectl create secret`：
```
kubectl create secret generic prod-db-secret \
  --from-literal=username=produser \
  --from-literal=password=Y4nys7f11
```

```
kubectl create secret generic test-db-secret \
  --from-literal=username=testuser \
  --from-literal=password=iluvtests
```

>**说明：**
>特殊字符（例如 $、\、*、= 和 !）会被你的 Shell解释，因此需要转义。 在大多数 Shell 中，对密码进行转义的最简单方式是用单引号（'）将其括起来。 例如，如果您的实际密码是 S!B\*d$zDsb，则应通过以下方式执行命令：
>```
>kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
>```
>您无需对文件中的密码（--from-file）中的特殊字符进行转义。

创建 pod ：

``` yaml
$ cat <<EOF > pod.yaml
apiVersion: v1
kind: List
items:
- kind: Pod
  apiVersion: v1
  metadata:
    name: prod-db-client-pod
    labels:
      name: prod-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: prod-db-secret
    containers:
    - name: db-client-container
      image: myClientImage
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
- kind: Pod
  apiVersion: v1
  metadata:
    name: test-db-client-pod
    labels:
      name: test-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: test-db-secret
    containers:
    - name: db-client-container
      image: myClientImage
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
EOF
```

将 Pod 添加到同一个 kustomization.yaml 文件

```
$ cat <<EOF >> kustomization.yaml
resources:
- pod.yaml
EOF
```

通过下面的命令应用所有对象

```
kubectl apply -k .
```

两个容器都会在其文件系统上存在以下文件，其中包含容器对应的环境的值：

```
/etc/secret-volume/username
/etc/secret-volume/password
```

请注意，两个 Pod 的规约配置中仅有一个字段不同；这有助于使用共同的 Pod 配置模板创建 具有不同能力的 Pod。

您可以使用两个服务账号进一步简化基本的 Pod 规约：

1. 名为 prod-user 的服务账号拥有 prod-db-secret
2. 名为 test-user 的服务账号拥有 test-db-secret

然后，Pod 规约可以缩短为：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-db-client-pod
  labels:
    name: prod-db-client
spec:
  serviceAccount: prod-db-client
  containers:
  - name: db-client-container
    image: myClientImage
```

### 案例：Secret 卷中以句点号开头的文件

你可以通过定义以句点开头的键名，将数据“隐藏”起来。 例如，当如下 Secret 被挂载到 `secret-volume` 卷中：

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  containers:
  - name: dotfile-test-container
    image: k8s.gcr.io/busybox
    command:
    - ls
    - "-l"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

卷中将包含唯一的叫做 `.secret-file` 的文件。 容器 `dotfile-test-containe` 中，该文件处于 `/etc/secret-volume/.secret-file` 路径下。

> **说明：** 以点号开头的文件在 `ls -l` 的输出中会被隐藏起来； 列出目录内容时，必须使用 `ls -la` 才能看到它们。

### 案例：Secret 仅对 Pod 中的一个容器可见

考虑一个需要处理 HTTP 请求、执行一些复杂的业务逻辑，然后使用 HMAC 签署一些消息的应用。 因为应用程序逻辑复杂，服务器中可能会存在一个未被注意的远程文件读取漏洞， 可能会将私钥暴露给攻击者。

解决的办法可以是将应用分为两个进程，分别运行在两个容器中： 前端容器，用于处理用户交互和业务逻辑，但无法看到私钥； 签名容器，可以看到私钥，响应来自前端（例如通过本地主机网络）的简单签名请求。

使用这种分割方法，攻击者现在必须欺骗应用程序服务器才能进行任意的操作， 这可能比使其读取文件更难。

## 最佳实践
### 客户端使用 Secret API

当部署与 Secret API 交互的应用程序时，应使用 [鉴权策略](https://kubernetes.io/zh/docs/reference/access-authn-authz/authorization/)， 例如 [RBAC](https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/)，来限制访问。


## 安全属性

### 保护

因为 Secret 对象可以独立于使用它们的 Pod 而创建，所以在创建、查看和编辑 Pod 的流程中 Secret 被暴露的风险较小。系统还可以对 Secret 对象采取额外的预防性保护措施， 例如，在可能的情况下避免将其写到磁盘。

只有当某节点上的 Pod 需要用到某 Secret 时，该 Secret 才会被发送到该节点上。 Secret 不会被写入磁盘，而是被 kubelet 存储在 tmpfs 中。 一旦依赖于它的 Pod 被删除，Secret 数据的本地副本就被删除。

同一节点上的很多个 Pod 可能拥有多个 Secret。 但是，只有 Pod 所请求的 Secret 在其容器中才是可见的。 因此，一个 Pod 不能访问另一个 Pod 的 Secret。

同一个 Pod 中可能有多个容器。但是，Pod 中的每个容器必须通过 volumeeMounts 请求挂载 Secret 卷才能使卷中的 Secret 对容器可见。 这一实现可以用于在 Pod 级别[构建安全分区](https://kubernetes.io/zh/docs/concepts/configuration/secret/#secret-visible-to-only-one-container)。

在大多数 Kubernetes 发行版中，用户与 API 服务器之间的通信以及 从 API 服务器到 kubelet 的通信都受到 SSL/TLS 的保护。 通过这些通道传输时，Secret 受到保护。

FEATURE STATE: Kubernetes v1.13 [beta]

你可以为 Secret 数据开启[静态加密](https://kubernetes.io/zh/docs/tasks/administer-cluster/encrypt-data/)， 这样 Secret 数据就不会以明文形式存储到etcd 中。


## 风险

 * API 服务器上的 Secret 数据以纯文本的方式存储在 etcd 中，因此：
  - 管理员应该为集群数据开启静态加密（要求 v1.13 或者更高版本）。
  - 管理员应该限制只有 admin 用户能访问 etcd；
  - API 服务器中的 Secret 数据位于 etcd 使用的磁盘上；管理员可能希望在不再使用时擦除/粉碎 etcd 使用的磁盘
  - 如果 etcd 运行在集群内，管理员应该确保 etcd 之间的通信使用 SSL/TLS 进行加密。
 * 如果您将 Secret 数据编码为 base64 的清单（JSON 或 YAML）文件，共享该文件或将其检入代码库，该密码将会被泄露。 Base64 编码不是一种加密方式，应该视同纯文本。
 * 应用程序在从卷中读取 Secret 后仍然需要保护 Secret 的值，例如不会意外将其写入日志或发送给不信任方。
 * 可以创建使用 Secret 的 Pod 的用户也可以看到该 Secret 的值。即使 API 服务器策略不允许用户读取 Secret 对象，用户也可以运行 Pod 导致 Secret 暴露。
 * 目前，任何节点的 root 用户都可以通过模拟 kubelet 来读取 API 服务器中的任何 Secret。 仅向实际需要 Secret 的节点发送 Secret 数据才能限制节点的 root 账号漏洞的影响， 该功能还在计划中

