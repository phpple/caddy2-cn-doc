# 入门指南

欢迎使用Caddy！本教程将探索使用Caddy的基础知识，并帮助你在更高的层次上熟悉它。

目标：

1. 运行demo
2. 使用API
3. 对Caddy进行配置
4. 测试配置
5. 制作Caddyfile
6. 使用配置适配器
7. 通过初始化配置启动
8. 比较JSON和Caddyfile
9. 比较API和配置文件
10. 后台运行
11. 平滑重启

前提：

* 基本的终端/命令行知识
* 基本的文本编辑器知识
* `caddy`和`curl`命令可以运行

我们开始运行它：

```bash
caddy
```

哈，没有第2个命令，caddy命令只是显示帮助内容。你忘记如何使用时使用该命令即可。

要让caddy运行一个demo，使用这个子命令：

```caddy
caddy run
```

> 目标1完成：运行demo


它以阻塞的方式运行，但它在做什么呢？此刻……什么都没有。默认情况下，Caddy的配置("config")是空的。我们这在另一个终端使用管理API验证一下：

```bash
curl localhost:2019/config/
```

> 目标2完成：尝试API

我们可以通过给它一个配置使它变得有用。这可以通过多种方式实现，但是我们将在下一节中使用curl向`/load`网址发出POST请求。


## 你的第一个配置

为了完成该请求，我们需要做一个配置。Caddy的核心配置只是一个JSON文档。

把下面的内容保存成一个JSON文件：

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

你不需要使用文件进行配置。管理API总是可以在没有文件的情况下使用，这在自动化管理时非常方便。
把刚才的文件上传：

```bash
curl localhost:2019/load \
	-X POST \
	-H "Content-Type: application/json" \
	-d @caddy.json
```

> 任务3完成：对Caddy进行配置

我们使用另一个GET请求验证Caddy是否应用了我们的新配置：

```bash
curl localhost:2019/config/
```

在浏览器中打开localhost:2015或使用curl测试它是否符合预期：

```bash
curl localhost:2015
Hello, world!
```

如果你看到"Hello, world!"，那么恭喜你——配置起作用了！ 确保配置按照预期工作总是一个好的习惯，尤其是在部署到生产环境之前。

> 任务4完成：测试配置

## 你的第一个Caddyfile

要让"Hello World"运行出来，实际上有好多工作要做。

配置Caddy的另一种方法是使用Caddyfile。我们在JSON中编写的配置可以简单地表示为：

```caddy
:2015

respond "Hello, world!"
```
将其保存到当前目录中的文件，命名为Caddyfile(没有扩展名)。

> 任务5完成：制作Caddyfile

如果Caddy已经运行，停止它（Ctrl+C），然后运行：

```bash
caddy adapt
```

或者如果你把Caddyfile存储在其他地方，或者把它命名为Caddyfile以外的东西：

```bash
caddy adapt --config /path/to/Caddyfile
```

你将看到JSON输出！这里发生了什么？

我们只是使用了一个配置适配器来将我们的Caddyfile转换成Caddy的原生JSON结构。


> 任务6完成：使用配置适配器

虽然我们可以获取输出并发出另一个API请求，但我们可以跳过所有这些步骤，因为caddy命令可以为我们完成这些工作。如果当前目录中有一个名为Caddyfile的文件，并且没有指定其他配置，那么Caddy将加载Caddyfile，为我们调整它，并立即运行它。

现在有一个Caddyfile在当前文件夹，让我们再次运行`caddy run`：

```bash
caddy run
```

如果Caddyfile放在别的位置，则可以运行：

```bash
caddy run --config /path/to/Caddyfile
```
（如果它被调用的文件名称不是“Caddyfile”开头的，则需要指定`--adapter caddyfile`。）

你现在可以再次尝试加载你的网站，你会看到它是工作！

> 任务7完成：通过初始化配置启动

正如你所看到的，有几种方法可以让你用一个初始配置启动Caddy：
1. `--config`选项（也可以通过`--adapter`）
2. `--resume`选项（如果之前加载了一个配置）


## JSON vs. Caddyfile
现在你知道了Caddyfile只是被转换成了JSON。

