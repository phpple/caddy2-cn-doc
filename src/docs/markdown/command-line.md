# 命令行

Caddy有一个标准的类unix命令行接口，基本用法如下：

```bash
caddy <command> [<args...>]
```

- `<>`代表要被你输入替换参数。
- `[]`代表可选的参数。
- `...`表示延续，即一个或多个参数。

__快速开始：`caddy help`__

- [caddy adapt](#caddy-adapt) 将配置文档适配为原生JSON
- [caddy build-info](#caddy-build-info) 打印构建信息
- [caddy environ](#caddy-environ) 打印环境
- [caddy file-server](#caddy-file-server) 一个简单但可用于生产的文件服务器
- [caddy fmt](#caddy-fmt) 格式化一个 Caddyfile
- [caddy hash-password](#caddy-hash-password) 散列密码并输出 base64
- [caddy help](#caddy-help) 查看 caddy 命令的帮助
- [caddy list-modules](#caddy-list-modules) 列出已安装的 Caddy 模块
- [caddy reload](#caddy-reload) 更改正在运行的 Caddy 进程的配置
- [caddy reverse-proxy](#caddy-reverse-proxy) 一个简单但可用于生产的 HTTP(S) 反向代理
- [caddy run](#caddy-run) 在前台启动 Caddy 进程
- [caddy start](#caddy-start) 在后台启动 Caddy 进程
- [caddy stop](#caddy-stop) 停止正在运行的 Caddy 进程
- [caddy trust](#caddy-trust) 将证书安装到本地信任存储中
- [caddy untrust](#caddy-untrust) 不信任来自本地信任存储的证书
- [caddy upgrade](#caddy-upgrade) 将 Caddy 升级到最新版本
- [caddy add-package](#caddy-add-package) 将 Caddy 升级到最新版本，添加了额外的插件
- [caddy remove-package](#caddy-remove-package) 将 Caddy 升级到最新版本，删除了一些插件
- [caddy validate](#caddy-validate) 测试配置文件是否有效
- [caddy version](#caddy-version) 打印版本

## 子命令

### caddy adapt

```bash
caddy adapt
  [--config <path>]
  [--adapter <name>]
  [--pretty]
  [--validate]
```
将配置适配成Caddy的原生JSON配置结构，并通过stdout输出，如果有任何stderr的警告，则直接退出。

`--config`是配置文件的路径。如果省略，则假定当前目录存在`Caddyfile`文件；否则，必须指定该选项。

`--adapter`指定要使用的配置适配器；默认为`caddyfile`。

`--pretty`将使用缩进格式化输出以提高可读性。

`--validate`将加载并提供适配了的配置以检查有效性（但它实际上不会开始运行配置）。

请注意，成功适配了的配置仍可能无法通过验证。例如，使用这个Caddyfile：

```caddy
localhost

tls cert_notexist.pem key_notexist.pem
```
尝试适配它：
```bash
caddy adapt --config Caddyfile
```
它成功了且没有错误，然后运行下面的命令：

```bash
caddy adapt --config Caddyfile --validate
adapt: validation: loading app modules: module name 'tls': provision tls: loading certificates: open cert_notexist.pem: no such file or directory
```

即使Caddyfile可以毫无错误地适配JSON，但实际的证书和/或密钥文件不存在，因此验证失败，因为该错误出现在配置阶段。因此，验证(`--validate`)是比适配更强的错误检查。

#### 例子
要使Caddyfile适配JSON，可以轻松地手动读取和调整：

```bash
caddy adapt--config /path/to/Caddyfile --pretty
```

### caddy build-info

```bash
caddy build-info
```

打印`Go`提供的关于构建的信息（主模块路径、包版本、模块替换）。

### caddy environ
```bash
caddy environ
```

打印 caddy 看到的环境，然后退出。在调试 init 系统或进程管理器单元（如 systemd）时很有用。

### caddy file-server

```bash
caddy file-server
  [--root <path>]
  [--listen <addr>]
  [--domain <example.com>]
  [--browse]
  [--templates]
  [--access-log]
```
启动一个简单但可用于生产的静态文件服务器。

`--root`指定根文件路径。默认为当前工作目录。

`--listen`接受一个监听地址。默认为:80，除非--domain使用，否则:443将是默认值。

`--domain`将仅通过该主机名提供文件，并且 Caddy 将尝试通过 HTTPS 提供文件，因此如果它是公共域名，请确保首先正确配置任何公共 DNS。默认端口将更改为 443。

`--browse`如果请求没有索引文件的目录，将启用目录列表。

`--templates`将启用模板渲染。

`--access`-log启用请求/访问日志。

此命令禁用管理 API，从而更容易在本地开发机器上运行多个实例。

### caddy fmt

```bash
caddy fmt [--overwrite] [<path>]
```
格式化或美化 Caddyfile，然后退出。除非使用，否则结果将打印到标准输出--overwrite。

<path>指定 Caddyfile 的路径。如果-，则从标准输入读取输入。如果省略，则假定为当前目录中名为 Caddyfile 的文件。

`--overwrite`导致结果写入输入文件而不是打印到终端。如果输入不是常规文件，则此标志无效。

### caddy hash-password

```bash
caddy hash-password
  [--plaintext <password>]
  [--algorithm <name>]
  [--salt <string>]
```
散列密码并以 base64 编码将输出写入标准输出，然后退出。

`--plaintext`是密码的明文形式。如果省略，将采用交互模式，并提示用户手动输入密码。

`--algorithm`可能是 bcrypt 或 scrypt。默认为 bcrypt。

`--salt`仅在算法需要外部盐（如 scrypt）时使用。

### caddy help

```bash
caddy help[<command>]
```
打印 CLI 帮助文本，可选择用于特定子命令，然后退出。

### caddy list-modules

```bash
caddy list-modules
  [--packages]
  [--versions]
  [--skip-standard]
```
打印已安装的 Caddy 模块，可选择包含来自其关联 Go 模块的包和/或版本信息，然后退出。

在某些脚本化的情况下，打印所有标准模块可能是多余的，因此你可以使用--skip-standard从输出中省略那些。

注意：由于Go 中的一个错误，版本信息仅在 Caddy 构建为依赖项而不是主模块时可用。使用xcaddy使这更容易。

### caddy reload

```bash
caddy reload
  [--config <path>]
  [--adapter <name>]
  [--address <interface>]
  [--force]
```
为正在运行的 Caddy 实例提供新配置。这与将文档发布到/load 端点具有相同的效果，但是此命令对于围绕配置文件的简单工作流很方便。与stop、start和run命令相比，这个单一命令是更改/重新加载运行配置的正确语义方式。

由于此命令使用 API，因此不得禁用管理端点。

`--config`是要应用的配置文件。如果-，则从标准输入读取配置。如果未指定，它将尝试Caddyfile在当前工作目录中调用的文件，如果存在，它将使用caddyfile配置适配器对其进行调整；否则，如果没有要加载的配置文件，则会出错。

`--adapter`指定要使用的配置适配器（如果有）。

`--address`如果管理端点没有监听默认地址并且它与提供的配置文件中的地址不同，则需要使用。请注意，此时仅支持 TCP 地址。

`--force`即使指定的配置与 Caddy 已经在运行的配置相同，也会导致重新加载。强制 Caddy 重新配置其模块可能很有用，这可能会产生副作用，例如：重新加载手动加载的 TLS 证书。

### caddy reverse-proxy

```bash
caddy reverse-proxy
  [--from <addr>]
`--to` <addr>
  [--change-host-header]
```
启动一个简单但可用于生产的 HTTP(S) 反向代理。

`--from`是代理的地址。

`--to`是要代理的地址。

`--change`-host-header将导致 Caddy 将 Host 标头从传入值更改为上游的地址。

`--from`和参数都--to可以是 URL，因为方案和域名将从提供的 URL 中推断出来（路径和查询字符串被忽略）。或者它们可以是简单的网络地址而不是完整的 URL。

此命令禁用管理 API，因此更容易在本地开发机器上运行多个实例。

### caddy run

```bash
caddy run
  [--config <path>]
  [--adapter <name>]
  [--pidfile <file>]
  [--environ]
  [--envfile <file>]
  [--resume]
  [--watch]
```

使用“守护进程”模式无限期地运行Caddy。

`--config`指定要立即加载和使用的初始配置文件。如果指定为`-`，则从标准输入读取配置。如果未指定配置，Caddy将以空白配置运行，并使用管理API端点的默认设置，可用于为其提供新配置。作为一种特殊情况，如果当前工作目录有一个名为“Caddyfile”的文件并且配置了`caddyfile`适配器（默认），那么即使没有任何命令行标志，该文件也将被加载并用于配置Caddy。

`--adapter`是加载初始配置时使用的配置适配器的名称（如果有）。如果`--config`指定的文件名以“Caddyfile”开头，则已经假定使用`caddyfile`作为适配器，则不需要此标志。否则，如果提供的配置文件不是Caddy的原生JSON格式，则需要此标志。任何警告都将打印到日志中，但请注意，任何没有错误的适配都会立即被使用，即使有警告也是如此。如果要先查看适配结果，请使用[`caddy adapt](#caddy-adapt)子命令。

`--pidfile`将PID写入指定文件。

`--environ`在开始之前打印出环境。这与[`caddy environ`](#caddy-environ)命令相同，但打印后不退出。

`--envfile`从指定文件加载环境变量。

`--resume`使用自动保存的最后加载的配置，覆盖`--config`标志（如果存在）。使用此标志可通过机器重新启动或进程重新启动来保证配置的持久性。它在以[API](api)为中心的部署中最有用。

`--watch`将监视配置文件并在更改后自动重新加载它。
⚠️此功能仅供本地开发环境使用！

> 在生产中运行时不要停止服务器来更改配置！这将导致停机。（这应该很明显，但你会惊讶于我们收到了多少关于它的投诉。）请改用[`caddy reload`](#caddy-reload)命令。

### caddy start

```bash
caddy start
  [--config <path>]
  [--adapter <name>]
  [--envfile <file>]
  [--pidfile <file>]
  [--watch]
```
与`caddy run`相同，但该命令在后台运行。此命令仅在后台进程运行成功（或运行失败）之前阻塞，然后返回。

注意：该标志`--config`不支持通过`-`选项从标准输入读取配置。

不鼓励在系统服务或Windows上使用此命令。在Windows上，子进程将保持连接到终端，因此关闭窗口将强制停止Caddy，这并不明显。考虑改为将Caddy[作为服务](/docs/running)运行。

启动后，你可以使用[`caddy stop`](#caddy-stop)或者[`/ stop`](api#post-stop)API端点退出后台进程。

### caddy stop

```bash
caddy stop[--address <interface>]
```

> 停止（和重新启动）服务器与配置更改正交。__不要使用stop命令更改生产中的配置，除非你想要停机__。请改用[caddy reload](#caddy-reload)命令。

优雅地停止正在运行的Caddy进程（而不是直接停止进程）并使其退出。它使用管理API的[`POST /stop`](api#post-stop)端点来执行平滑关闭。

`--address` 如果正在运行的实例的管理API不在默认端口上，则可以使用；也可以在此处指定备用地址。

如果要停止当前配置但不想退出进程，请使用`caddy reload`空白配置或[`DELETE /config/`](api#delete-configpath)端点。

### caddy trust

```bash
caddy trust
```
将Caddy的默认内部CA（名为“local”）的根证书安装到本地信任库中；仅用于开发环境。如果没有足够的权限，可能会提示输入密码。

这个命令通常是不必要的。因为Caddy将在第一次需要时自动将其根证书安装到本地信任存储中，所以此命令仅在你需要在具有提升的权限时预安装证书时有用，例如在自动化环境中的系统配置期间。

### caddy untrust

```bash
caddy untrust
  [--ca <id>]
  [--cert <path>]
```
不信任来自本地信任存储的根证书。仅用于开发环境。可以分别指定`--ca`或`--cert`标志，但不能同时指定两者。如果两者均未指定，则为Caddy的默认CA(`local`)。

`--ca`指定不信任的Caddy CA的ID。默认的CA的ID是`local`。

`--cert`指定要卸载的PEM编码证书文件的路径。

### caddy upgrade

```bash
caddy upgrade
  [--keep-backup]
```
将当前的Caddy二进制文件替换为[我们下载页面](https://caddyserver.com/download)中安装了相同模块的最新版本，包括在Caddy网站上注册的所有第三方插件。

升级不会中断正在运行的服务器；目前，该命令仅替换磁盘上的二进制文件。如果我们能找到更好的方法，这种升级模式可能会被改变。

这种升级过程是容错的；当前二进制文件首先会被备份（在当前二进制文件复制出来一份到同目录）并在出现任何问题时自动恢复。如果你希望在升级过程完成后保留备份，你可以使用`--keep-backup`选项。

该命令执行时如果你的用户无权写入可执行文件，则此需要提升权限。

### caddy add-package

```bash
caddy add-package <packages...>
  [--keep-backup]
```
与`caddy upgrade`类似，将当前Caddy二进制文件替换为安装了相同模块的最新版本，另外会安装通过参数指定的包。从我们的[下载页面](https://caddyserver.com/download)找到你可以安装的软件包列表。每个参数都应该是完整的包名。

例如：
```bash
caddy add-package github.com/caddy-dns/cloudflare
```

### caddy remove-package

```bash
caddy remove-package <packages...>
  [--keep-backup]
```
与`caddy upgrade`命令类似，将当前Caddy二进制文件替换为安装了相同模块的最新版本，且会将通过参数列出的包删除掉。运行`caddy list-modules --packages`可以查看当前二进制文件中包含的非标准模块的包名列表。

### caddy validate

```bash
caddy validate
  [--config <path>]
  [--adapter <name>]
```
验证配置文件，然后退出。此命令将反序列化配置，然后加载和配置其所有模块，就好像启动配置一样（但实际上并未启动配置）。这会把加载或配置阶段出现的配置错误暴露出来，是比仅将配置序列化为JSON更强大的错误检查方式。

`--config`是要验证的配置文件。如果指定该选项为`-`，则从标准输入stdin读取配置。默认使用当前目录下的`Caddyfile`，前提是这个文件存在。

如果配置文件不是Caddy的原生JSON格式，`--adapter`是要使用的配置适配器的名称。如果配置文件以`Caddyfile`开头，则默认使用`caddyfile`这个适配器。

### caddy version

```bash
caddy version
```
打印版本并退出。