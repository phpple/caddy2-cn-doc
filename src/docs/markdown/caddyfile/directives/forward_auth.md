---
title: forward_auth (Caddyfile指令)
---

# forward_auth

一个专用的(opinionated)指令，它会把请求复制给认证网关，认证网关可以决定是否应该继续处理，或者需要跳转到一个登录页面。

- [语法](#syntax)
- [扩展的形式](#expanded-form)
- [例子](#example)
  - [authelia](#authelia)
  - [tailscale](#tailscale)

Caddy的[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)能够对外部服务进行 "预检查请求"，但这个指令是专门为认证用例定制的。这条指令实际上只是提供了一种可以使用更长、更常见配置的方法（如下）。

这条指令向配置的上游发出`GET`请求，并重写了`uri`。
- 如果上游响应的是`2xx`状态码，那么访问被允许，`copy_headers`中的头域被复制到原始请求中，并继续处理。
- 否则，如果上游响应任何其他状态代码，那么上游的响应将被复制回客户端。这个响应通常应该包括重定向到认证网关的登录页面。

如果这个行为不是你想要的，你可以把下面的[expanded form](#expanded-form)作为基础，根据你的需要进行定制。

该指令支持[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)的所有子指令，并将它们传递给底层的`reverse_proxy`处理器。


<h3 id="syntax">语法</h3>

```caddy-d
forward_auth [<matcher>] [<upstreams...>] {
	uri          <to>
	copy_headers <fields...> {
		<fields...>
	}
}
```

- **&lt;upstreams...&gt;** 是一个上游（后端）的列表，向其发送认证请求。

- **uri** 是URI（路径和查询），在发送到上游的请求中设置。这通常是认证网关的验证端点。

- **copy_headers** 是一个HTTP头字段的列表，当请求有成功状态代码时，要从响应中复制给原始请求。

  该字段可以通过使用`>`后面的新名称来重命名，例如`Before>After`。

  如果想提升可读性，你可以用一个块来列出所有的字段，每行一个。

由于该指令是对反向代理的专门的(opinionated)包装，你可以使用[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy#syntax)的任何子指令来定制它。

<h3 id="expanded-form">扩展的形式</h3>

`forward_auth`指令与以下配置相同。像[Authelia](https://www.authelia.com/)这样的验证网关在这个预设中能正常运行。如果你还没有任何配置，请随意使用，并根据需要进行定制，而不是使用`forward_auth`快捷方式。

```caddy-d
reverse_proxy <upstreams...> {
	# 总是GET，这样传入的请求的主体(request's body)不会被带入
	method GET

	# 改变URI到认证网关的验证端点
	rewrite <to>

	# 转发原来的方法和URI，因为它们在上面被重写了；
	# 这是对其他已经被reverse_proxy设置的X-Forwarded-*头信息的补充。
	header_up X-Forwarded-Method {method}
	header_up X-Forwarded-Uri {uri}

	# 在一个成功的响应中，复制响应头文件
	@good status 2xx
	handle_response @good {
		request_header {
			# 例如，对于每个 copy_headers 字段...
			Remote-User {rp.header.Remote-User}
			Remote-Email {rp.header.Remote-Email}
		}
	}
}
```


<h3 id="example">示例</h3>


### Authelia

在通过反向代理提供你的应用程序之前，将认证委托给[Authelia](https://www.authelia.com/)。

```caddy
# 服务于认证网关本身
auth.example.com {
	reverse_proxy authelia:9091
}

# 服务于你的应用程序
app1.example.com {
	forward_auth authelia:9091 {
		uri /api/verify?rd=https://auth.example.com
		copy_headers Remote-User Remote-Groups Remote-Name Remote-Email
	}

	reverse_proxy app1:8080
}
```

更多信息，请参阅[Authelia的文档](https://www.authelia.com/integration/proxies/caddy/)，便于与Caddy集成。


### Tailscale

将认证委托给[Tailscale](https://tailscale.com/)（目前名为[`nginx-auth`](https://tailscale.com/blog/tailscale-auth-nginx/)，但它仍然可以与Caddy一起使用），并使用`copy_headers`的替代语法来*重命名*复制的头文件（注意每个头文件中的`>`）。

```caddy-d
forward_auth unix//run/tailscale.nginx-auth.sock {
	uri /auth
	header_up Remote-Addr {remote_host}
	header_up Remote-Port {remote_port}
	header_up Original-URI {uri}
	copy_headers {
		Tailscale-User>X-Webauth-User
		Tailscale-Name>X-Webauth-Name
		Tailscale-Login>X-Webauth-Login
		Tailscale-Tailnet>X-Webauth-Tailnet
		Tailscale-Profile-Picture>X-Webauth-Profile-Picture
	}
}
```