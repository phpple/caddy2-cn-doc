---
title: 约定
---

# 约定

Caddy生态系统遵循一些约定，以使整个平台的事情保持一致和直观。


## 网络地址

指定要拨号或绑定的网络地址时，Caddy接受以下格式的字符串：

```
network/address
```

网络部分是可选的，是[Go的`net`包](https://golang.org/pkg/net/)可以识别的任何内容。默认网络是`tcp`。如果指定了网络，则必须用单个正斜杠`/`分隔网络和地址部分。

地址部分可以是以下任何一种形式：

- `host`
- `host:port`
- `:port`
- `/path/to/unix/socket`

主机可以是任何主机名、可解析的域名或IP地址。

端口可以是单个值 ( `:8080`)或包含范围(`:8080-8085`))。端口范围将乘以单个地址。并非所有配置字段都接受端口范围。特殊端口`:0`是指任何可用的端口。

仅当使用 unix* 网络类型时，unix 套接字路径才可接受。分隔网络和地址的正斜杠不被视为路径的一部分。

有效示例：

```
:8080
127.0.0.1:8080
localhost:8080
localhost:8080-8085
tcp/localhost:8080
tcp/localhost:8080-8085
udp/localhost:9005
unix//path/to/socket
```

<aside class="tip">
	Caddy网络地址不是URL。URL将<a href="https://en.wikipedia.org/wiki/OSI_model#Layer_architecture">OSI模型</a>的较低层和较高层耦合在一起，但Caddy经常使用独立于特定应用程序的网络地址，因此将它们组合起来会有问题。在Caddy中，网络地址就是指L3-L5可以拨号或绑定的资源，但是URL极大地结合了L3-L7。网络地址要求主机+端口和路径互斥，但URL不需要。网络地址有时支持端口范围，但URL不支持。
</aside>


## 占位符

Caddy 的配置支持使用 _占位符_ （变量）。使用占位符是一种将动态值注入静态配置的简单方法。

<aside class="tip">
	占位符与其他软件中的变量类似。例如，<a href="https://nginx.org/en/docs/varindex.html">nginx有</a>像`$uri`和`$document_root`这样的变量。
</aside>

占位符的两边都用花括号括起来`{ }`，里面包含变量名，例如：`{foo.bar}`。占位符大括号可以转义，如`\{like so\}`。变量名通常用点命名，以避免模块之间的冲突。

哪些占位符可用取决于上下文。并非所有占位符在配置的所有部分都可用。例如，[HTTP应用程序设置的占位符](/docs/json/apps/http/)仅在与处理 HTTP 请求相关的配置区域中可用。

以下占位符始终可用：

占位符 | 描述
------------|-------------
`{env.*}` | 环境变量（例如`{env.HOME}`）
`{system.hostname}` | 系统的本地主机名
`{system.slash}` | 系统的文件路径分隔符
`{system.os}` | 系统的操作系统
`{system.arch}` | 系统架构
`{time.now}` | Go时间结构的当前时间
`{time.now.unix}` | 当前时间，以秒为单位的unix时间戳
`{time.now.unix_ms}` | 当前时间，以毫秒为单位的unix时间戳
`{time.now.common_log}` | 通用日志格式的当前时间
`{time.now.year}` | YYYY格式的当前年份

并非所有配置字段都支持占位符，但大多数都支持你期望的位置。


## 文件位置

本节包含有关在何处查找各种文件的信息。此处描述的文件和目录路径充其量是默认值；有些可以被覆盖。

### 你的配置文件

没有一个单一的、传统的地方可以放置你的配置文件。将它们放在对你最有意义的地方。

<aside class="tip">
	唯一的例外可能是当前工作目录中名为“Caddyfile”的文件，如果未指定其他配置文件，则为方便起见，caddy 命令会尝试该文件。
</aside>

带有默认配置文件的发行版应该记录这个配置文件的位置，即使它对包/发行版维护者来说是显而易见的。

### 数据目录

Caddy 将 TLS 证书和其他重要资产存储在数据目录中，该目录由[配置的存储模块](/docs/json/storage/)支持（默认：本地文件系统）。

如果`XDG_DATA_HOME`设置了环境变量，则为`$XDG_DATA_HOME/caddy`。

否则，它的路径因平台而异，遵守操作系统约定：

操作系统 | 数据目录路径
---|---------------------
**Linux, BSD** | `$HOME/.local/share/caddy`
**Windows** | `%AppData%\Caddy`
**macOS** | `$HOME/Library/Application Support/Caddy`
**Plan 9** | `$HOME/lib/caddy`
**Android** | `$HOME/caddy`（或`/sdcard/caddy`）

所有其他操作系统都使用 Linux/BSD 目录路径。

**数据目录不能被视为缓存。** 它的内容**不是**短暂的，也不是仅仅为了表演。Caddy 将 TLS 证书、私钥、OCSP 订书钉和其他必要信息存储到数据目录中。在不了解其含义的情况下，不应将其清除。

至关重要的是，这个目录是持久的并且可由Caddy写入。


### 配置目录

这是 Caddy 可以将某些配置存储到磁盘的地方。最值得注意的是，它将最后一个活动配置（默认情况下）保存到此文件夹，以便以后使用[`caddy run --resume`](/docs/command-line#caddy-run)。

<aside class="tip">
	配置目录不是你需要存储<a href="#your-config-files">配置文件</a>的地方。（不过，你可以这样做。）
</aside>

如果`XDG_CONFIG_HOME`设置了环境变量，则为`$XDG_CONFIG_HOME/caddy`。

否则，它的路径因平台而异，遵守操作系统约定：


操作系统 | 配置目录路径
---|---------------------
**Linux, BSD** | `$HOME/.config/caddy`
**Windows** | `%AppData%\Caddy`
**macOS** | `$HOME/Library/Application Support/Caddy`
**Plan 9** | `$HOME/lib/caddy`

所有其他操作系统都使用 Linux/BSD 目录路径。

至关重要的是，这个目录是持久的并且可由Caddy写入。


## 持续时间

持续时间字符串通常在Caddy的配置中使用。它们采用与[Go的`time.ParseDuration`语法](https://golang.org/pkg/time/#ParseDuration)，除此之外，你还能`d`表示一天（为简单起见，我们假定1天=24小时)。有效单位是：

- `ns` (纳秒)
- `us`/`µs` (微秒)
- `ms` (毫秒)
- `s` (秒)
- `m` (分)
- `h` (小时)
- `d` (天)

例子：

- `250ms`
- `5s`
- `1.5h`
- `2h45m`
- `90d`

在[JSON配置](/docs/json/)中，持续时间值也可以是表示纳秒的整数。
