---
title: 安装
---

# 安装

本页介绍了在你的系统上安装Caddy的各种方法。

__官方的：__
- [静态二进制文件](#static-binaries)
- [Debian、Ubuntu、Raspbian](#debian-ubuntu-raspbian)
- [Fedora、Redhat、CentOS](#fedora-redhat-centos)
- [Arch Linux, Manjaro, Parabola](#arch-linux-manjaro-parabola)
- [Docker](#docker)
- [DigitalOcean](#digitalocean)

> 我们的[官方软件包](https://github.com/caddyserver/dist)仅附带标准模块。如果你需要第三方插件，请使用[`xcaddy`从源代码构建](https://caddyserver.com/docs/build#xcaddy)，或者使用我们的[下载页面](https://caddyserver.com/download)。


__社区维护：__
- [Homebrew](#homebrew)
- [Webi](#webi)
- [Chocolatey](#chocolatey)
- [Ansible](#ansible)
- [Scoop](#scoop)
- [Termux](#termux)


<h2 id="static-binaries">静态二进制文件</h2>

只需要简单地下载Caddy二进制文件，并不会[将其安装为服务](/docs/running#manual-installation)，这种方式在开发或升级现有安装时非常有用。

- [在GitHub上查看发布](https://github.com/caddyserver/caddy/releases)（展开“Assets”）
- [使用官方下载页面](https://caddyserver.com/download)

<h2 id="debian-ubuntu-raspbian">Debian、Ubuntu、Raspbian</h2>

安装此软件包会自动启动将Caddy作为systemd服务（名称为`caddy`）运行，另外，还有一个名为`caddy-api`可供使用，它默认没有被启用。

稳定版本：

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

测试版本（包括 beta 和候选版本）：

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/testing/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-testing-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/testing/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-testing.list
sudo apt update
sudo apt install caddy
```

[查看Cloudsmith仓库](https://cloudsmith.io/~caddy/repos/)

如果你希望将包的支持文件（systemd 服务、bash 完成和默认配置）与自定义Caddy一起构建，可以在[这里](/docs/build#package-support-files-for-custom-builds-for-debianubunturaspbian)找到相关说明。

<h2 id="fedora-redhat-centos">Fedora、Redhat、CentOS</h2>

这个包附带了 Caddy 的两个[systemd服务](/docs/running#linux-service)单元文件，但默认情况下不启用它们。

Fedora 或 RHEL/CentOS 8：

```bash
dnf install 'dnf-command(copr)'
dnf copr enable @caddy/caddy
dnf install caddy
```

RHEL/CentOS 7：

```bash
yum install yum-plugin-copr
yum copr enable @caddy/caddy
yum install caddy
```

[查看COPR](https://copr.fedorainfracloud.org/coprs/g/caddy/caddy/)


<h2 id="arch-linux-manjaro-parabola">Arch Linux, Manjaro, Parabola</h2>

这个包附带了 Caddy 的两个[systemd服务](/docs/running#linux-service)单元文件，但默认情况下不启用它们。

```bash
pacman -Syu caddy
```

[查看Arch Linux 仓库](https://archlinux.org/packages/community/x86_64/caddy/)

## Docker

```bash
docker pull caddy
```

[查看Docker Hub](https://hub.docker.com/_/caddy)

## DigitalOcean

[在 DigitalOcean 上部署 Caddy Droplet](https://marketplace.digitalocean.com/apps/caddy)

通过[`apt`库](/docs/install#debian-ubuntu-raspbian)安装，Droplet被预先配置为将Caddy作为[systemd服务](https://caddyserver.com/docs/running#linux-service)运行。


## Homebrew

__注意：这是社区维护的安装方法。__

```bash
brew install caddy
```

[查看Homebrew的formula](https://formulae.brew.sh/formula/caddy)


## Webi

__注意：这是社区维护的安装方法。__

Linux 和 macOS：

```bash
curl -sS https://webinstall.dev/caddy | bash
```

Windows：

```bash
curl.exe -A MS https://webinstall.dev/caddy | powershell
```

你可能需要调整 Windows 防火墙规则以允许非本地主机传入连接。

[查看Webi](https://webinstall.dev/caddy)

## Chocolatey

__注意：这是社区维护的安装方法。__

```bash
choco install caddy
```

[查看Chocolatey包](https://chocolatey.org/packages/caddy)

## Ansible

__注意：这是社区维护的安装方法。__

```bash
ansible-galaxy install nvjacobo.caddy
```

[查看Ansible角色仓库](https://github.com/nvjacobo/caddy)

## Scoop

__注意：这是社区维护的安装方法。__

```bash
scoop install caddy
```

[查看Scoop的manifest文件](https://github.com/ScoopInstaller/Main/blob/master/bucket/caddy.json)

## Termux

__注意：这是社区维护的安装方法。__

```bash
pkg install caddy
```

[查看Termux的build.sh文件](https://github.com/termux/termux-packages/blob/master/packages/caddy/build.sh)
