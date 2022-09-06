---
title: 保持Caddy运行
---

# 保持Caddy运行

虽然Caddy可以通过直接使用它的[命令行界面](/docs/command-line)成功运行，但使用服务管理器来保持它的运行有很多好处，例如确保它在系统启动时重新启动，并捕捉stdout/stderr日志。

- [Linux服务](#linux-service)
  - [单元文件](#unit-files)
  - [使用服务](#using-the-service)
  - [手动安装](#manual-installation)
  - [重写](#overrides)
- [Windows服务](#windows-service)
- [Docker Compose](#docker-compose)


<h2 id="linux-service">Linux服务</h2>

推荐使用我们的官方systemd单元文件，在带有systemd的Linux发行版上运行Caddy。

<h3 id="unit-files">单元文件</h3>

我们提供了两个不同的systemd单元文件，你可以根据你的使用情况来选择:

- 使用[Caddyfile]（/docs/caddyfile）配置Caddy：[**`caddy.service`**](https://github.com/caddyserver/dist/blob/master/init/caddy.service)。如果你喜欢使用JSON配置文件，你可以[覆盖](#overrides) `ExecStart`和`ExecReload`命令。

- 通过[API](/docs/api)配置Caddy：[**`caddy-api.service`**](https://github.com/caddyserver/dist/blob/master/init/caddy-api.service)。 这项服务使用[`--resume`](/docs/command-line#caddy-run)选项，它将使用默认[保存(persisted)](/docs/json/admin/config/)的`autosave.json`启动Caddy。

它们非常相似，但在`ExecStart`和`ExecReload`命令中有所不同，以适应工作流程。

如果需要在服务之间切换，应该先禁用和停止前一个服务，然后再启用和启动另一个服务。例如，要从`caddy`服务切换到`caddy-api`服务：

```bash
sudo systemctl disable --now caddy
sudo systemctl enable --now caddy-api
```


<h3 id="using-the-service">使用服务</h3>

如果使用Caddyfile，你可以用`nano`，`vi`或者你喜欢的编辑器来编辑你的配置：
```bash
sudo nano /etc/caddy/Caddyfile
```
你可以把你的静态站点文件放在`/var/www/html`或`/srv`中。确保caddy用户有读取文件的权限。

查询服务是否正在运行：
```bash
systemctl status caddy
```

status命令还将显示当前运行的服务文件的位置。

当使用我们的官方服务文件运行时，Caddy的输出将被重定向到`journalctl`。为了阅读你的完整日志，避免行数被截断：
```bash
journalctl -u caddy --no-pager | less +G
```

如果使用一个配置文件，你可以在做任何改变后优雅地重新加载Caddy：
```bash
sudo systemctl reload caddy
```

你可以使用下面的命令停止服务：
```bash
sudo systemctl stop caddy
```

<aside class="advice">
	不要直接停止服务然后再来修改Caddy的配置，因为这会产生停机时间。请使用reload命令。
</aside>

Caddy进程将作为`caddy`用户运行，它的`$HOME`设置为`/var/lib/caddy`。这意味着。
- 默认的[数据存储位置]（/docs/conventions#data-directory）（用于证书和其他状态信息）将在`/var/lib/caddy/.local/share/caddy`。
- 默认的[配置存储位置](/docs/conventions#configuration-directory)(用于自动保存的JSON配置，主要对`caddy-api`服务有用)将在`/var/lib/caddy/.config/caddy`。


<h3 id="manual-installation">手动安装</h3>

一些[安装方法](/docs/install)自动设置Caddy作为服务运行。如果你选择的方法没有这样做，你可以按照这些指示来做。

**要求：**

- [下载](/download)或[从源代码构建](/docs/build)的`caddy`二进制文件
- `systemctl --version`版本至少为232
- `sudo`权限

把caddy二进制文件移到`$PATH`目录，比如：
```bash
sudo mv caddy /usr/bin/
```

检查是否能正常工作：
```bash
caddy version
```

创建一个叫`caddy`的群组：
```bash
sudo groupadd --system caddy
```

创建一个有可写权限家目录的`caddy`用户：
```bash
sudo useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy
```

如果有配置文件，需要确保`caddy`用户具有刻度权限。

接下来，选择一个满足使用需求的[systemd单元文件](#unit-files)。

**仔细检查`ExecStart`和`ExecReload`指令。** 确保二进制文件的位置和命令行参数对你的安装来说是正确的！例如：如果使用一个配置文件，如果你的`--config`路径与默认值不同，请改变它。

通常保存服务文件的地方是： `/etc/systemd/system/caddy.service`。

保存好服务文件后，你可以用通常的systemctl方式来首次启动服务。

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now caddy
```

验证它是否正在运行。
```bash
systemctl status caddy
```

现在，你已经准备好[使用服务](#using-the-service)了


<h3 id="overrides">覆盖</h3>

覆盖服务文件各个方面的最好方法是使用这个命令：
```bash
sudo systemctl edit caddy
```

这将用你的默认终端文本编辑器打开一个空白文件，你可以在其中覆盖或添加指令到单元定义。这被称为"插入式"文件。

例如，如果你需要定义环境变量用于你的配置，你可以这样做：
```
```systemd
[Service]
Environment="CF_API_TOKEN=super-secret-cloudflare-tokenvalue"
```

或者，例如你需要将配置文件从Caddyfile的默认值改为使用JSON文件（注意`Exec*`指令[必须在设置新值之前用空字符串重置](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=))。

```systemd
[Service]
ExecStart=
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/caddy.json
ExecReload=
ExecReload=/usr/bin/caddy reload --config /etc/caddy/caddy.json
```

然后，保存文件并退出文本编辑器，并重新启动服务使之生效：
```bash
sudo systemctl restart caddy
```


<h2 id="windows-service">Windows服务</h2>

用这些说明在Windows上把Caddy安装成一个服务。

**要求：**

- [下载](/download)或[从源代码构建](/docs/build)的`caddy.exe`二进制文件。
- 任何来自[WinSW](https://github.com/winsw/winsw/releases/latest)服务封装器（以下服务配置是为v2.x版本编写的）最新版本的`.exe`。

将所有文件放入一个服务目录。在下面的例子中，我们使用`C:\caddy`。

将`WinSW-x64.exe`文件重命名为`caddy-service.exe`。

在同一目录下添加一个`caddy-service.xml`。

```xml
<service>
  <id>caddy</id>
  <!-- 服务的显示名称 -->
  <name>Caddy Web Server (powered by WinSW)</name>
  <!-- 服务描述 -->
  <description>Caddy Web Server (https://caddyserver.com/)</description>
  <executable>%BASE%\caddy.exe</executable>
  <arguments>run</arguments>
  <log mode="roll-by-time">
    <pattern>yyyy-MM-dd</pattern>
  </log>
</service>
```

现在你可以用以下方式安装该服务：
```bash
caddy-service install
```

你可能想启动Windows服务控制台，看看该服务是否正确运行：
```bash
services.msc
```

请注意，Windows服务不能被重新加载，所以你必须直接告诉caddy进行重新加载：
```bash
caddy reload
```

重启可以通过正常的Windows服务命令进行，例如通过任务管理器的 "服务 "标签。

关于定制服务包装器，请参见[WinSW文档](https://github.com/winsw/winsw/tree/master#usage)。


<h2 id="docker-compose">Docker Compose</h2>

使用Docker启动和运行的最简单方法是使用Docker Compose。_以下只是摘录，更多细节请参见[Docker Hub](https://hub.docker.com/_/caddy)上的文档_。

首先，创建一个文件`docker-compose.yml`（或者把这个服务添加到你现有的文件中）：

```yaml
version: "3.7"

services:
  caddy:
    image: caddy:<version>
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - $PWD/site:/srv
      - caddy_data:/data
      - caddy_config:/config

volumes:
  caddy_data:
  caddy_config:
```

确保在`<version>`中填写最新的版本号，你可以在[Docker Hub](https://hub.docker.com/_/caddy)的 "Tags "部分找到。

然后，在`docker-compose.yml`旁边创建一个名为`Caddyfile`的文件，并写下你的[Caddyfile](/docs/caddyfile/concepts)配置。

如果你有静态文件需要服务，你可以把它们放在配置旁边的`site/`目录中，然后把[`root`指令](/docs/caddyfile/directives/root)设为`/srv/`。如果你不这样做，那么你可以删除`/srv`卷装载。

然后，你就可以启动这个容器了：
```bash
docker-compose up -d
```

在对你的Caddy文件进行修改后，要重新加载Caddy：
```bash
docker-compose exec -w /etc/caddy caddy caddy reload
```

查看caddy的日志：
```bash
docker-compose logs caddy
```

