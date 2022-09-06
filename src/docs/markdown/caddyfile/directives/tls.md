---
title: tls (Caddyfile指令)
---

为网站配置TLS。

**Caddy的默认TLS设置是安全的。只有在你有充分的理由并了解其影响的情况下才能改变这些设置。**这个指令最常见的用途是指定一个ACME账户的电子邮件地址，改变ACME CA的端点，或者提供你自己的证书。

兼容性说明：由于其作为安全协议的敏感性质，在新的次要版本或补丁版本中可能会对TLS默认值进行有意的调整。旧的或坏的TLS版本、密码、功能等可能在任何时候被删除。如果你的部署对变化非常敏感，你应该明确指定那些必须保持不变的值，并对升级保持警惕。在几乎所有情况下，我们建议使用默认设置。


## 语法

```caddy-d
tls [internal|<email>] | [<cert_file> <key_file>] {
	protocols <min> [<max>]
	ciphers   <cipher_suites...>
	curves    <curves...>
	alpn      <values...>
	load      <paths...>
	ca        <ca_dir_url>
	ca_root   <pem_file>
	key_type  ed25519|p256|p384|rsa2048|rsa4096
	dns       <provider_name> [<params...>]
	dns_challenge_override_domain <domain>
	resolvers <dns_servers...>
	eab       <key_id> <mac_key>
	on_demand
	client_auth {
		mode                   [request|require|verify_if_given|require_and_verify]
		trusted_ca_cert        <base64_der>
		trusted_ca_cert_file   <filename>
		trusted_leaf_cert      <base64_der>
		trusted_leaf_cert_file <filename>
	}
	issuer          <issuer_name>  [<params...>]
	get_certificate <manager_name> [<params...>]
}
```

