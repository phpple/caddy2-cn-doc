---
title: 全局选项
---

<script>
$(function() {
	// We'll add links on the options in the code block at the top
	// to their associated anchor tags.
	let headers = $('article h5').map((i, el) => el.id.replace(/-/g, "_")).toArray();
	$('pre.chroma .k')
		.filter((k, item) => headers.includes(item.innerText))
		.map(function(k, item) {
			let text = item.innerText.replace(/</g,'&lt;').replace(/>/g,'&gt;');
			let url = '#' + item.innerText.replace(/_/g, "-");
			$(item).html('<a href="' + url + '" style="color: inherit;" title="' + text + '">' + text + '</a>');
		});
	$('pre.chroma .k:contains("servers")')
		.map(function(k, item) {
			let text = item.innerText.replace(/</g,'&lt;').replace(/>/g,'&gt;');
			$(item).html('<a href="#server-options" style="color: inherit;" title="Server Options">' + text + '</a>');
		});
});
</script>

# 全局选项

Caddyfile可以让你指定全局应用的选项。一些选项充当默认值，而其他选项自定义Caddyfile[适配器](/docs/config-adapters)的行为。

Caddyfile的最顶部可以是**全局选项块**。这是一个没有键的块：

```caddy
{
	...
}
```

最多只能有一个，而且必须是Caddyfile的第一个块。

可能的选项是：

```caddy
{
	# General Options
	debug
	http_port  <port>
	https_port <port>
	order <dir1> first|last|[before|after <dir2>]
	storage <module_name> {
		<options...>
	}
	storage_clean_interval <duration>
	admin   off|<addr> {
		origins <origins...>
		enforce_origin
	}
	log [name] {
		output  <writer_module> ...
		format  <encoder_module> ...
		level   <level>
		include <namespaces...>
		exclude <namespaces...>
	}
	grace_period <duration>

	# TLS Options
	auto_https off|disable_redirects|ignore_loaded_certs
	email <yours>
	default_sni <name>
	local_certs
	skip_install_trust
	acme_ca <directory_url>
	acme_ca_root <pem_file>
	acme_eab <key_id> <mac_key>
	acme_dns <provider> ...
	on_demand_tls {
		ask      <endpoint>
		interval <duration>
		burst    <n>
	}
	key_type ed25519|p256|p384|rsa2048|rsa4096
	cert_issuer <name> ...
	ocsp_stapling off
	preferred_chains [smallest] {
		root_common_name <common_names...>
		any_common_name  <common_names...>
	}

	# Server Options
	servers [<listener_address>] {
		listener_wrappers {
			<listener_wrappers...>
		}
		timeouts {
			read_body   <duration>
			read_header <duration>
			write       <duration>
			idle        <duration>
		}
		max_header_size <size>
		protocol {
			allow_h2c
			experimental_http3
			strict_sni_host
		}
	}
}
```


## 常规选项

##### `debug`

