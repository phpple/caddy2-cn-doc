---
title: "API"
---

# API

Caddy是通过管理端点进行配置的，该端点可以使用[REST](https://en.wikipedia.org/wiki/Representational_state_transfer)API通过HTTP访问。您可以在Caddy配置中[配置此端点](/docs/json/admin/)。

**默认地址：`localhost:2019`**

<aside class="tip">
	如果您在服务器上运行不受信任的代码（哎呀😬)，请确保通过隔离进程、修补易受攻击的程序以及将端点配置为绑定到许可的unix套接字来保护您管理的端点。
</aside>

最新的配置将在任何更改后保存到磁盘（除非[禁用](/docs/json/admin/config/))在重启后恢复上一个工作配置，这可以保证在电源循环或类似情况下配置的持久性。

要开始使用 API，请尝试我们的[API教程](/docs/api-tutorial)或者，如果您只有一分钟时间，请尝试我们的[API快速入门指南](/docs/quick-starts/api)。

---

- **[POST /load](#post-load)**
  设置或替换当前配置

- **[POST /stop](#post-stop)**
  停止当前配置并退出进程

- **[GET /config/[path]](#get-configpath)**
  导出指定路径的配置

- **[POST /config/[path]](#post-configpath)**
  设置或替换对象；追加到数组

- **[PUT /config/[path]](#put-configpath)**
  创建新对象；插入数组

- **[PATCH /config/[path]](#patch-configpath)**
  替换现有对象或数组元素

- **[DELETE /config/[path]](#delete-configpath)**
  删除指定路径的值

- **[在JSON中使用`@id`](#在JSON中使用`@id`)**
  轻松遍历配置结构

- **[GET /reverse_proxy/upstreams](#get-reverse-proxyupstreams)**
  返回配置的代理上游的当前状态


## POST /load

设置 Caddy 的配置，覆盖任何以前的配置。它会一直阻塞，直到重新加载完成或失败。配置更改是轻量级、高效的，并且会导致零停机。如果新配置因任何原因失败，则旧配置将回滚到原位而不会停机。

该端点使用配置适配器支持不同的配置格式。请求的`Content-Type`标头指示请求正文中使用的配置格式。通常这个值应该是`application/json`，代表Caddy的原生配置格式。对于其他配置格式，请指定适当的`Content-Type`，正斜杠`/`之后的值是要使用的配置适配器的名称。例如，在提交Caddyfile时，使用类似于`text/caddyfile`的值；或者对于JSON 5，使用类似于`application/json5`的值; 等等。

如果新配置与当前配置相同，则不会发生重新加载。要强制重新加载，请在请求标头中设置`Cache-Control: must-revalidate`。

### 示例

设置新的活动配置：

<pre><code class="cmd bash">curl -X POST "http://localhost:2019/load" \
	-H "Content-Type: application/json" \
	-d @caddy.json</code></pre>

注意：`curl`的`-d`标志会删除换行符，因此如果您的配置格式对换行符敏感（例如 Caddyfile），请改用`--data-binary`：

<pre><code class="cmd bash">curl -X POST "http://localhost:2019/load" \
	-H "Content-Type: text/caddyfile" \
	--data-binary @Caddyfile</code></pre>


## POST /stop

优雅地关闭服务器并退出进程。要仅停止正在运行的配置而不退出进程，请使用[DELETE /config/](#delete-configpath)。

### 示例

停止进程：

<pre><code class="cmd bash">curl -X POST "http://localhost:2019/stop"</code></pre>


## GET /config/[path]

在命名路径中导出 Caddy 的当前配置。返回 JSON 正文。

### 示例

导出整个配置并漂亮地打印它：

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/config/" | jq</span>
{
	"apps": {
		"http": {
			"servers": {
				"myserver": {
					"listen": [
						":443"
					],
					"routes": [
						{
							"match": [
								{
									"host": [
										"example.com"
									]
								}
							],
							"handle": [
								{
									"handler": "file_server"
								}
							]
						}
					]
				}
			}
		}
	}
}</code></pre>

仅导出侦听器地址：

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/config/apps/http/servers/myserver/listen"</span>
[":443"]</code></pre>



## POST /config/[path]

将 Caddy 的配置更改为请求的 JSON 正文的命名路径。如果目标值是一个数组，则追加 POST；如果是一个对象，则会进行创建或替换。

作为一种特殊情况，如果满足以下条件，可以将许多项目添加到数组中：

1. 路径结束于`/...`
2. `/...`之前的路径元素指的是一个数组
3. 有效载荷是一个数组

在这种情况下，有效载荷数组中的元素将被扩展，并且每个元素都将附加到目标数组中。在 Go 术语中，这将具有与以下相同的效果：

```go
baseSlice = append(baseSlice, newElems...)
```

### 示例

添加监听地址：

<pre><code class="cmd bash">curl -X POST \
	-H "Content-Type: application/json" \
	-d '":8080"' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen"</code></pre>

添加多个监听地址：

<pre><code class="cmd bash">curl -X POST \
	-H "Content-Type: application/json" \
	-d '[":8080", ":5133"]' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen/..."</code></pre>

## PUT /config/[path]

将 Caddy 的配置更改为请求的 JSON 正文的命名路径。如果目标值是数组中的位置（索引），则 PUT 插入；如果是一个对象，它会严格创建一个新值。

### 示例

在第一个槽中添加监听地址：

<pre><code class="cmd bash">curl -X PUT \
	-H "Content-Type: application/json" \
	-d '":8080"' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen/0"</code></pre>


## PATCH /config/[path]

将 Caddy 的配置更改为请求的 JSON 正文的命名路径。PATCH 严格替换现有值或数组元素。

### 示例

替换监听地址：

<pre><code class="cmd bash">curl -X PATCH \
	-H "Content-Type: application/json" \
	-d '[":8081", ":8082"]' \
	"http://localhost:2019/config/apps/http/servers/myserver/listen"</code></pre>



## DELETE /config/[path]

在命名路径中删除 Caddy 的配置。DELETE 删除目标值。

### 示例

要卸载整个当前配置但保持进程运行：

<pre><code class="cmd bash">curl -X DELETE "http://localhost:2019/config/"</code></pre>

只停止一个 HTTP 服务器：

<pre><code class="cmd bash">curl -X DELETE "http://localhost:2019/config/apps/http/servers/myserver"</code></pre>


## 在JSON中使用`@id`

您可以在 JSON 文档中嵌入 ID，以便更轻松地直接访问 JSON 的这些部分。

只需添加一个称为`"@id"`对象的字段并为其指定一个唯一名称。例如，如果您有一个想要经常访问的反向代理处理程序：

```json
{
	"@id": "my_proxy",
	"handler": "reverse_proxy"
}
```

要使用它，只需以与`/config/`相应端点相同的方式向`/id/`API端点发出请求，但无需完整路径。ID 将请求直接带入您的配置范围。

例如，要在没有 ID 的情况下访问反向代理的上游，路径将类似于：

```
/config/apps/http/servers/myserver/routes/1/handle/0/upstreams
```

但是有了ID，路径就变成了：

```
/id/my_proxy/upstreams
```

这更容易记忆和手写。


## GET /reverse_proxy/upstreams

将配置的反向代理上游（后端）的当前状态作为 JSON 文档返回。

<pre><code class="cmd"><span class="bash">curl "http://localhost:2019/reverse_proxy/upstreams" | jq</span>
[
	{"address": "10.0.1.1:80", "healthy": true, "num_requests": 4, "fails": 2},
	{"address": "10.0.1.2:80", "healthy": true, "num_requests": 5, "fails": 4},
	{"address": "10.0.1.3:80", "healthy": true, "num_requests": 3, "fails": 3}
]</code></pre>

JSON 数组中的每个条目都是存储在全局上游池中的已配置[upstream](/docs/json/apps/http/servers/routes/handle/reverse_proxy/upstreams/)。

- **address** 上游的拨号地址。对于SRV上游，这是`lookup_srv`的DNS名称。
- **healthy** 反映了Caddy是否认为上游是健康的。请注意，“健康”是与“可用性”不同的概念。不健康的后端将始终不可用，但健康的后端可能不可用。无论特定的反向代理处理程序配置如何，运行状况都是一个全局特性，而可用性由特定的反向代理处理程序的配置决定。例如，如果处理程序被配置为一次只允许N个请求并且它当前有N个活动请求，那么健康的后端将不可用。“健康”属性不反映可用性。
- **num_requests** 是上游当前正在处理的活动请求的数量。
- **fails** 当前记录的失败请求数，由被动运行状况检查配置。

如果您的目标是确定后端的可用性，则需要根据您正在使用的处理程序配置交叉检查上游的相关属性。例如，如果您为代理启用了[被动健康检查](/docs/json/apps/http/servers/routes/handle/reverse_proxy/health_checks/passive/)，那么你还需要考虑`fails`和`num_requests`值来确定上游是否可用：检查`fails`数量是否小于为你的代理配置的最大故障数量代理（即[`max_fails`](/docs/json/apps/http/servers/routes/handle/reverse_proxy/health_checks/passive/max_fails)），并且`num_requests`小于或等于您配置的每个上游的最大请求量（即对于整个代理的[`unhealthy_request_count`](/docs/json/apps/http/servers/routes/handle/reverse_proxy/health_checks/passive/unhealthy_request_count)，或对于单个上游的[`max_requests`](/docs/json/apps/http/servers/routes/handle/reverse_proxy/upstreams/max_requests)）。