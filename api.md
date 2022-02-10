# API快速入门

__先决条件：__

- 基本的终端/命令行技能
- `PATH`环境变量支持`caddy`和`curl`

首先启动Caddy：

```bash
caddy start
```

Caddy现在处于空闲状态（配置为空白）。通过`curl`可以给它加上一个简单的配置：

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

使用[Heredoc](https://en.wikipedia.org/wiki/Here_document#Unix_shells)提供POST包体可能很乏味，因此，如果你更喜欢使用文件，请将JSON保存成`caddy.json`的文件，然后改用以下命令：

```bash
curl localhost:2019/load \
  -X POST \
  -H "Content-Type: application/json" \
  -d @caddy.json
```

现在在浏览器中访问<localhost:2015>，或者通过`curl`调用：

```bash
curl localhost:2015
Hello, world!
```

我们还可以使用这个JSON在不同的端口上定义多个站点：

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

更新您的 JSON，然后再次执行 API 请求。

在[浏览器中](http://localhost:2016/)尝试新的“bye”端点，或使用`curl`以确保它有效：

```bash
curl localhost:2016
Goodbye, world!
```

当你使用完Caddy后，请务必停止它：

```bash
caddy stop
```

使用API还可以做更多事情，包括导出配置和对配置进行细粒度地更改（而不是更新整个内容）。请务必阅读[完整的 API教程](https://caddyserver.com/docs/api-tutorial)，了解如何进行具体的操作！

## 进一步阅读
- [完整的API教程](https://caddyserver.com/docs/api-tutorial)
- [API文档](https://caddyserver.com/docs/api)