启用调试模式，将默认日至器的日志级别设置为`DEBUG`。这将展示在排除故障时展示更多有用的细节（且在生产模式下将非常详细）。请你在[社区](https://caddy.community)寻求帮助之前先启用此功能。


##### `http_port`
服务器用于HTTP的端口。仅限内部使用; 不会更改客户端的HTTP端口。默认：`80`


##### `https_port`

服务器用于HTTPS的端口。仅限内部使用; 不会更改客户端的HTTPS端口。默认：`443`


##### `order`
对HTTP处理指令进行排序。由于HTTP处理器按照顺序链执行，因此处理程序必须以正确的顺序执行。标准指令有[预定义顺序](/docs/caddyfile/directives#directive-order)，但如果使用第三方 HTTP 处理程序模块，则需要通过使用此选项或将指令放在

例如，要使用[`replace-response`插件](https://github.com/caddyserver/replace-response)，而且你希望它在`encode`之后才被执行，以便它可以在响应编码之前进行。（因为响应流向上处理程序链，而不是向下）：

```caddy-d
order replace after encode
```


##### `storage`

配置Caddy的存储机制。默认值为 [`file_system`](/docs/json/storage/file_system/)。还有许多其他可用的存储模块作为插件提供。

例如，要更改文件系统的存储位置：

```caddy-d
storage file_system /path/to/custom/location
```

在跨多个Caddy实例同步Caddy的存储时，通常需要自定义存储模块，以确保它们都使用相同的证书和密钥。有关更多详细信息，请参阅[有关存储的自动HTTPS]部分(/docs/automatic-https#storage)。


##### `storage_clean_interval`

多久扫描一次存储单元以查找旧资源或过期资源并将其移除。这些扫描会在存储模块上进行大量读取（和列表操作），因此对于大型部署需要设置成更长的时间间隔。该值是一个持续时间值。默认值：24小时。

该过程首次启动时，总是会清理存储。然后，如果上次清理在该间隔的一半时间内完成，则在上次清理开始后的这段时间内将开始新的清洁（否则将跳过下次启动）。

##### `admin`

自定义[管理API端点](/docs/api)。如果为`off`，则管理端点将被禁用。如果禁用，则在不停止和启动服务器的情况下将无法更改配置。

- **origins** 配置允许连接到端点的远程/源列表。

- **enforce_origin** 启用Origin标头的强制执行。（这与通常强制执行起源不同，后者总是执行。）


##### `log`

自定义命名日志器。可以传递名称以指示要为其自定义行为的特定日志器。如果未指定名称，则修改默认日志器的行为。可以多次指定此选项以配置不同的日志器。你可以在[日志记录文档](/docs/logging)中阅读有关默认日志器和其他日志记录行为的更多信息。

- **output** 配置写入日志的位置。有关更多信息，请参阅[log指令](/docs/caddyfile/directives/log#output-modules)文档，该文档具有相同的结构。
- **format** 描述如何编码或格式化日志。有关更多信息，请参阅[log指令](/docs/caddyfile/directives/log#format-modules)文档，该文档具有相同的结构。
- **level** 是要记录的最低入口级别。默认：`INFO`
- **include** 标识此日志配置中包含的记录器。有关更多信息，请参阅[JSON文档](/docs/json/logging/logs/include/)。
- **exclude** 标识从该日志配置中排除的记录器。有关更多信息，请参阅[JSON文档](/docs/json/logging/logs/exclude/)。


##### `grace_period`

定义在配置重新加载期间关闭HTTP服务器的宽限期。如果客户端没有在宽限期内完成他们的请求，服务器将被强制终止以允许重新加载完成并释放资源。默认情况下，没有设置宽限期。

## TLS选项

##### `auto_https`

配置自动HTTPS。它可以完全禁用(`off`)，仅禁用HTTP到HTTPS的重定向(`disable_redirects`)，或者配置为自动生成证书，即使是出现在手动加载的证书上的名称(`ignore_loaded_certs`)。有关更多详细信息，请参阅[自动HTTPS](/docs/automatic-https) 页面。


##### `email`

你的电子邮件地址。主要在使用 CA 创建 ACME 帐户时使用，强烈建议在证书出现问题时使用。

##### `default_sni`

当客户端在其`ClientHello`中不使用SNI时，设置默认TLS服务器名称(TLS ServerName)。

##### `local_certs`

导致所有证书默认在内部颁发，而不是通过（公共）ACME CA，例如Let's Encrypt。这在开发环境中很有用。

##### `skip_install_trust`

跳过将本地CA的根安装到系统信任库以及Java和Mozilla Firefox信任库的尝试。

##### `acme_ca`

指定ACME CA目录的URL。强烈建议将此设置为Let's Encrypt的[暂存端点](https://letsencrypt.org/docs/staging-environment/)以进行测试或开发。默认值：ZeroSSL和Let's Encrypt 的生产端点。

##### `acme_ca_root`

指定一个PEM文件，其中包含ACME CA端点的受信任根证书（如果不在系统信任库中）。

##### `acme_eab`

指定用于所有 ACME 交易的外部帐户绑定(External Account Binding)。

##### `acme_dns`

配置用于所有ACME事务的ACME DNS质询提供者。在该标识后设置的提供者名字指定了提供者，就像在[`tls`指令的`acme`问题](/docs/caddyfile/directives/tls#acme)中指定的一样。

##### `on_demand_tls`

在启用的地方配置[按需TLS](/docs/automatic-https#on-demand-tls)，但不启用它（要启用它，请使用[按需`tls`子指令](/docs/caddyfile/directives/tls#syntax)）。强烈建议只在生产环境中使用，以防止滥用。

- **ask** 将导致Caddy使用包含域名值的查询字符串`?domain=`向给定URL发出HTTP请求。如果端点返回 200 OK，Caddy 将被授权获取该名称的证书。
- **interval**和**burst** 允许在 `<duration>`间隔进行`<n>`次证书操作。

##### `key_type`

指定为 TLS 证书生成的密钥类型；仅当你有特定需要对其进行自定义时才更改此设置。可能的值为：`ed25519`、`p256`、`p384`、`rsa2048`、`rsa4096`。

##### `cert_issuer`

定义TLS证书的颁发者（或来源）。发行者名称后面的标记设置发行者，就像在[tls指令](/docs/caddyfile/directives/tls#issuer)中指定的一样。如果你希望配置多个发行者来尝试，可以重复。它们将按照定义的顺序进行尝试。

##### `ocsp_stapling`

可以设置`off`为禁用OCSP装订。在由于防火墙而无法访问响应者的环境中很有用。

##### `preferred_chains`

如果你的CA提供多个证书链，你可以使用此选项来指定Caddy应该首选哪个链。设置以下选项之一：

- **smallest** 将告诉Caddy首选字节数最少的链。
- **root_common_name** 是一个或多个常用名称的列表；Caddy将选择第一个具有与至少一个指定的通用名称匹配的根的链。
- **any_common_name** 是一个或多个常用名称的列表；Caddy将选择第一个具有与至少一个指定的通用名称匹配的发行者的链。

请注意，如果没有任何[覆盖发行人级别的配置](/docs/caddyfile/directives/tls#acme)，`preferred_chains`将被作为全局选项影响所有发行人。

## 服务器选项

使用可能跨越多个站点的设置自定义[HTTP服务器](/docs/json/apps/http/servers/)，因此无法在站点块中正确配置。这些选项会影响侦听器/套接字或 HTTP 层下的其他行为。

可以多次指定，使用不同的`listener_address`值，为每个服务器配置不同的选项。例如，`servers :443`将仅适用于绑定到侦听器地址的服务器`:443`。省略侦听器地址会将选项应用于任何剩余的服务器。

<aside class="tip">
	使用该<a href="/docs/command-line#caddy-adapt"><code>caddy适配器</code></a>命令在 Caddyfile 中查找服务器的侦听地址。
</aside>

例如，要为port`:80`和`:443`指定不同的服务器配置，你可以指定两个`servers`块：

```caddy
{
	servers :443 {
		protocol {
			experimental_http3
		}
	}

	servers :80 {
		protocol {
			allow_h2c
		}
	}
}
```

##### `listener_wrappers`

允许配置[监听包装器](/docs/json/apps/http/servers/listener_wrappers/)，它可以修改基本侦听器的行为。它们按给定的顺序应用。

有一个特殊的无操作[`tls`](/docs/json/apps/http/servers/listener_wrappers/tls/)侦听器包装器作为标准模块提供，它标记应在侦听器包装器链中处理TLS的位置。仅当必须在TLS握手之前放置另一个侦听器包装器时才应使用它。

例如，假设你安装了[`proxy_protocol`](/docs/json/apps/http/servers/listener_wrappers/proxy_protocol/)插件：

```caddy-d
listener_wrappers {
	proxy_protocol {
		timeout 2s
		allow 192.168.86.1/24 192.168.86.1/24
	}
	tls
}
```


##### `timeouts`

- **read_body** 是一个[持续时间值](/docs/conventions#durations)，它设置允许从客户端上传读取多长时间。将此设置为一个较短的非零值可以减轻慢速攻击，但也可能影响合法慢速客户端。默认为无超时。

- **read_header** 是一个[持续时间值](/docs/conventions#durations)，它设置允许从客户端的请求标头读取多长时间。默认为无超时。

- **write** 是一个[持续时间值](/docs/conventions#durations)，用于设置允许写入客户端的时间长度。请注意，在提供大文件时将其设置为较小的值可能会对合法缓慢的客户端产生负面影响。默认为无超时。
- 
- **idle** 是一个[持续时间值](/docs/conventions#durations)，用于设置启用 keep-alives 时等待下一个请求的最长时间。默认为 5 分钟，以帮助避免资源耗尽。

##### `max_header_size`

从客户端的 HTTP 请求标头中解析的最大大小。它接受[go-humanize](https://github.com/dustin/go-humanize/blob/master/bytes.go)支持的所有格式。

##### `protocol`([DEPRECATED since 2022-08](https://github.com/caddyserver/caddy/blob/6d9a83376b5e19b3c0368541ee46044ab284038b/caddyconfig/httpcaddyfile/serveroptions.go#L241))

- **allow_h2c** 启用 H2C（“明文 HTTP/2”或“H2 over TCP”）支持，如果客户端支持，它将通过明文 TCP 连接提供 HTTP/2。由于 Go 标准库没有实现这一点，因此使用 H2C 与该服务器的大多数其他选项不兼容。不要仅仅为了实现最大的客户端兼容性而启用它。在实践中，很少有客户端实现 H2C，甚至更少需要它。此设置仅适用于未加密的 HTTP 侦听器。⚠️实验功能；可能会更改或删除。

- **experimental_http3** 启用实验性草案 HTTP/3 支持。请注意，HTTP/3 不是一个完整的规范，客户端支持非常有限。此选项将在未来消失。此选项不受兼容性承诺的约束。

- **strict_sni_host** 要求请求的`Host`标头与客户端的TLS ClientHello发送的ServerName的值匹配；使用TLS客户端身份验证时，通常是必要的保护措施。
