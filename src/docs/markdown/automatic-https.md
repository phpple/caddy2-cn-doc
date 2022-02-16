---
title: "自动HTTPS"
---

# 自动HTTPS

**Caddy是第一个也是唯一一个_默认_自动使用HTTPS的Web服务器。**

自动HTTPS为你的所有站点提供TLS证书并保持更新。它还为你将HTTP重定向到HTTPS！Caddy使用安全且现代的默认设置——无需停机、额外配置或单独的工具。

<aside class="tip">Caddy创新自动HTTPS技术；我们从2015年第一天就开始这样做了。Caddy的HTTPS自动化逻辑是世界上最成熟和最强大的。</aside>

这是一个28秒的视频，展示了它的工作原理：

<iframe width="100%" height="480" src="https://www.youtube-nocookie.com/embed/nk4EWHvvZtI?rel=0" frameborder="0" allowfullscreen=""></iframe>


**菜单：**

- [概览(overview)](#overview)
- [激活(Activation)](#activation)
- [效果(Effects)](#effects)
- [主机名要求(Hostname requirements)](#hostname-requirements)
- [本地(Local)HTTPS](#local-https)
- [测试(Testing)](#testing)
- [ACME质询(Challenges)](#acme-challenges)
- [按需(On-Demand)TLS](#on-demand-tls)
- [错误(Errors)](#errors)
- [存储(Storage)](#storage)
- [通配符证书(Wildcard certificates)](#wildcard-certificates)



## 概览(Overview)

**默认情况下，Caddy通过HTTPS为所有站点提供服务。**

- Caddy 使用本地自动信任的自签名证书（如果允许）通过 HTTPS 提供 IP 地址和本地/内部主机名。
    - 例子：`localhost`、`127.0.0.1`
- Caddy使用来自公开的ACME CA的证书，通过HTTPS提供公共DNS名称，例如[Let's Encrypt](https://letsencrypt.org)或者[ZeroSSL](https://zerossl.com).
    - 例子：`example.com`、`sub.example.com`、`*.example.com`

Caddy 会自动更新所有托管证书并将 HTTP（默认端口 80）重定向到 HTTPS（默认端口 443）。

**对于本地 HTTPS：**

- Caddy 可能会提示输入密码以将其唯一的根证书安装到你的信任库中。每个根只发生一次；你可以随时删除它。
- 任何在不信任Caddy根目录的情况下访问该站点的客户端都会显示安全错误。

**对于公共域名：**

<aside class="tip">
    这些是任何基本生产网站的常见要求，而不仅仅是Caddy。<b>主要区别是在运行 Caddy之前正确设置 DNS 记录，以便它可以提供证书。
</aside>

- 如果你的域的A/AAAA记录指向你的服务器，
- 端口80和443对外开放，
- Caddy 可以绑定到那些端口（ _或者_ 这些端口被转发到 Caddy），
- 你的[data目录](/docs/conventions#data-directory)是可写且持久的，
- 并且你的域名出现在配置中的相关位置，

然后网站将自动通过 HTTPS 提供服务。你无需为此做任何其他事情。它就是管用！

由于HTTPS使用共享的公共基础架构，因此作为服务器管理员，你应该了解此页面上的其余信息，以便避免不必要的问题，在问题发生时进行故障排除，并正确配置高级部署。


## 激活(Activation)

当Caddy知道它所服务的域名（即主机名）或IP地址时，它会隐式激活自动HTTPS。有多种方法可以告诉Caddy你的域名或IP，具体取决于你运行或配置Caddy的方式：

- [Caddyfile](/docs/caddyfile)的[站点地址](/docs/caddyfile/concepts#addresses)
- [路由](/docs/modules/http#servers/routes)中的[域名匹配器](/docs/json/apps/http/servers/routes/match/host/)
- 命令行标志，如[--domain](/docs/command-line#caddy-file-server)或[--from](/docs/command-line#caddy-reverse-proxy)
- [自动化](/docs/json/apps/tls/certificates/automate/)证书加载器

以下任何一项都将阻止全部或部分激活自动HTTPS：

- [明确禁用它](/docs/json/apps/http/servers/automatic_https/)
- 在配置中不提供任何域名或IP
- 仅监听HTTP端口
- 手动加载证书（除非[这个配置属性](/docs/json/apps/http/servers/automatic_https/ignore_loaded_certificates/)为`true`）

## 效果(Effects)

激活自动HTTPS后，会发生以下情况：

- [所有域名](#hostname-requirements)的证书会被获取和更新
- 默认端口（如果有）会被更改为[HTTPS端口](/docs/modules/http#https_port)`443`
- HTTP（使用[HTTP端口](/docs/modules/http#http_port)`80`）会被重定向到HTTPS

自动HTTPS永远不会覆盖显式配置。

如有必要，你可以[自定义或禁用HTTPS](/docs/json/apps/http/servers/automatic_https/)；例如，你可以跳过某些域名或禁用重定向（对于 Caddyfile，请使用[全局选项](/docs/caddyfile/options)执行该操作）。


## 主机名要求(Hostname requirements)

如果所有主机名（域名）符合以下条件，则它们都有资格获得完全托管的证书：

- 非空
- 仅由字母数字、连字符、点和通配符（`*`）组成
- 不要以点开头或结尾（[RFC 1034](https://tools.ietf.org/html/rfc1034#section-3.5)）

此外，如果主机名符合以下条件，则它们有资格获得公开信任的证书：

- 不是本地主机（包括`.localhost`和`.local`TLD）
- 不是IP地址
- 只有一个通配符`*`作为最左边的标签


## 本地(Local) HTTPS

为了通过 HTTPS 为非公共站点提供服务，Caddy 生成自己的证书颁发机构 (CA) 并使用它来签署证书。信任链由根证书和中间证书组成。叶证书由中间人签名。它们存储在[Caddy的数据目录](/docs/conventions#data-directory)中的`pki/authorities/local`。

Caddy 的本地CA由[Smallstep库](https://smallstep.com/certificates/)提供支持。

本地HTTPS不使用ACME，也不执行任何DNS验证。它仅在本地机器上工作，并且仅在安装了CA的根证书的地方才受信任。

### CA根(Root)

根的私钥是使用加密安全的伪随机源唯一生成的，并以有限的权限保存到存储中。它仅被加载到内存中以执行签名任务，之后它留下了垃圾收集的范围。

虽然可以将Caddy配置为直接使用root签名（以支持不兼容的客户端），但默认情况此功能被禁用，且root密钥仅用于签署中间体。

第一次使用根密钥时，Caddy将尝试将其安装到系统的本地信任库中。如果它没有权限这样做，它会提示输入密码。如果不需要，可以在配置中禁用此行为。

<aside class="tip">
只要你的计算机没有受到威胁并且你的唯一根密钥没有泄露，那么在你自己的计算机上信任 Caddy 的根证书是安全的。
</aside>

安装 Caddy 的根 CA 后，你将在本地信任存储中看到它作为“Caddy Local Authority”（除非你配置了不同的名称）。如果你愿意，你可以随时卸载它（该[`caddy untrust`](/docs/command-line#caddy-untrust)命令使这很容易）。

请注意，自动将证书安装到本地信任存储中只是为了方便，不能保证能正常工作，尤其是在使用容器或Caddy作为非特权系统服务运行的情况下。最后，如果你依赖内部PKI，系统管理员有责任确保将Caddy的根CA正确添加到必要的信任存储中（这超出了Web服务器的范围）。

### CA中间体(Intermediates)

中间证书和密钥也将被生成，用于签署叶(leaf)（单个站点）证书。

与根证书不同，中间证书的生命周期要短得多，并且会根据需要自动更新。


## 测试(Testing)

要测试或试验你的 Caddy 配置，请确保[将ACME端点更改](/docs/modules/tls.issuance.acme#ca)为暂存或开发URL，否则你可能会达到速率限制，这可能会阻止你访问HTTPS长达一周，具体取决于你遇到的速率限制。

Caddy的默认CA 之一是[Let's Encrypt](https://letsencrypt.org/)，它有一个[暂存端点](https://letsencrypt.org/docs/staging-environment/)不被相同的[速率限制](https://letsencrypt.org/docs/rate-limits/)影响：

```
https://acme-staging-v02.api.letsencrypt.org/directory
```

## ACME质询(challenges)

获取公众信任的 TLS 证书需要来自公众信任的第三方机构的验证。如今，此验证过程通过[ACME协议](https://tools.ietf.org/html/rfc8555)自动执行，并且可以执行以下三种方式之一（“质询类型”），如下所述。

前两种质询类型默认启用。如果启用了多种质询，Caddy会随机选择一种，以避免依赖特定质询带来的意外。随着时间的推移，它会了解哪种质询类型最成功，并会开始倾向于选取它，但如果有必要，它还是会重新选取其他可用的质询类型。

### HTTP质询(challenge)

HTTP质询对候选主机名的A/AAAA记录执行权威DNS查找，然后使用HTTP通过端口80请求临时加密资源。如果CA看到预期的资源，则会颁发证书。

此质询要求端口80可从外部访问。如果Caddy无法监听80端口，则必须将来自80端口的数据包转发到Caddy的[HTTP端口](/docs/json/apps/http/http_port/)。

默认情况下启用此质询，不需要显式配置。


### TLS-ALPN质询(challenge)

TLS-ALPN 质询对候选主机名的A/AAAA记录执行权威DNS查找，然后使用包含特殊ServerName和ALPN值的TLS握手通过端口443请求临时加密资源。如果CA看到预期的资源，则会颁发证书。

此种质询要求端口443可从外部访问。如果Caddy无法监听443端口，则必须将来自443端口的数据包转发到Caddy的[HTTPS端口](/docs/json/apps/http/https_port/)。

默认情况下启用此质询，不需要显式配置。


### DNS质询(challenge)

DNS质询对候选主机名的TX 记录执行权威DNS查找，并查找具有特定值的特殊TXT记录。如果CA看到预期值，则颁发证书。

此质询不需要任何开放端口，并且请求证书的服务器不需要可从外部访问。但是，DNS 质询需要配置。Caddy 需要知道访问你域的 DNS 提供商的凭据，以便它可以设置（和清除）特殊的 TXT 记录。如果启用 DNS 质询，则默认禁用其他质询。

DNS提供商支持是一项社区工作。[在我们的wiki上了解如何为你的提供商启用DNS质询](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)。


## 按需(On-Demand) TLS

Caddy开创了一种我们称为**按需(On-Demand) TLS**的新技术，它在需要它的第一次TLS握手期间动态获取新证书，而不是在配置加载时。至关重要的是，这不需要提前在配置中指定域名。

许多企业依靠这一独特的功能以更低的成本扩展他们的 TLS 部署，并且在为数万个站点提供服务时不会遇到运营难题。

按需 TLS 在以下情况下很有用：

- 当你启动或重新加载服务器时，你并不知道所有域名，
- 域名可能未立即正确配置（尚未设置 DNS 记录），
- 你无法控制域名（例如，它们是客户的域名）。

启用按需 TLS 后，你无需在配置中指定域名即可为其获取证书。相反，当收到 Caddy 尚未获得证书的服务器名称 (SNI) 的 TLS 握手时，会在 Caddy 获得用于完成握手的证书时保持握手。延迟通常只有几秒钟，只有最初的握手很慢。所有未来的握手都很快，因为证书被缓存和重用，并且更新发生在后台。未来的握手可能会触发对证书的维护以使其保持更新，但如果证书尚未过期，则此维护将在后台进行。

### 使用按需(Using On-Demand) TLS

**在生产环境中，必须同时启用和限制按需TLS。在不限制的情况下启用会使你的服务器受到攻击。**

如果使用 JSON 配置，则在[TLS自动化策略](/docs/json/apps/tls/automation/policies/)中启用按需TLS，如果使用Caddyfile，则在带有[带有`tls`指令的站点块](/docs/caddyfile/directives/tls)中启用它。

为防止滥用此功能，你必须配置限制。这是在[`automation`JSON配置对象](/docs/json/apps/tls/automation/on_demand/)或Caddyfile的[`on_demand_tls`全局选项](/docs/caddyfile/options#on-demand-tls)中完成的。限制是“全局的”，不能按站点或按域进行配置。主要限制是“询问”端点，Caddy 将向其发送 HTTP 请求以询问它是否有权在握手中获取和管理域的证书。这意味着你将需要一些内部后端，例如，可以查询数据库的帐户表并查看客户是否已使用该域名注册。

你还可以将速率限制配置为限制，但仅速率限制不足以提供保护。

请注意你的 CA 能够以多快的速度颁发证书。如果花费的时间超过几秒钟，这将对用户体验产生负面影响（仅适用于第一个客户端）。

由于它的延迟性质和滥用的可能性（如果没有通过适当的配置来缓解），我们建议仅在你的实际用例在上面描述时才启用按需 TLS。

[有关有效使用按需TLS的更多信息，请参阅我们的wiki文章](https://caddy.community/t/serving-tens-of-thousands-of-domains-over-https-with-caddy/11179)。

## 错误(Errors)

如果证书管理出现错误，Caddy会尽力继续。

默认情况下，证书管理在后台执行。这意味着它不会阻止启动或减慢你的网站。但是，这也意味着服务器将在所有证书可用之前运行。在后台运行允许Caddy在很长一段时间内以指数退避重试。

如果在获取或更新证书时出错，会发生以下情况：

1. Caddy在短暂暂停后重试一次，以防万一这是偶然事件
2. Caddy短暂停顿，然后切换到下一个启用的质询类型
3. 在尝试了所有启用的质询类型后，[它会尝试下一个配置的颁发者](#issuer-fallback)
    - Let's Encrypt
    - ZeroSSL
4. 在尝试了所有发行人之后，它会以指数方式回退
    - 尝试之间最多间隔 1 天
    - 长达30天

在使用Let's Encrypt重试期间，Caddy 切换到他们的[暂存环境](https://letsencrypt.org/docs/staging-environment/)以避免速率限制问题。这不是一个完美的策略，但总的来说它很有帮助。

ACME质询至少需要几秒钟才能完成，内部速率限制有助于减少意外滥用。除了你或CA配置的内容之外，Caddy还使用内部速率限制，这样你就可以给Caddy一个包含一百万个域名的配置(platter)，它会逐渐（但尽可能快地）获得所有域名的证书。Caddy的内部速率限制目前是每个ACME帐户每10秒尝试10次。

为避免资源泄漏，Caddy会在配置更改时中止正在进行的任务（包括ACME事务）。虽然Caddy能够处理频繁的配置重新加载，但请注意诸如此类的操作注意事项，并考虑批量配置更改以减少重新加载并让Caddy有机会在后台实际完成获取证书。

### 发行人后备(Issuer fallback)

Caddy 是第一个（也是迄今为止唯一一个）在无法成功获得证书时支持完全冗余、自动故障转移到其他CA的服务器。

默认情况下，Caddy启用两个与ACME兼容的CA：[**Let's Encrypt**](https://letsencrypt.org)和[**ZeroSSL**](https://zerossl.com)。如果Caddy无法从Let's Encrypt获得证书，它将尝试使用ZeroSSL；如果两者都失败，它将退避并稍后重试。在你的配置中，你可以自定义Caddy使用哪些颁发者来获取证书，可以是通用的，也可以是特定名称的。

## 存储(Storage)

Caddy 将在其[配置的存储设施](/docs/json/storage/)中存储公共证书、私钥和其他资产（或默认存储设施，如果未配置 - 请参阅链接了解详细信息）。

**使用默认配置需要了解的主要内容是该`$HOME`文件夹必须是可写且持久的。`--environ`的作用是帮助你排除故障，如果指定了标志，Caddy会在启动时打印该环境变量。

任何配置为使用相同存储的Caddy实例都将自动共享这些资源并作为集群协调证书管理。

在尝试任何 ACME 事务之前，Caddy 将测试配置的存储以确保它是可写的并且有足够的容量。这有助于减少不必要的锁争用。

## 通配符证书(Wildcard certificates)

当 Caddy 配置为使用合格的通配符名称为站点提供服务时，它可以获取和管理通配符证书。如果只有最左边的域标签是通配符，则站点名称有资格使用通配符。例如，`*.example.com`符合条件，但这些不符合条件：`sub.*.example.com`、`foo*.example.com`、`*bar.example.com`和`*.*.example.com`。

如果使用 Caddyfile，Caddy 会根据证书主题名称获取站点名称。换句话说，定义为的站点`sub.example.com`将导致 Caddy管理`sub.example.com`的证书, ，定义为的站点`*.example.com`将导致Caddy管理`*.example.com`的通配符证书。你可以在我们的[常用Caddyfile模式](/docs/caddyfile/patterns#wildcard-certificates)页面上看到这一点。如果你需要不同的行为，JSON 配置可以让你更精确地控制证书主题和站点名称（“主机匹配器”）。

通配符证书代表了广泛的权限，并且仅应在你拥有如此多的子域以致为它们管理单个证书会使 PKI 紧张或导致你达到 CA 强制的速率限制时使用。

**注意：** [Let's Encrypt](https://letsencrypt.org/docs/challenge-types/)需要使用[DNS质询](#dns-challenge)才能获得通配符证书。