- **internal**意味着使用Caddy内部的、本地信任的CA来为这个网站制作证书。要进一步配置[`internal`](#internal)发行者，请使用[`issuer`](#issuer)子指令。
- **&lt;email&gt;**是用于管理网站证书的ACME账户的电子邮件地址。
- **&lt;cert_file&gt;**和**&lt;key_file&gt;**是证书和私钥PEM文件的路径。只指定一个是无效的。
- **protocols** <span id="protocols"/>指定最小和最大协议版本。默认最小：`tls1.2`。默认最大。`tls1.3'.
- **ciphers** <span id="ciphers"/>指定了以降序排列的密码套件名称列表。建议不要改变这些，除非你知道你在做什么。请注意，对于TLS 1.3来说，密码套件是不可定制的；而且并非所有的TLS 1.2密码都是默认启用的。支持的名称是（这里没有特定的顺序）。
    - tls_rsa_with_3des_ede_cbc_sha
    - tls_rsa_with_aes_128_cbc_sha
    - 含有256个字母的ls_rsa_aes_cbc_sha的ls_rsa_sha
    - TLS_RSA_WITH_AES_128_GCM_SHA256
    - TLS_RSA_WITH_AES_256_GCM_SHA384
    - 含有AES_128_GCM_SHA256的TLS_AES_128_GCM_SHA384
    - TLS_AES_256_GCM_SHA384
    - tls_chacha20_poly1305_sha256
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
    - TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
    - 含有3des_ede_cbc_sha的Tls_ecdhe_rsa。
    - TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
    - TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
- **curves** <span id="curves"/> 指定支持EC曲线的列表。建议不要改变这些。支持的值是。
    - x25519
    - secp256r1
    - secp384r1
    - secp521r1
- **alpn** <span id="alpn"/>是在TLS握手的ALPN扩展中公布的值的列表。
- **load** <span id="load"/>指定一个文件夹列表，从该文件夹中加载证书+密钥捆绑的PEM文件。
- **ca** <span id="ca"/>改变ACME CA端点。这最常用于测试时设置[Let's Encrypt's staging endpoint](https://letsencrypt.org/docs/staging-environment/)，或内部ACME服务器。(要为整个Caddyfile改变这个值，请使用`acme_ca`[全局选项](/docs/caddyfile/options)代替。)
- **ca_root** <span id="ca_root"/>指定一个PEM文件，其中包含ACME CA端点的可信根证书，如果不在系统信任存储中。
- **key_type** <span id="key_type"/>是生成CSR时要使用的密钥类型。只有在你有特定要求时才设置这个。

- **dns* <span id="dns"/>使用指定的提供者插件启用[DNS挑战](/docs/automatic-https#dns-challenge)，该插件必须从[`caddy-dns`](https://github.com/caddy-dns) 仓库中插入。每个提供者插件的名字后面可能有他们自己的语法，详情请参考他们的文档。维护对每个DNS提供商的支持是一项社区工作。[了解如何在我们的维基上为你的提供商启用DNS挑战。](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)
- **dns_challenge_override_domain** <span id="dns_challenge_override_domain"/> 覆盖用于DNS挑战的域名。这是为了将挑战委托给一个不同的域，例如，其DNS提供商有一个[`caddy-dns`](https://github.com/caddy-dns)插件。
- **resolvers** <span id="resolvers"/>自定义执行DNS挑战时使用的DNS解析器；这些解析器优先于系统解析器或任何默认的解析器。如果在这里设置，解析器将传播到所有配置的证书颁发者。
- **eab** <span id="eab"/>为这个站点配置ACME外部账户绑定（EAB），使用你的CA提供的密钥ID和MAC密钥。
- **on_demand** <span id="on_demand"/>为网站块地址中给出的主机名启用[on-demand TLS]（/docs/automatic-https#on-demand-tls）。**安全警告：**在生产中这样做是不安全的，除非你也配置了[`on_demand_tls`全局选项](https://caddyserver.com/docs/caddyfile/options#on-demand-tls)以减少滥用。
- **client_auth** <span id="client_auth"/>启用并配置TLS客户端认证。
    - **mode** <span id="mode"/>是验证客户端的模式。允许的值是。

      | 模式 | 说明 |
      |--------------------|------------------------------------------------------------------------------------------|
      | request | 要求客户提供证书，但即使没有证书也允许；不对其进行验证。|
      | require | 要求客户出示证书，但不进行验证。|
      | verify_if_given | 要求客户提供证书，即使没有也允许，但如果有则验证。|
      | require_and_verify | 要求客户出示经过验证的有效证书。|

  默认值: `require_and_verify`如果提供了任何`trusted_ca_cert`或`trusted_leaf_cert`；否则，`require`。

    - **trusted_ca_cert** <span id="trusted_ca_cert"/>是一个base64 DER编码的CA证书，用于验证客户证书。
    - **trusted_ca_cert_file** <span id="trusted_ca_cert_file"/> 是一个 PEM CA 证书文件的路径，用来验证客户证书。
    - **trusted_leaf_cert** <span id="trusted_leaf_cert"/> 是一个接受base64 DER编码的客户叶子证书。
    - **trusted_leaf_cert_file** <span id="trusted_leaf_cert_file"/> 是一个 PEM CA 证书文件的路径，用来验证客户证书。

  多个`trusted_*`指令可用于指定多个CA或叶子证书。未被列为叶子证书之一或未被任何指定CA签署的客户证书将根据**模式**被拒绝。

- **issuer** <span id="issuer"/>配置一个自定义的证书颁发者，或一个获得证书的来源。使用哪一个签发者以及这一段后面的选项取决于可用的签发者模块（标准签发者见下文；插件可以添加其他的）。其他一些子指令，如`ca`和`dns`实际上是配置`acme`签发者的快捷方式（这个子指令是后来添加的），所以指定这个指令和其他一些指令是混乱的，因此禁止使用。这个子指令可以被多次指定，以配置多个冗余的签发者；如果一个签发失败，将尝试下一个签发者。
- **get_certificate** <span id="get_certificate"/>可以在握手时从一个_管理模块_获取证书。[关于标准的证书管理器模块，见下文。](#certificate-managers)

### 发行人

这些发行商都有标准的 "tls "指令。

#### acme

使用ACME协议获取证书。

```caddy
... acme [<directory_url>] {
	dir      <directory_url>
	test_dir <test_directory_url>
	email    <email>
	timeout  <duration>
	disable_http_challenge
	disable_tlsalpn_challenge
	alt_http_port    <port>
	alt_tlsalpn_port <port>
	eab <key_id> <mac_key>
	trusted_roots <pem_files...>
	dns <provider_name> [<options>]
	propagation_timeout <duration>
	propagation_delay <duration>
	resolvers <dns_servers...>
	preferred_chains [smallest] {
		root_common_name <common_names...>
		any_common_name  <common_names...>
	}
}
```

- **dir** <span id="dir"/>是ACME CA目录的URL。默认：`https://acme-v02.api.letsencrypt.org/directory`。
- **test_dir** <span id="test_dir"/>是一个可选的后备目录，在重试挑战时使用；如果所有挑战失败，在重试时将使用该端点；如果CA有一个暂存端点，你想避免其生产端点的速率限制，则非常有用。默认：`https://acme-staging-v02.api.letsencrypt.org/directory`。
- **email** <span id="email"/>是ACME帐户的联系电子邮件地址。
- **timeout** <span id="timeout"/>是一个[持续时间值](/docs/conventions#durations)，设置ACME操作超时前要等待多长时间。
- **disable_http_challenge** <span id="disable_http_challenge"/>将禁用HTTP挑战。
- **disable_tlsalpn_challenge** <span id="disable_tlsalpn_challenge"/> 将禁用TLS-ALPN挑战。
- **alt_http_port** <span id="alt_http_port"/>是提供HTTP挑战的备用端口；它必须发生在80端口，所以你必须将数据包转发到这个备用端口。
- **alt_tlsalpn_port** <span id="alt_tlsalpn_port"/>是一个备用端口，用于提供TLS-ALPN挑战；它必须发生在443端口，所以你必须将数据包转发到这个备用端口。
- eab** <span id="eab"/> 指定一个外部账户绑定，这在某些ACME CA中可能是必需的。
- **trusted_roots** <span id="trusted_roots"/>是一个或多个根证书（作为PEM文件名），当连接到ACME CA服务器时要信任。
- **dns** <span id="dns"/>配置了DNS挑战。
- **propagation_timeout** <span id="propagation_timeout"/>是一个[持续时间值]（/docs/conventions#durations），设置使用DNS挑战时等待DNS TXT记录出现的最长时间。设置为`-1`以禁用传播检查。默认为2分钟。
- **propagation_delay** <span id="propagation_delay"/>是一个[持续时间值](/docs/conventions#durations)，设置使用DNS挑战时，在开始DNS TXT记录传播检查之前要等待多长时间。默认为0（不等待）。
- **resolvers** <span id="resolvers"/>自定义执行DNS挑战时使用的DNS解析器；这些解析器优先于系统解析器或任何默认的解析器。
- **preferred_chains** <span id="preferred_chains"/> 指定Caddy应该偏爱哪些证书链；如果你的CA提供了多个证书链，则很有用。使用以下选项之一。
	- **smallest** <span id="smallest"/>将告诉Caddy倾向于使用字节数最少的链。
	- **root_common_name** <span id="root_common_name"/>是一个或多个通用名称的列表；Caddy将选择第一个根部与指定通用名称中至少一个相匹配的链。
	- **any_common_name** <span id="any_common_name"/>是一个或多个通用名称的列表；Caddy将选择第一个具有与指定通用名称中至少一个相匹配的发行者的链。

#### zerossl

使用ACME协议获取证书，特别是使用ZeroSSL。

```caddy
... zerossl [<api_key>] {
	...
}
```

`zerossl`的语法与`acme`的语法完全相同，只是它的名字是`zerossl`，而且它可以选择使用你的ZeroSSL API密钥。

`zerossl`发行器的功能与`acme`发行器相同，只是它默认使用ZeroSSL的目录，并且可以自动协商EAB凭证（而对于`acme`发行器，你必须手动提供EAB凭证并设置目录端点）。

当明确配置`zerossl`时，需要一个电子邮件地址，以便你的证书能够出现在你的ZeroSSL仪表板上。

请注意，ZeroSSL是一个默认的签发者，所以明确配置它通常是不必要的。

#### 内部

从一个内部证书颁发机构获取证书。

```caddy
... internal {
	ca       <name>
	lifetime <duration>
	sign_with_root
}
```

- **ca** <span id="ca"/>是要使用的内部CA的名称。默认值：`local'。参见[PKI应用程序全局选项](/docs/caddyfile/options#pki-options)以配置替代的CA。
- **lifetime** <span id="lifetime"/>是一个[持续时间值](/docs/conventions#durations)，设置内部颁发的叶子证书的有效期。默认值：12h。除非绝对必要，否则不建议改变这个。
- **sign_with_root** <span id="sign_with_root"/> 迫使根节点成为签发者而不是中间节点。不建议这样做，只应在设备/客户不能正确验证证书链的情况下使用（非常不常见）。



###证书管理器

证书管理器模块与签发者模块不同，使用管理器模块意味着外部工具或服务保持证书的更新，而签发者模块则意味着Caddy本身在管理证书。(发行人模块接受证书签名请求（CSR）作为输入，但证书管理器模块接受TLS ClientHello作为输入）。

这些管理模块的标准配置是`tls`指令：

#### tailscale

从本地运行的[Tailscale](https://tailscale.com)实例获取证书。[HTTPS必须在你的Tailscale账户中启用](https://tailscale.com/kb/1153/enabling-https/)（或你的开源[Headscale服务器](https://github.com/juanfont/headscale)）；而且Caddy进程必须以root身份运行，或者你必须配置`tailscaled`给你的Caddy用户[获取证书的权限](https://github.com/caddyserver/caddy/pull/4541#issuecomment-1021568348)。

_**注意：这通常是不必要的！** Caddy对所有`*.ts.net`域名自动使用Tailscale，不需要任何额外配置。

```caddy-d
get_certificate tailscale  # often unnecessary!
```


#### http

通过发出HTTP(S)请求来获取证书。响应必须有200状态代码，正文必须包含一个PEM链，包括完整的证书（含中介）以及私钥。

```caddy-d
get_certificate http <url>
```

- **url** <span id="url"/>是进行请求的全称URL。出于性能原因，强烈建议这是一个本地端点。该URL将被添加以下查询字符串参数。`server_name` = SNI值，`signature_schemes` = 签名算法的十六进制ID的逗号分隔列表，`cipher_suites` = 密码套件的十六进制IDS的逗号分隔列表。


## 示例

使用自定义的证书和密钥：

```caddy-d
tls cert.pem key.pem
```

为当前网站块上的所有主机使用本地信任的证书，而不是通过ACME / Let's Encrypt的公共证书（在开发环境中很有用）：

```caddy-d
tls internal
```

使用本地信任的证书，但在后台按需管理，而不是在后台：

```caddy-d
tls internal {
	on_demand
}
```

对内部CA使用自定义选项（不能使用`tls internal`的快捷方式）：

```caddy-d
tls {
	issuer internal {
		ca foo
	}
}
```

为你的ACME账户指定一个电子邮件地址（但如果所有网站只使用一个电子邮件，我们建议用`email`[全局选项](/docs/caddyfile/options)代替）：

```caddy-d
tls your@email.com
```

为Cloudflare上管理的域名启用DNS挑战，在环境变量中使用账户凭证：

```caddy-d
tls {
	dns cloudflare {env.CLOUDFLARE_API_TOKEN}
}
```

通过HTTP获取证书链，而不是让Caddy管理它：

```caddy-d
tls {
	get_certificate http http://localhost:9007/certs
}
```

启用TLS客户认证，并要求客户出示有效的证书，通过`trusted_ca_cert_file`对所有提供的CA进行验证：

```caddy-d
tls {
	client_auth {
		mode                 require_and_verify
		trusted_ca_cert_file ../caddy.ca.cer
		trusted_ca_cert_file ../root.ca.cer
	}
}
```
