# 从源码构建

要求：
- [Go](https://golang.org/doc/install) 1.16 或更新的版本

克隆这个仓库：

```bash
git clone "https://github.com/caddyserver/caddy.git"
```

如果您没有git，您可以从[GitHub](https://github.com/caddyserver/caddy)下载源代码压缩包。每个[版本](https://github.com/caddyserver/caddy/releases)也有源码快照。

构建：

```bash
cd caddy/cmd/caddy/
go build
```

> 由于[Go语言的一个bug](https://github.com/golang/go/issues/29228)，这些基本步骤没有嵌入版本信息。如果您想要版本(`caddy version`)，则需要将Caddy编译为依赖项而不是主模块。这方面的说明在Caddy的[main.go](https://github.com/caddyserver/caddy/blob/master/cmd/caddy/main.go)文件中。当然，您也可以使用[`xcaddy`](https://caddyserver.com/docs/build#xcaddy)自动完成此操作。


## xcaddy

使用[`xcaddy`命令](https://github.com/caddyserver/xcaddy)是构建包含版本信息和/或插件的最简单方法。

要求：

- 安装了Go（见上文）
- 确保[`xcaddy`](https://github.com/caddyserver/xcaddy/releases)在您的`PATH`变量中

你不需要下载Caddy的源码，xcaddy会自动帮你完成。

然后，你可以很简单地构建Caddy（带有版本信息）：

```bash
xcaddy build
```

如果想装上插件，请使用`--with`：

```bash
xcaddy build \
  --with github.com/caddyserver/nginx-adapter
	--with github.com/caddyserver/ntlm-transport@v0.1.1
```

如你所见，可以使用`@`语法自定义插件的版本。版本可以是标签名称、提交SHA或者分支。

跨平台编译`xcaddy`使用相同的`go`命令（见下文）。

## 跨平台

Go 程序很容易为其他平台编译，只需设置不同的`GOOS`、`GOARCH`和/或`GOARM`环境变量即可。（有关详细信息，请参阅[go文档](https://golang.org/doc/install/source#environment)。）

例如，当你在非Windows平台时要编译适用于Windows的Caddy：

```bash
GOOS=windows go build
```

或者当你不在Linux或者ARMv6上时，却要编译Linux ARMv6版本类似：
```bash
GOOS=linux GOARCH=arm GOARM=6 go build
```

同样的方法适用于`xcaddy`。为macOS平台进行交叉编译：

```bash
GOOS=darwin xcaddy build
```

## Debian/Ubuntu/Raspbian 自定义构建的软件包支持文件

此步骤目的在于简化运行自定义`caddy`二进制文件，同时保留`caddy`包中的支持文件。

此步骤允许用户利用官方软件包中的默认配置、systemd服务文件和脚本自动提示。

要求：

- 根据这些[说明](https://caddyserver.com/docs/install#debian-ubuntu-raspbian)安装`caddy`
- 根据本文档中列出的过程构建自定义二进制文件。（往上看）
- 自定义`caddy`二进制文件应位于当前目录中。


步骤：

```bash
dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy
mv ./caddy /usr/bin/caddy.custom
update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10
update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom 50
```

`dpkg-divert`将`/usr/bin/caddy`二进制文件移动到`/usr/bin/caddy.default`，并放置一个转移(diversion)，防止任何包想要将文件安装到这个位置（指`/usr/bin/caddy`）。

`update-alternatives`将从所需的caddy二进制文件创建一个符号链接到`/usr/bin/caddy`。

你可以通过下面的命令在自定义和默认caddy二进制文件之间进行修改

```bash
update-alternatives --config caddy
```

并按照屏幕上的信息进行操作。
