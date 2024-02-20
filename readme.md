Caddy v2 中文文档
=================

这是Caddy v2中文文档的网站, [https://caddy2.dengxiaolong.com/](https://caddy2.dengxiaolong.com/docs/).


## 要求

- 安装Caddy 2 (在PATH可直接运行`caddy`)


## 快速开始

1. `git clone https://github.com/phpple/caddy2-cn-doc/`
2. `cd caddy2-cn-doc`
3. `caddy run`

第一次，系统可能会提示你输入密码。这样Caddy就可以通过本地HTTPS为站点提供服务。如果无法绑定低端口，请更改[`Caddyfile`](Caddyfile)顶部的地址，例如`localhost:2015`。

然后你可以通过浏览器访问[http://127.0.0.1:22020/](http://127.0.0.1:22020/) (或者你配置的其他地址)。
