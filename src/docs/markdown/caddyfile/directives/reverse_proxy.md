---
title: reverse_proxy (Caddyfile指令)
---

# reverse_proxy

通过可配置的传输、负载平衡、健康检查、请求操作和缓冲选项，将请求代理到一个或多个后端。

- [语法](#syntax)
- [上游](#upstreams)
    - [上游地址](#upstream-addresses)
    - [动态上游](#dynamic-upstreams)
        - [SRV](#srv)
        - [A/AAAA](#aaaaa)
- [负载平衡](#load-balancing)
    - [主动健康检查](#active-health-checks)
    - [被动健康检查](#passive-health-checks)
- [流媒体](#streaming)
- [头信息](#headers)
- [重写](#rewrites)
- [传输](#transports)
	- [`http`传输](#the-http-transport)
	- [`fastcgi`传输](#the-fastcgi-transport)
- [拦截响应](#intercepting-responses)
- [例子](#examples)



<h3 id="syntax">Syntax</h3>

```caddy-d
reverse_proxy [<matcher>] [<upstreams...>] {
	# 后端
	to      <upstreams...>
	dynamic <module> ...

	# 负载平衡
	lb_policy       <name> [<options...>]
	lb_try_duration <duration>
	lb_try_interval <interval>

	# 主动健康检查
	health_uri      <uri>
	health_port     <port>
	health_interval <interval>
	health_timeout  <duration>
	health_status   <status>
	health_body     <regexp>
	health_headers {
		<field> [<values...>]
	}

	# 被动健康检查
	fail_duration     <duration>
	max_fails         <num>
	unhealthy_status  <status>
	unhealthy_latency <duration>
	unhealthy_request_count <num>

	# 流媒体
	flush_interval <duration>
	buffer_requests
	buffer_responses
	max_buffer_size <size>

	# 请求/header操作
	trusted_proxies [private_ranges] <ranges...>
	header_up   [+|-]<field> [<value|regexp> [<replacement>]]
	header_down [+|-]<field> [<value|regexp> [<replacement>]]
	method <method>
	rewrite <to>

	# 传输(round trip)
	transport <name> {
		...
	}

	# 可以选择拦截来自上游的响应
	@name {
		status <code...>
		header <field> [<value>]
	}
	replace_status [<matcher>] <status_code>
	handle_response [<matcher>] {
		<directives...>

		# 只在 handle_response 中可用的特殊指令
		copy_response [<matcher>] [<status>] {
			status <status>
		}
		copy_response_headers [<matcher>] {
			include <fields...>
			exclude <fields...>
		}
	}
}
```



<h3 id="upstreams">上游</h3>

- **&lt;upstreams...&gt;** 是一个要代理的上游（后端）的列表。
- **to** <span id="to"/> 是一个要代理的上游（后端）的列表。
- **dynamic** <span id="dynamic"/> 配置了一个_动态上游模块。这允许为每个请求动态地获得上游列表。参见下面的[动态上游](#dynamic-upstreams)，了解标准动态上游模块的描述。动态上游在每次代理循环迭代时被检索（即如果启用了负载平衡重试，每个请求有可能被检索多次），并且将比静态上游更受欢迎。如果发生错误，代理将退回到使用任何静态配置的上游。

<h4 id="upstream-addresses">上游地址</h4>

静态上游地址可以采取传统的[Caddy网络地址](/docs/conventions#network-addresses)或只包含scheme和host/port的URL形式。有效的例子。

- `localhost:4000`
- `127.0.0.1:4000`
- `http://localhost:4000`
- `https://example.com`
- `h2c://127.0.0.1`
- `example.com`
- `unix//var/php.sock`

注意：方案不能混合使用，因为它们修改了常见的传输配置（一个启用了TLS的传输不能同时携带HTTPS和纯文本HTTP）。任何明确的传输配置都不会被覆盖，省略方案或使用其他端口将不会假设一个特定的传输。

此外，上游地址不能包含路径或查询字符串，因为这将意味着在代理时同时重写请求，这种行为没有被定义或支持。如果需要，你可以使用[`rewrite`](/docs/caddyfile/directives/rewrite)指令。

如果地址不是URL（即没有方案），那么可以使用占位符，但这使得上游_动态静态_，这意味着在健康检查和负载平衡方面，可能有许多不同的后端作为一个单一的静态上游行事。

当通过HTTPS代理时，你可能需要覆盖`Host`头，使其与TLS SNI值相匹配，该值被服务器用于路由和证书选择。更多细节见下面的[HTTPS](#https)部分。

<h4 id="dynamic-upstreams">动态上游</h4>

Caddy的反向代理标准配备了一些动态上游模块。请注意，使用动态上游对负载平衡和健康检查有影响，这取决于具体的策略配置：主动健康检查不对动态上游运行；如果上游列表相对稳定和一致，负载平衡和被动健康检查效果最好（特别是轮流使用）。理想情况下，动态上游模块只返回健康、可用的后端。

<h5 id="srv">SRV</h5>

从SRV DNS记录中检索上游。

```caddy-d
	dynamic srv [<full_name>] {
		service   <service>
		proto     <proto>
		name      <name>
		refresh   <interval>
		resolvers <ip...>
		dial_timeout        <duration>
		dial_fallback_delay <duration>
	}
```

- **&lt;full_name&gt;** 是要查询的记录的完整域名（即`_service._proto.name`）。
- **service** 是全名的服务部分。
- **proto** 是全名中的协议部分。可以是`tcp`或`udp`。
- **name** 是名称部分。或者，如果`service`和`proto`是空的，则是要查询的全域名称。
- **refresh**是刷新缓存结果的频率。默认：`1m`。
- **resolvers** 是DNS解析器的列表，用于覆盖系统解析器。
- **dial_timeout**是拨号查询的超时时间。
- **dial_fallback_delay**是在生成RFC 6555快速回退连接之前要等待多长时间。默认：`300ms`。

<h5 id="a-aaaa">A/AAAA</h5>

从A/AAAA DNS记录中检索上游信息。

```caddy-d
	dynamic a [<name> <port>] {
		name      <name>
		port      <port>
		refresh   <interval>
		resolvers <ip...>
		dial_timeout        <duration>
		dial_fallback_delay <duration>
	}
```

- **name**是要查询的域名。
- **port**是后端使用的端口。
- **refresh**是刷新缓存结果的频率。默认：`1m`。
- **resolvers**是DNS解析器的列表，用于覆盖系统解析器。
- **dial_timeout**是拨号查询的超时时间。
- **dial_fallback_delay**是在生成RFC 6555快速回退连接之前要等待多长时间。默认值: `300ms`。


<h3 id="load-balancing">负载均衡</h3>

只要定义了一个以上的上游，就会使用负载平衡。

- **lb_policy** <span id="lb_policy"/> 是负载均衡策略的名称，以及任何选项。默认值：`random`。

  对于涉及散列的策略，使用[最高随机权重(HRW)](https://en.wikipedia.org/wiki/Rendezvous_hashing)算法，以确保具有相同散列键的客户端或请求被映射到相同的上游，即使上游列表改变。

    - `random`随机选择一个上游

    - `random_choose <n>`随机选择两个或多个上游，然后选择负载最小的一个（`n`通常为2）。

    - `first`根据配置中定义的顺序，选择第一个可用的上游

    - `round_robin`依次轮询每条上游

    - `least_conn`选择当前请求数最少的上游；如果有多个主机的请求数最少，那么就随机选择其中一个主机

    - `ip_hash`将客户的IP映射到一个对应的上游

    - `uri_hash`将请求的URI（路径和查询）映射到一个对应的上游。

    - `header [field]`通过散列头的值，将请求头映射到一个对应的上游；如果指定的头字段不存在，将随机选择一个上游。

    - `cookie [<name> [<secret>]]`在客户端的第一个请求中（当没有cookie时），随机选择一个上游，并在响应中添加一个`Set-Cookie'头（如果没有指定，默认cookie名称为`lb`）。Cookie的值是所选上游的拨号地址，用HMAC-SHA256加密（使用`<secret>`作为共享私钥，如果不指定则为空字符串）。

      在随后的请求中，如果存在cookie，cookie值将被映射到相同的上游；如果不可用或没有找到，将选择一个新的随机上游，cookie将被添加到响应中。

      如果你希望使用一个特定的上游来进行调试，你可以用密文对上游地址进行散列，并在你的HTTP客户端（浏览器或其他）设置cookie。例如，使用PHP，你可以运行以下程序来计算cookie值，其中`10.1.0.10:8080`是你的一个上游的地址，`secret`是你配置的秘密。

      ```php
      echo hash_hmac('sha256', '10.1.0.10:8080', 'secret');
      // cdd96966817dd14a99f47ee17451464f29998da170814a16b483e4c1ff4c48cf
      ```

      你可以通过Javascript控制台在浏览器中设置cookie，例如，设置名为`lb`的cookie。
      ```js
      document.cookie = "lb=cdd96966817dd14a99f47ee17451464f29998da170814a16b483e4c1ff4c48cf";
      ```

- **lb_try_duration** <span id="lb_try_duration"/> 是一个[持续时间值](/docs/conventions#durations)，它定义了在下一个可用的主机停机时，为每个请求尝试选择可用的后端多长时间。默认情况下，这种重试是禁用的。当负载均衡试图找到一个可用的上游主机时，客户将等待最多这么长时间。一个合理的起点可能是5s，因为HTTP传输的默认拨号超时是3s，所以如果不能到达第一个选定的上游，应该至少有一次重试的机会；这个值可以自由设置和验证，为你的使用场景寻求合适的平衡点。

- **lb_try_interval** <span id="lb_try_interval"/> 是一个[持续时间值](/docs/conventions#durations)，定义了从池中选择下一个主机的等待时间。默认是`250ms`。只有在向上游主机的请求失败时才会被用到。请注意，如果所有的后端都关闭了，且延迟很低的情况下，把这个值设置为0，并且`lb_try_duration`不为零的话，会导致CPU死锁(spin)。



<h4 id="active-health-checks">主动健康检查</h4>

主动健康检查是在后台以计时器的方式进行健康检查：

- **health_uri** <span id="health_uri"/> 是主动健康检查的URI路径（和可选查询）。

- **health_port** <span id="health_port"/> 是用于主动健康检查的端口，如果与上游的端口不同。

- **health_interval** <span id="health_interval"/>是一个[持续时间值](/docs/conventions#durations)，定义了执行主动健康检查的频率。

- **health_timeout** <span id="health_timeout"/>是一个[持续时间值](/docs/conventions#durations)，它定义了在将后端标记为停机之前要等待多久的回复。

- **health_status** <span id="health_status"/>是一个健康的后端所期望的HTTP状态代码。可以是一个3位数的状态代码，或以`xx`结尾的状态代码类。例如：`200`（这是默认的），或`2xx`。

- **health_body** <span id="health_body"/>是一个子字符串或正则表达式，用于匹配活动健康检查的响应体。如果后端没有返回一个匹配的主体，它将被标记为停机。

- **health_headers** <span id="health_headers"/> 允许指定在活动健康检查请求中设置的头文件。如果你需要改变`Host`头，或者你需要向你的后端提供一些认证作为健康检查的一部分，这很有用。


<h4 id="passive-health-checks">被动健康检查</h4>

被动健康检查是在实际代理的请求中进行的。

- **fail_duration** <span id="fail_duration"/>是一个[duration值](/docs/conventions#durations)，定义了记住一个失败请求的时间。持续时间> `0`可以启用被动健康检查；默认值是`0`（关闭）。一个合理的起点可能是`30s`，以平衡错误率和使不健康的上游重新上线时的响应性；但请自由试验，为你的用例找到正确的平衡。

- **max_fails** <span id="max_fails"/>是在考虑后端宕机之前，在`fail_duration`内失败请求的最大数量；必须>=`1`；默认是`1`。

- **unhealthy_status** <span id="unhealthy_status"/>如果一个请求的响应是这些状态码之一，则算作失败。可以是3位数的状态码或以`xx`结尾的状态码类别，例如：`404`或`5xx`。

- **unhealthy_latency** <span id="unhealthy_latency"/>是一个[持续时间值](/docs/conventions#durations)，如果一个请求需要这么长时间才能得到响应，则算作失败。

- **unhealthy_request_count** <span id="unhealthy_request_count"/>是将一个后端标记为停机前允许的同时请求的数量。换句话说，如果一个特定的后端目前正在处理这么多的请求，那么它被认为是 "超载"，其他后端将被优先考虑。

  这应该是一个相当大的数字；配置这个意味着代理将有一个`unhealthy_request_count × upstreams_count`的总同时请求的限制，任何超过这个点的请求将导致一个错误，因为没有上游可用。

<h3 id="events">事件</h3>
当上游从健康状态转变为不健康状态，或者反之，都会触发一个事件。这些事件可以用于触发其他操作，比如发送通知或记录消息。事件如下所示：

- `healthy`：在上游从之前的不健康状态标记为健康时触发。
- `unhealthy`：在上游从之前的健康状态标记为不健康时触发。

在这两种情况下，`host`将作为元数据包含在事件中，以标识状态发生变化的上游。例如，可以在使用`exec`事件处理程序时将其作为占位符与`{event.data.host}`一起使用。


<h3 id="streaming">流媒体</h3>

默认情况下，代理会对响应进行部分缓冲，以提高传输效率。

代理还支持WebSocket连接，它会执行HTTP升级请求，然后将连接转换为双向隧道。

>> 默认情况下，当配置重新加载时，WebSocket连接会被强制关闭（会向客户端和上游服务器发送Close控制消息）。每个请求都会持有对配置的引用，因此关闭旧连接是为了控制内存使用。这种关闭行为可以通过[stream_timeout](#stream_timeout)和[stream_close_delay](#stream_close_delay)选项进行自定义设置。

- **flush_interval** <span id="flush_interval"/>是一个[持续时间值](/docs/conventions#durations)，用于调整Caddy刷新响应缓冲区到客户端的频率。默认情况下，不进行周期性刷新。负值（通常为`-1`）表示“低延迟模式”，它完全禁用响应缓冲，会在每次写入客户端后立即刷新。与此同时，即使客户端提前断开连接，也不会取消对后端的请求。如果响应满足以下情况之一，则该选项会被忽略，响应会被立即刷新到客户端：
  - `Content-Type`为`text/event-stream`
  - `Content-Length`未知
  - 在代理的两端使用`HTTP/2`，`Content-Length`未知，并且`Accept-Encoding`未设置或为"identity"。

- **request_buffers** 会导致代理从请求体中读取最多`<size>`字节的数据到缓冲区中，然后再将其发送给上游服务器。这非常低效，只有在上游服务器需要立即读取请求体而没有延迟的情况下才应该这样做（这是上游应用程序应该修复的问题）。该选项接受所有由[go-humanize](https://github.com/dustin/go-humanize/blob/master/bytes.go)支持的大小格式。

- **response_buffers** 会导致代理从响应体中读取最多`<size>`字节的数据到缓冲区中，然后再返回给客户端。基于性能考虑，应尽量避免使用此选项，但如果后端对内存有更严格的限制，这可能会有用。该选项接受所有由[go-humanize](https://github.com/dustin/go-humanize/blob/master/bytes.go)支持的大小格式。

- **stream_timeout** 是一个[持续时间值](/docs/conventions#durations)，超过这个时间后，诸如WebSockets之类的流式请求将在超时结束时被强制关闭。这实质上会在连接保持开启时间过长时取消连接。一个合理的起始值可能是`24h`，以便清除一天前的连接。默认值：无超时。

- **stream_close_delay** 是一个[持续时间值](/docs/conventions#durations)，它会延迟流式请求（如WebSockets）在配置卸载时被强制关闭；相反，流将保持开启状态，直到延迟完成。换句话说，启用此选项可以防止在重新加载Caddy配置时立即关闭流。启用此选项可能是一个不错的主意，以避免在前一个配置关闭时断掉了连接它们的大量客户端。一个合理的起始值可能是`5m`，以便在配置重新加载后让用户在5分钟的时间后自然地离开页面。默认值：无延迟。


<h3 id="headers">头信息</h3>

代理可以在自己和后端之间**操纵头信息**。

- **header_up** <span id="header_up"/> 设置、添加（使用`+`前缀）、删除（使用`-`前缀）或执行替换（通过使用两个参数，搜索和替换）在请求头的上游到后端。

- **header_down** <span id="header_down"/> 在来自后端下游的响应头中设置、添加（使用`+`前缀）、删除（使用`-`前缀）或执行替换（通过使用两个参数，搜索和替换）。

例如，要设置一个请求头，覆盖任何现有的值。

```caddy-d
header_up Some-Header "the value"
```

添加一个响应头；注意一个头域可以有多个值。

```caddy-d
header_down +Some-Header "first value"
header_down +Some-Header "second value"
```

要删除一个请求头，防止它到达后端。

```caddy-d
header_up -Some-Header
```

要删除所有匹配的请求，使用后缀匹配。

```caddy-d
header_up -Some-*
```

对一个请求头进行正则表达式替换。

```caddy-d
header_up Some-Header "^prefix-([A-Za-z0-9]*)$" "replaced-$1-suffix"
```

使用的正则表达式语言是RE2，包含在Go中。参见 [RE2语法参考](https://github.com/google/re2/wiki/Syntax) 和 [Go正则表达式语法概述](https://pkg.go.dev/regexp/syntax)。替换字符串是[expanded](https://pkg.go.dev/regexp#Regexp.Expand)，允许使用捕获值，例如`$1`是第一个捕获组。

<h4 id="defautls">默认值</h4>

默认情况下，Caddy将传入的头信息&mdash;包括`Host`&mdash;传给后端，不做任何修改，有三个例外：

- 它设置或增加[`X-Forwarded-For`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)头域。
- 它设置[`X-Forwarded-Proto`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto)头域。
- 它设置了[`X-Forwarded-Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host)头字段。

<span id="trusted_proxies"/>对于这些`X-Forwarded-*`头，默认情况下，Caddy会忽略它们在传入请求中的值，以防止被欺骗。如果Caddy不是客户连接的第一个服务器（例如，当CDN在Caddy前面时），你可以用一个IP范围（CIDRs）列表来配置`trusted_proxies`，从这些IP范围传入的请求被信任为发送了这些头信息的允许的值。作为一个捷径，`private_ranges`可以被配置为信任所有私人IP范围。

```caddy-d
trusted_proxies private_ranges
```

<aside class="tip">
如果你在Caddy前面使用Cloudflare，要注意你可能会受到`X-Forwarded-For`头的欺骗。我们在[Authelia](https://www.authelia.com)的朋友记录了一个[解决方法](https://www.authelia.com/integration/proxies/fowarded-headers/)，以配置Cloudflare忽略这个头的传入值。
</aside>

此外，当使用[`http`传输](#the-http-transport)时，如果客户端的请求中缺少`Accept-Encoding: gzip`头，则会设置该头。这种行为可以通过在传输上使用[`compression off`](#compression)来禁用。

<h4 id="https">HTTPS</h4>

由于（大多数）头信息在被代理时保留其原始值，当代理到HTTPS时，通常需要用配置的上游地址覆盖`Host`头信息，这样`Host`头信息与TLS ServerName值相匹配。例如。

```caddy-d
reverse_proxy https://example.com {
	header_up Host {upstream_hostport}
}
```



<h3 id="rewrites">重写</h3>

默认情况下，Caddy用与传入请求相同的HTTP方法和URI执行上游请求，除非在到达`reverse_proxy`之前，在中间件链中进行了重写。

在代理之前，请求被克隆；这确保了在处理过程中对请求的任何修改不会泄露给其他处理程序。这在代理后需要继续处理的情况下很有用。

除了[头处理](#headers)之外，请求的方法和URI在被发送到上游之前可以被改变。

- **method** <span id="method"/>改变克隆请求的HTTP方法。如果方法被改为 "GET "或 "HEAD"，那么传入请求的主体将不会被这个处理程序发送到上游。如果你想让一个不同的处理程序消费请求正文，这是很有用的。
- **rewrite** <span id="rewrite"/> 改变克隆请求的URI（路径和查询）。这类似于[`rewrite`指令](/docs/caddyfile/directives/rewrite)，除了它不会在这个处理程序的范围内持续进行重写。

这些重写对于像 "预检查请求 "这样的模式通常是有用的，即请求被发送到另一个服务器，以帮助决定如何继续处理当前的请求。

例如，该请求可以被发送到一个认证网关，以决定该请求是否来自一个认证用户（例如，该请求有一个会话cookie）并应该继续，或者应该被重定向到一个登录页面。对于这种模式，Caddy提供了一个快捷指令[`forward_auth`](/docs/caddyfile/directives/forward_auth)来跳过大部分的配置模板。




<h3 id="transports">传输</h3>

Caddy的代理**传输**是可插入的。

- **transport** <span id="transport"/>定义了与后端通信的方式。默认是 "http"。


<h4 id="the-http-transport"><code>http</code>传输</h4>

```caddy-d
transport http {
	read_buffer             <size>
	write_buffer            <size>
	max_response_header     <size>
	dial_timeout            <duration>
	dial_fallback_delay     <duration>
	response_header_timeout <duration>
	expect_continue_timeout <duration>
	resolvers <ip...>
	tls
	tls_client_auth <automate_name> | <cert_file> <key_file>
	tls_insecure_skip_verify
	tls_timeout <duration>
	tls_trusted_ca_certs <pem_files...>
	tls_server_name <server_name>
	tls_renegotiation <level>
	tls_except_ports <ports...>
	keepalive [off|<duration>]
	keepalive_interval <interval>
	keepalive_idle_conns <max_count>
	keepalive_idle_conns_per_host <count>
	versions <versions...>
	compression off
	max_conns_per_host <count>
}
```

- **read_buffer** <span id="read_buffer"/>是读取缓冲区的大小，单位是字节。它接受[go-humanize](https://github.com/dustin/go-humanize/blob/master/bytes.go)支持的所有格式。默认值：`4KiB`。

- **write_buffer** <span id="write_buffer"/> 是写缓冲区的大小，单位是字节。它接受[go-humanize](https://github.com/dustin/go-humanize/blob/master/bytes.go)支持的所有格式。默认值：`4KiB`。

- **max_response_header** <span id="max_response_header"/> 是从响应头文件中读取的最大字节量。它接受[go-humanize]（https://github.com/dustin/go-humanize/blob/master/bytes.go）支持的所有格式。默认值：`10MiB`。

- **dial_timeout** <span id="dial_timeout"/>是连接到上游套接字时需要等待的时间。接受[持续时间值](/docs/conventions#durations)。默认值。没有超时。

- **dial_fallback_delay** <span id="dial_fallback_delay"/>是在生成RFC 6555快速回落连接之前要等待多长时间。负值会使其失效。接受[持续时间值]（/docs/conventions#durations）。默认值：`300ms`。

- **response_header_timeout** <span id="response_header_timeout"/>是从上游读取响应头的等待时间。接受[持续时间值]（/docs/conventions#durations）。默认值。没有超时。

- **expect_continue_timeout** <span id="expect_continue_timeout"/>是指如果请求有头 "Expect: 100-continue"，在完全写入请求头之后，等待上游的第一个响应头的时间。接受[持续时间值]（/docs/conventions#durations）。默认值。没有超时。

- **resolvers** <span id="resolvers"/>是一个DNS解析器的列表，用于覆盖系统解析器。

- **tls** <span id="tls"/>使用HTTPS与后端。如果你使用`https://`方案或端口`:443`指定后端，或者配置了以下任何`tls_*`选项，这将被自动启用。
- **tls_client_auth** <span id="tls_client_auth"/>启用TLS客户端认证的两种方式之一：（1）通过指定一个域名，Caddy应该为其获得证书并保持更新，或者（2）通过指定一个证书和密钥文件，以便向后端提交TLS客户端认证。

- **tls_insecure_skip_verify** <span id="tls_insecure_skip_verify"/>关闭TLS握手验证，使连接不安全，容易受到中间人攻击。请不要在生产中使用。

- **tls_timeout** <span id="tls_timeout"/>是一个[持续时间值](/docs/conventions#durations)，指定等待TLS握手完成的时间。默认值。没有超时。

- **tls_trusted_ca_certs** <span id="tls_trusted_ca_certs"/>是一个PEM文件的列表，指定连接到后端时要信任的CA公钥。

- **tls_server_name** <span id="tls_server_name"/>设置验证TLS握手中收到的证书时使用的服务器名称。默认情况下，这将使用上游地址的主机部分。

  只有当你的上游地址与上游可能使用的证书不匹配时，你才需要重写这个。例如，如果上游地址是一个IP地址，那么你需要将其配置为上游服务器所提供的主机名。

  可以使用一个请求占位符，在这种情况下，每次请求都会使用HTTP传输配置的克隆，这可能会产生性能损失。

- **tls_renegotiation** <span id="tls_renegotiation"/> 设置TLS重新协商级别。TLS重新协商是在第一次握手后执行后续握手的行为。该级别可以是以下之一。
	- `never`（默认）禁止重新协商。
	- `once`允许远程服务器在每个连接中请求重新协商一次。
	- `freely`允许远程服务器反复请求重新协商。

- **tls_except_ports** <span id="tls_except_ports"/>当TLS被启用时，如果上游目标使用给定的端口之一，TLS将被禁止用于这些连接。这在配置动态上游时可能很有用，一些上游期待HTTP，另一些期待HTTPS请求。

- **keepalive** <span id="keepalive"/>是`off`或者一个[持续时间值](/docs/conventions#durations)，指定保持连接开放的时间（超时）。默认值：`2m`。

- **keepalive_interval** <span id="keepalive"/>是一个[持续时间值](/docs/conventions#durations)，它指定了探测有效性的频率。默认值：`30s'。

- **keepalive_idle_conns** <span id="keepalive_idle_conns"/> 定义了要保持活的最大连接数。默认值。没有限制。

- **keepalive_idle_conns_per_host** <span id="keepalive_idle_conns_per_host"/> 如果非零，控制每个主机保持的最大空闲（keep-alive）连接。默认值：`32`。

- **versions** <span id="version"/> 允许自定义支持哪些版本的HTTP。作为一个特例，"h2c "是一个有效的值，它将启用到上游的明文HTTP/2连接（然而，这是一个非标准的功能，不使用Go的默认HTTP传输，所以它是排斥其他功能的；可能会被改变或删除）。默认：`1.1 2`，如果方案是`h2c://`，则是`h2c 2`。

- **压缩** <span id="compression"/>可以通过设置为 "off "来禁止对后端进行压缩。

- **max_conns_per_host** <span id="max_conns_per_host"/> 可以选择限制每个主机的总连接数，包括处于拨号、活动和空闲状态的连接。默认值。没有限制。


<h4 id="the-fastcgi-transport"><code>fastcgi</code>传输</h4>

```caddy-d
transport fastcgi {
	root  <path>
	split <at>
	env   <key> <value>
	resolve_root_symlink
	dial_timeout  <duration>
	read_timeout  <duration>
	write_timeout <duration>
}
```

- **root** <span id="root"/>是网站的根。默认：`{http.vars.root}`或当前工作目录。

- **split** <span id="split"/>是分割路径的位置，以便在URI的末端获得PATH_INFO。

- **env** <span id="env"/>设置一个额外的环境变量为给定值。可以为多个环境变量指定一次以上。

- **resolve_root_symlink** <span id="resolve_root_symlink"/> 通过评估一个符号链接（如果存在的话），可以将`root`目录解析为其实际值。

- **dial_timeout** <span id="dial_timeout"/>是连接到上游套接字时要等待多长时间。接受[持续时间值]（/docs/conventions#durations）。默认值：`3s'。

- **read_timeout** <span id="read_timeout"/>是指从FastCGI服务器读取数据时的等待时间。接受[持续时间值]（/docs/conventions#durations）。默认值：没有超时。

- **write_timeout** <span id="write_timeout"/>是向FastCGI服务器发送时要等待多长时间。接受[持续时间值]（/docs/conventions#durations）。默认：没有超时。

<aside class="tip">
如果你想为一个现代的PHP应用程序提供服务，你可能会寻找[`php_fastcgi`指令](/docs/caddyfile/directives/php_fastcgi)，这是一个使用`fastcgi`指令的代理的快捷方式，并对使用`index.php`作为路由入口进行必要的重写。
</aside>



<h3 id="intercepting-responses">拦截响应</h3>

反向代理可以被配置为拦截来自后端的响应。为了方便，可以定义响应匹配器（类似于请求匹配器的语法），第一个匹配的`handle_response`路由将被调用。

当一个响应处理程序被调用时，来自后端的响应不会被写入客户端，而配置的`handle_response`路由将被执行，并由该路由来写入响应。如果路由不写响应，那么请求处理将继续由任何在这个`reverse_proxy`之后的处理程序进行。

- **@name**是一个[响应匹配器](#response-matcher)的名字。只要每个响应匹配器有一个唯一的名字，就可以定义多个匹配器。响应可以根据状态代码和响应头的存在或值进行匹配。
- **replace_status** <span id="replace_status"/>当被给定匹配器匹配时，简单地改变响应的状态代码。
- **handle_response** <span id="handle_response"/>定义了在被给定的匹配器（或者，如果省略了匹配器，则是所有响应）匹配时要执行的路由。第一个匹配块将被应用。在`handle_response`块内，可以使用任何其他[指令]（/docs/caddyfile/指令）。

此外，在`handle_response`里面，可以使用两个特殊的处理指令。

- **copy_response** <span id="copy_response"/>将从后端收到的响应体复制到客户端。在这样做的时候，可以选择允许改变响应的状态代码。这个指令[在`respond`之前排序]（/docs/caddyfile/directives#directive-order）。
- **copy_response_headers** <span id="copy_response_headers"/>从后端复制响应头信息到客户端，可以选择包括_或排除一列头信息字段（不能同时指定`include`和`exclude`）。这个指令是[在`header`之后排序]（/docs/caddyfile/directives#directive-order）。

在 "handle_response "路由中，有三个占位符可用：

- `{rp.status_code}` 来自后端响应的状态代码。
- `{rp.status_text}` 来自后端响应的状态文本。
- `{rp.header.*}` 来自后端响应的头信息。


<h4 id="response-matcher">响应匹配器</h4>

**响应匹配器**可用于按特定标准过滤（或分类）响应。

##### status

```caddy-d
status <code...>
```

按HTTP状态代码。

- **&lt;code...&gt;** 是一个HTTP状态代码的列表。特殊情况是`2xx`, `3xx`, ...分别与200-299, 300-399, ...范围内的所有状态码相匹配。
- 
##### header

参见[`header`](/docs/caddyfile/matchers#header)请求匹配器的支持语法。

<h2 id="examples">示例</h2>

反向代理所有请求到本地后端：

```caddy-d
reverse_proxy localhost:9005
```

在3个后端之间负载平衡所有请求：

```caddy-d
reverse_proxy node1:80 node2:80 node3:80
```


同样的，但只有`/api`内的请求，并且使用`header`策略实现负载均衡：

```caddy-d
reverse_proxy /api/* node1:80 node2:80 node3:80 {
	lb_policy header X-My-Header
}
```


配置一些传输选项：

```caddy-d
reverse_proxy localhost:8080 {
	transport http {
		dial_timeout 2s
		response_header_timeout 30s
	}
}
```


反向代理到一个HTTPS端点：

```caddy-d
reverse_proxy https://example.com {
	header_up Host {upstream_hostport}
}
```


在代理前去除一个路径前缀：

```caddy-d
handle_path /prefix/* {
	reverse_proxy localhost:9000
}
```

在代理之前替换一个路径前缀：

```caddy-d
handle_path /old-prefix/* {
	rewrite * /new-prefix{path}
	reverse_proxy localhost:9000
}
```

当Caddy在另一个代理或负载平衡器后面，其IP是`123.123.123.123`，可能设置`X-Forwarded-*`头以识别原始客户请求的细节，该下游代理必须被列为可信任的，否则Caddy将忽略这些传入的头：

```caddy-d
reverse_proxy localhost:8080 {
	trusted_proxies 123.123.123.123
}
```


支持X-Accel-Redirect，也就是说，按照上游代理的要求提供静态文件：

```caddy-d
reverse_proxy localhost:8080 {
	@accel header X-Accel-Redirect *
	handle_response @accel {
		root    * /path/to/private/files
		rewrite * {rp.header.X-Accel-Redirect}
		file_server
	}
}
```


为来自上游的错误定制错误页面：

```caddy-d
reverse_proxy localhost:8080 {
	@error status 500 503
	handle_response @error {
		root    * /path/to/error/pages
		rewrite * /{rp.status_code}.html
		file_server
	}
}
```


从A/AAAA记录的DNS查询中动态地获取后端：

```caddy-d
reverse_proxy {
	dynamic a example.com 9000
}
```


从SRV记录的DNS查询中动态地获取后端：

```caddy-d
reverse_proxy {
	dynamic srv _api._tcp.example.com
}
```
