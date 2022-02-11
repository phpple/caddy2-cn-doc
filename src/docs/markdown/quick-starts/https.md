---
title: HTTPS快速入门
---

# HTTPS快速入门

本指南将向你展示如何立即启动并运行[完全托管的HTTPS](/docs/automatic-https)。

<aside class="tip">
    Caddy默认对所有站点使用HTTPS，只要在配置中提供了主机名。本教程假设你希望通过HTTPS获得一个公共信任的站点（即不是“localhost”），因此我们将使用公共域名和外部端口。
</aside>

**先决条件：**
- 基本的终端/命令行技能
- 对DNS的基本了解
- 已注册的公共域名
- 外部访问端口 80 和 443
- PATH变量中包含`caddy`和`curl`

---

在本教程中，替换`example.com`为你的实际域名。

将你域的`A/AAAA`记录指向你的服务器。你可以通过登录你的DNS提供商并管理你的域名来做到这一点。

在继续之前，请使用权威查找验证正确的记录。替换`example.com`为你的域名，如果你使用的是`IPv6`，请替换`type=A`为`type=AAAA`：

<pre><code class="cmd bash">curl "https://cloudflare-dns.com/dns-query?name=example.com&type=A" \
  -H "accept: application/dns-json"</code></pre>

还要确保你的服务器可从公共接口通过端口`80`和`443`进行外部访问。

<aside class="tip">
    如果你在家庭或其他受限网络上，你可能需要转发端口或调整防火墙设置。
</aside>

我们所要做的就是在配置中使用你的域名启动Caddy。有几种方法可以做到这一点。

## Caddyfile

这是获取HTTPS最常用的方法。

创建一个名为`Caddyfile`（无扩展名）的文件，其中第一行是你的域名，例如：

```caddy
example.com

respond "Hello, privacy!"
```

然后从同一目录运行：

<pre><code class="cmd bash">caddy run</code></pre>

你将看到Caddy提供TLS证书并通过HTTPS为你的站点提供服务。这是可能的，因为你的站点在Caddyfile中的地址包含一个域名。

## `file-server`命令

如果你只需要通过 HTTPS 提供静态文件，请运行以下命令（替换你的域名）：

<pre><code class="cmd bash">caddy file-server --domain example.com</code></pre>

你将看到Caddy提供TLS证书并通过HTTPS为你的站点提供服务。


## `reverse-proxy`命令

如果你只需要一个基于HTTPS的简单反向代理（作为TLS终结器），请运行以下命令（替换你的域名和实际后端地址）：

<pre><code class="cmd bash">caddy reverse-proxy --from example.com --to localhost:9000</code></pre>

你将看到Caddy提供TLS证书并通过HTTPS为你的站点提供服务。

## JSON配置

一般的经验法则是任何[主机匹配器]((/docs/json/apps/http/servers/routes/match/host/))都会触发自动HTTPS。

因此，如下所示的JSON配置将启用生产就绪的[自动HTTPS](/docs/automatic-https)：

```json
{
	"apps": {
		"http": {
			"servers": {
				"hello": {
					"listen": [":443"],
					"routes": [
						{
							"match": [{
								"host": ["example.com"]
							}],
							"handle": [{
								"handler": "static_response",
								"body": "Hello, privacy!"
							}]
						}
					]
				}
			}
		}
	}
}
```