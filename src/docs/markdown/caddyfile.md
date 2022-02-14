---
title: Caddyfile
---

# Caddyfile

**Caddyfile**是一种方便用户使用的Caddy配置格式。这是大多数人最喜欢使用Caddy的方式，因为它易于编写、易于理解，且能满足绝大部分的使用场景。

它看起来像这样

```caddy
example.com

root * /var/www/wordpress
php_fastcgi unix//run/php/php-version-fpm.sock
file_server
```

（这是一个真正的、生产就绪的Caddyfile，它通过完全托管的HTTPS为WordPress提供服务。）

基本思路是，先填写网站的地址，然后填写网站需要具备的特性或功能。查看更多[常见模式](/docs/caddyfile/patterns)。

## 菜单

- #### [快速入门指南](/docs/quick-starts/caddyfile)
- #### [完整的Caddyfile教程](/docs/caddyfile-tutorial)
- #### [Caddyfile概念](/docs/caddyfile/concepts)
- #### [指令](/docs/caddyfile/directives)
- #### [请求匹配器](/docs/caddyfile/matchers)
- #### [全局选项](/docs/caddyfile/options)
- #### [常见模式](/docs/caddyfile/patterns)
<!-- - #### [Caddyfile规范](/docs/caddyfile/spec) TODO: Finish this -->


## 备注

Caddyfile只是Caddy的[配置适配器](/docs/config-adapters)。在手工制作配置时通常首选它，但它不如Caddy的[原生JSON结构](/docs/json/)那样富有表现力、灵活或可编程。如果你正在尝试将你的的Caddy配置/部署实现自动化，你可能希望将JSON结合Caddy的[API](/docs/api)一起使用。（实际上，也可以将Caddyfile与API一起使用，只是在有限的范围内。）
