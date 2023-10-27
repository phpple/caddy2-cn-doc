---
title: "API教程"
---

# API教程

本教程将向你展示如何使用Caddy的[管理API](/docs/api)，这使得以可编程方式实现自动化成为可能。

**目标：**
- 🔲 运行守护程序
- 🔲 给 Caddy 一个配置
- 🔲 测试配置
- 🔲 替换活动配置
- 🔲 遍历配置
- 🔲 使用`@id`标签

**必备：**
- 基本的终端/命令行技能
- 基本的JSON经验
- PATH支持`caddy`和`curl`

---

要启动 Caddy 守护程序，请使用`run`子命令：

<pre><code class="cmd bash">caddy run</code></pre>

<aside class="complete">运行守护程序</aside>

这永远阻塞，但它在做什么？此刻……什么都没有。默认情况下，Caddy的配置（“config”）为空白。我们可以使用另一个终端中的[管理API](api)来验证这一点：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

我们可以通过给它一个配置来使Caddy变得有用。一种方法是向 [/load](/docs/api#post-load)端点发出 POST 请求。就像任何 HTTP 请求一样，有很多方法可以做到这一点，但在本教程中，我们将使用`curl`。

## 你的第一个配置

为了能发起请求，我们需要进行配置。Caddy的配置只是一个[JSON文档](/docs/json/) （或[任何能转换为JSON](/docs/config-adapters)的文件）。

<aside class="tip">
    不需要配置文件。配置API始终可以在没有文件的情况下使用，这非常利于实现自动化。本教程则使用文件，因为它更方便手动编辑。	
</aside>

将下面的内容保存到JSON文件：

```json
{
	"apps": {
		"http": {
			"servers": {
				"example": {
					"listen": [":2015"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Hello, world!"
							}]
						}
					]
				}
			}
		}
	}
}
```

然后上传：

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @caddy.json
</code></pre>

<aside class="tip">
    确保不要忘记文件名前面的`@`；这告诉`curl`你正在发送一个文件。
</aside>

<aside class="complete">给Caddy一个配置</aside>

我们可以验证 Caddy 是否通过另一个 GET 请求应用了我们的新配置：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

通过在浏览器中访问[localhost:2015](http://localhost:2015)或使用`curl`命令来测试它是否有效：

<pre><code class="cmd"><span class="bash">curl localhost:2015</span>
Hello, world!</code></pre>

<aside class="complete">测试配置</aside>

如果你看到_Hello, world!_，就恭喜啦——它已经工作了！确保你的配置按预期工作始终是一个好主意，尤其是在部署到生产环境之前。

```json
{
	"handler": "static_response",
	"body": "I can do hard things."
}
```

保存配置文件，然后通过再次运行相同的POST请求来更新Caddy的活动配置：

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @caddy.json
</code></pre>

<aside class="complete">替换活动配置</aside>

为了更好地衡量，请验证配置是否已更新：

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

通过在浏览器中刷新页面（或再次运行`curl`）来测试它，你将看到一条鼓舞人心的消息！

## 遍历配置

让我们使用 Caddy API 的强大功能来进行更改，而不需要修改我们的配置文件，而不是上传整个配置文件。

通过像我们上面所做的那样替换整个配置来对生产服务器进行少量更改可能是危险的；这就像拥有对文件系统的 root 访问权限。Caddy 的 API 允许你限制更改的范围，以确保配置的其他部分不会被意外更改。
使用请求 URI 的路径，我们可以遍历配置结构并仅更新消息字符串（如果被剪裁，请确保向右滚动）：

Instead of uploading the entire config file for a small change, let's use a powerful feature of Caddy's API to make the change without ever touching our config file.

<aside class="tip">
通过像我们上面所做的那样替换整个配置来对生产服务器进行少量更改可能是危险的；这就像拥有对文件系统的root访问权限。Caddy的API允许你限制更改的范围，以确保配置的其他部分不会被意外更改。
</aside>

使用请求 URI 的路径，我们可以遍历配置结构并仅更新消息字符串（如果被遮挡，请确保向右滚动）：

<pre><code class="cmd bash">curl \
	localhost:2019/config/apps/http/servers/example/routes/0/handle/0/body \
	-H "Content-Type: application/json" \
	-d '"Work smarter, not harder."'
</code></pre>

<aside class="tip">
	每次你使用 API 更改配置时，Caddy 都会保留一份新配置的副本，便于你可以通过<a href="/docs/command-line#caddy-run"><b>--resume</b>恢复</a>!
</aside>

你可以验证它是否适用于类似的GET请求，例如：

<pre><code class="cmd bash">curl localhost:2019/config/apps/http/servers/example/routes</code></pre>

你应该看到：

```json
[{"handle":[{"body":"Work smarter, not harder.","handler":"static_response"}]}]
```

<aside class="tip">
    你可以使用<a href="https://stedolan.github.io/jq/">jq command</a>命令来美化 JSON 输出：<code>curl ... | jq</code>
</aside>

<aside class="complete">遍历配置</aside>

**重要提示：**
显而易见，一旦你使用API进行配置，原始配置文件并不会被修改，这样你的配置文件就过时了。有几种方法可以处理这个问题：

- 使用`--resume`作为[caddy run](/docs/command-line#caddy-run)命令的最后一个参数。
- 不要混合使用配置文件和API两种方式进行更改; 始终使用同一种方式。
- 使用GET请求[导出Caddy的新配置](/docs/api#get-configpath) (不如前两个选项推荐)。

## 在JSON中使用`@id`

配置遍历当然有用，但是路径有点长，你不觉得吗？

我们可以给我们的处理对象一个[`@id`标签](/docs/api#using-id-in-json)，使它更容易访问：

<pre><code class="cmd bash">curl \
	localhost:2019/config/apps/http/servers/example/routes/0/handle/0/@id \
	-H "Content-Type: application/json" \
	-d '"msg"'
</code></pre>

这给我们的处理对象添加了一个属性："@id": "msg"，所以它现在看起来像这样：

```json
{
	"@id": "msg",
	"body": "Work smarter, not harder.",
	"handler": "static_response"
}
```

<aside class="tip">
    <b>@id</b>标签可以放在任何对象中，并且可以有任何原始值（通常是字符串）。<a href="/docs/api#using-id-in-json">了解更多</a>
</aside>

然后我们可以直接访问它：

<pre><code class="cmd bash">curl localhost:2019/id/msg</code></pre>

现在我们也通过更短的路径更改消息：

<pre><code class="cmd bash">curl \
	localhost:2019/id/msg/body \
	-H "Content-Type: application/json" \
	-d '"Some shortcuts are good."'
</code></pre>

并再次检查：

<pre><code class="cmd bash">curl localhost:2019/id/msg/body</code></pre>

<aside class="complete">使用<code>@id</code>标签</aside>
