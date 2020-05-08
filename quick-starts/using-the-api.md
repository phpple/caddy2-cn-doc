# API快速入门

前提：
* 基本的终端/命令行技巧；
* 已经支持`caddy`和`curl`命令

----------------------------

首先启动caddy：

```bash
caddy start
```

caddy现在在空转（使用空白配置）。通过`curl`命令给它一个简单的配置：

```bash

curl localhost:2019/load \
	-X POST \
    -H "Content-Type: application/json" \
    -d @- << EOF
	{
		"apps": {
			"http": {
				"servers": {
					"hello": {
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
EOF
```

通过[heredoc](https://en.wikipedia.org/wiki/Here_document#Unix_shells)的形式定义POST内容看起来非常繁琐，如果你喜欢使用文件，可以将JSON保存到一个名为caddy.json的文件中，然后使用这个命令：

```bash
curl localhost:2019/load \
  -X POST \
  -H "Content-Type: application/json" \
  -d @caddy.json
```

现在在浏览器打开localhost:2015，或者通过`curl`访问：

```bash
curl localhost:2015
Hello, world!

```

我们还可以通过该JSON文件在不同的端口上定义多个站点：

```javascript
{
	"apps": {
		"http": {
			"servers": {
				"hello": {
					"listen": [":2015"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Hello, world!"
							}]
						}
					]
				},
				"bye": {
					"listen": [":2016"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Goodbye, world!"
							}]
						}
					]
				}
			}
		}
	}
}
```

更新JSON，然后再次执行API请求。

在浏览器或`curl`中尝试一下新的“goodbye”接口，以确保它能正常工作：


```bash
curl localhost:2016
Goodbye, world!
```

当你使用完caddy，可以通过命令停止它：

```bash
caddy stop
```

通过API还可以做很多事情，包括导出配置和对配置进行更细粒度地管理(而不是更新整个配置)。请务必阅读完整的[API教程](../referrence/api.md)以了解如何使用!

## 深度阅读

* [完整API入门](../api-tutorial.md)
* [API文档](../api.md)