Caddyfile似乎比JSON更简单，但你应该总是使用它吗？每种方法都有利弊，答案取决于你的需求和使用场景。
 
| JSON                    |  Caddyfile            |
|:-------------------------|:----------------------|
| 通用的                    | 只在Caddy使用          |
| 易于生成                  | 易于手写                |
| 易于编程                  | 难以实现自动化           |
| caddy的所有功能           | caddy大部分通用的功能     |
| 极富表现力                | 适度表现力               |
| 允许配置遍历               | 不能再Caddyfile内遍历   |
| 允许变更部分配置           | 只允许变更全部配置         |    
| 可以被导出                | 不能被导出               |
| 支持所有API接口            | 只支持部分API接口        |
| 自动生成文档               | 文档支持手写             |
| 更高效                    | 更易计算                |
| 有点无聊                   | 更有意思                |
| 了解更多：[JSON结构](referrence/json-config-structure.md) | 了解更多：[Caddyfile文档](tutorials:caddyfile.md)  |

你需要根据场景选择哪种方式更适合你。

需要注意的是，JSON和Caddyfile(以及任何其他被支持的配置适配器)都可以与Caddy的API一起使用。但是，如果使用JSON，则可以获得完整的Caddy功能和API特性。如果使用配置适配器，使用API加载或更改配置的惟一方法是`/load`接口。

> 任务8完成：比较JSON和Caddyfile


## API vs. 配置文件

> 在底层，配置文件也要经过Caddy的API接口；caddy命令只是为你包装那些API调用。

你还需要决定哪些工作流是基于api的，哪些是基于命令的。(你可以在同一台服务器上同时使用API和配置文件，但我们不建议这样做：最好只有一种方式。)


| API	| 配置文件 |
|:-------|:-------------|
| 通过HTTP请求更改配置  |  通过shell命令更改配置 |
| 易于扩展  |  难于扩展  |
| 很难手工操作 | 易于手工操作 |
| 很有趣  | 也很有趣 |
| 了解更多：[API教程](referrence/api.md) | 了解更多：[Caddyfile教程]（tutorials/caddyfile.md） |

> 使用适当的工具(例如:任何REST客户端应用程序)完全可以使用API手动管理服务器的配置。

API或配置文件工作流的选择与配置适配器的使用是可以交替进行的：你可以使用JSON，但将其存储在文件中并使用命令行接口；相反，你也可以在API中使用Caddyfile。

但大多数人会使用JSON+API或Caddyfile+CLI组合。

正如您所看到的，Caddy非常适合各种场景和部署！

> 任务9完成：比较API和配置文件

## 启动，停止，运行

由于Caddy是服务器，所以它会一直运行。这意味着在执行`caddy run`之后，直到进程终止(通常使用Ctrl+C)，你的终端才会被释放出来。

虽然`caddy run`是最常见的，通常被推荐(特别是在做系统服务的时候!)，你也可以使用`caddy start`来启动caddy，让它在后台运行：

```bash
caddy start
```

这将使你能继续使用你的终端，这在一些互动无头的环境中非常方便。

当然，`Ctrl+C`不能再帮你停止caddy了，你需要自己去停止它：

```bash
caddy stop
```

或者使用API的`/stop`接口停止服务。

> 任务10完成：后台运行


## 重载配置
你的服务器可以执行不停机加载/更改配置。

所有用来加载或者更改配置的API接口都是优雅地，无需停止服务。

在使用命令行时，可能很容易使用`Ctrl+C`来停止服务器，然后重新启动它，以获得新的配置。但是不要这样做：停止和启动服务器与配置更改是正交的，并且会导致停服。

> 停止服务器将导致服务停止。

相反地，使用`caddy reload`命令可以进行优雅的配置更改：
```bash
caddy reload
```

这实际上只是使用了底层的API。它将加载并在必要时将配置文件调整为JSON格式，然后在没有停机的情况下优雅地替换正在使用的配置。

如果加载新配置时出现任何错误，Caddy将回滚到最后一个有效的配置。

从技术上讲，新配置是在旧配置停止之前启动的，所以在短时间内，两个配置都在运行！如果新配置失败，它会带来一个错误中止，而旧的配置则不会停止。

> 任务11完成：平滑重启
