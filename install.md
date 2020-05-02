# 安装

每个平台都可以使用Caddy的静态二进制文件(它没有依赖项)。您还可以从源代码构建以定制你的Caddy。

## 官方包

我们为以下平台提供官方发行版本s：

### Docker

```bash
docker pull caddy
```

[在Docker Hub上查看](https://hub.docker.com/_/caddy)

### Debian, Ubuntu, Raspbian

```bash
echo "deb [trusted=yes] https://apt.fury.io/caddy/ /" \
    | sudo tee -a /etc/apt/sources.list.d/caddy-fury.list
sudo apt update
sudo apt install caddy
```

### Fedora, RedHat, CentOS

查看如何[安装Caddy COPR](https://copr.fedorainfracloud.org/coprs/g/caddy/caddy/)。

### DigitalOcean

[创建Caddy droplet](https://marketplace.digitalocean.com/apps/caddy)，90s内开始启动。

## Linux服务

本节描述如何将Caddy作为Linux服务手动安装。

需求：
* 下载或者从源码构建的`caddy`
* `systemctl --version`版本至少为232
* `sudo`权限

将caddy二进制文件移到到`$PATH`所在目录，例如:
```bash
$ sudo mv caddy /usr/bin
```

测试一下是否ok：

```bash
caddy version
```

创建一个`caddy`的组

```bash
$ groupadd --system caddy
```

创建一个名字为`caddy`的用户，确保在家目录有写的权限。

```bash
useradd --system \
	--gid caddy \
	--create-home \
	--home-dir /var/lib/caddy \
	--shell /usr/sbin/nologin \
	--comment "Caddy web server" \
	caddy
```

接下来，按照使用场景创建对应的[systemd服务文件](https://github.com/caddyserver/dist/blob/master/init)。
* 如果使用一个文件来配置Caddy，则取名`caddy.service`
* 如果仅仅通过API配置Caddy，则取名`caddy-api.servie`

它们非常相似，不同点主要在于ExecStart和ExecReload命令，以适应你的工作流，需要相应地定制文件。

再次检查ExecStart和ExecReload指令，确保二进制文件的位置和命令行参数和你的安装是一致的!

通常保存服务文件的位置是：`/etc/systemd/system/caddy.service`。

保存你的服务文件后，可以用通常的systemctl命令完成它的第一次启动：

```bash
sudo systemctl daemon-reload
sudo systemctl enable caddy
sudo systemctl start caddy
```

验证一下是否运行成功。

```bash
systemctl status caddy
```

当运行我们的官方服务文件时，Caddy的输出将被重定向到`journalctl`：

```bash
journalctl -u caddy
```

如果使用配置文件，你可以优雅地应用任何变化：

```bash
sudo systemctl reload caddy
```

你可以使用下面的命令停止Caddy：

```bash
sudo systemctl stop caddy
```

> 如果要改变Caddy的配置，不要直接停止服务，这将导致服务终止。正确的做法是使用reload命令。

## 从源代码构建

需求：
* [Go](https://golang.org/dl)语言版本>=1.14
* 支持[Go Module](https://github.com/golang/go/wiki/Modules)功能

下载源代码：

```bash
git clone "https://github.com/caddyserver/caddy.git"
```

构建：

```bash
cd caddy/cmd/caddy/
go build
```

### 使用插件

使用[xcaddy](https://github.com/caddyserver/xcaddy)，你可以给Caddy编译额外的插件，例如：

```bash
xcaddy build \
    --with github.com/caddyserver/nginx-adapter
	--with github.com/caddyserver/ntlm-transport@v0.1.0
```


