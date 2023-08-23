---
title: "模块命名空间"
---

# 模块命名空间

Caddy的客户端模块以`interface{}`或`any`类型进行通用加载。为了使主机模块能够使用它们，加载的客户端模块通常首先进行已知类型的类型断言。本页面描述了所有标准模块的模块命名空间到Go类型的映射。

非标准模块命名空间的文档可以在定义它们的主机模块的文档中找到。

<aside class="tip">
	读取此表的一种方式是，"如果您的模块位于&lt;namespace&gt;，则它应该编译为&lt;type&gt;。"
</aside>

命名空间 | 预期类型                                                                                                                                      | 描述 | 备注
--------- |-------------------------------------------------------------------------------------------------------------------------------------------| ----------- | ----------
|         | [`caddy.App`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#App)                                                             | Caddy应用程序
caddy.config_loaders | [`caddy.ConfigLoader`](https://pkg.go.dev/github.com/caddyserver/caddy/v2#ConfigLoader)                                                   | 加载配置 | <i>⚠️ 实验性</i>
caddy.fs  | [`fs.FS`](https://pkg.go.dev/io/fs#FS)                                                                                                    | 虚拟文件系统 |  <i>⚠️ 实验性</i>
caddy.listeners | [`caddy.ListenerWrapper`](https://pkg.go.dev/github.com/caddyserver/caddy/v2#ListenerWrapper)                                             | 包装网络监听器
caddy.logging.encoders | [`zapcore.Encoder`](https://pkg.go.dev/go.uber.org/zap/zapcore#Encoder)                                                                   | 日志条目编码器
caddy.logging.encoders.filter | [`logging.LogFieldFilter`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/logging?tab=doc#LogFieldFilter)                     | 日志字段过滤器
caddy.logging.writers | [`caddy.WriterOpener`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#WriterOpener)                                           | 日志编写器
caddy.storage | [`caddy.StorageConverter`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#StorageConverter)                                   | 存储后端
dns.providers | [`certmagic.ACMEDNSProvider`](https://pkg.go.dev/github.com/caddyserver/certmagic#ACMEDNSProvider)                                        | DNS挑战求解器
events.handlers | [`caddyevents.Handler`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyevents#Handler)                                   | 事件处理程序 | <i>⚠️ 实验性</i>
http.authentication.hashes | [`caddyauth.Comparer`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/caddyauth?tab=doc#Comparer)                   | 密码哈希比较器
http.authentication.providers | [`caddyauth.Authenticator`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/caddyauth?tab=doc#Authenticator)         | HTTP身份验证提供程序
http.encoders | [`encode.Encoder`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/encode#Encoder)                                   | 通常是压缩
http.handlers | [`caddyhttp.MiddlewareHandler`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp#MiddlewareHandler)                   | HTTP处理程序
http.ip_sources | [`caddyhttp.IPRangeSource`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp#IPRangeSource)                           | 可信代理的IP范围
http.matchers | [`caddyhttp.RequestMatcher`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp?tab=doc#RequestMatcher)                 | HTTP请求匹配器
http.precompressed | [`encode.Precompressed`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/encode#Precompressed)                       | 支持的预压缩映射
http.reverse_proxy.circuit_breakers | [`reverseproxy.CircuitBreaker`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/reverseproxy?tab=doc#CircuitBreaker) | 反向代理断路器
http.reverse_proxy.selection_policies | [`reverseproxy.Selector`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/reverseproxy?tab=doc#Selector)             | 负载均衡选择策略
http.reverse_proxy.transport | [`http.RoundTripper`](https://pkg.go.dev/net/http?tab=doc#RoundTripper)                                                                   | HTTP反向代理传输
http.reverse_proxy.upstreams | [`reverseproxy.UpstreamSource`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddyhttp/reverseproxy?tab=doc#UpstreamSource) | 动态上游源 | <i>⚠️ 实验性</i>
tls.certificates | [`caddytls.CertificateLoader`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddytls?tab=doc#CertificateLoader)             | TLS证书源
tls.client_auth | [`caddytls.ClientCertificateVerifier`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddytls#ClientCertificateVerifier)     | 验证客户端证书
tls.handshake_match | [`caddytls.ConnectionMatcher`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddytls?tab=doc#ConnectionMatcher)             | TLS连接匹配器
tls.issuance | [`certmagic.Issuer`](https://pkg.go.dev/github.com/caddyserver/certmagic?tab=doc#Issuer)                                                  | TLS证书签发
tls.get_certificate | [`certmagic.Manager`](https://pkg.go.dev/github.com/caddyserver/certmagic?tab=doc#Manager)                                                | TLS证书管理器 | <i>⚠️ 实验性</i>
tls.stek | [`caddytls.STEKProvider`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/modules/caddytls?tab=doc#STEKProvider)                       | TLS会话票据密钥源

标记为"实验性"的命名空间可能会发生更改。（请和他们一起开发，以便我们可以最终确定他们的接口！）
