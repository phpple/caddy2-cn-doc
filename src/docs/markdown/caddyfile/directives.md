---
title: Caddyfile指令
---

<style>
#directive-table table {
	margin: 0 auto;
	overflow: hidden;
}

#directive-table tr:hover {
	background: rgba(0, 0, 0, 10%);
}

#directive-table tr td:first-child {
	position: relative;
}

#directive-table a:before {
	content: '';
	position: absolute;
	left: 0;
	top: 0;
	bottom: 0;
	display: block;
	width: 100vw;
}
</style>

# Caddyfile指令

以下指令是 Caddy 的标准配置，可在 HTTP Caddyfile 中使用：

<div id="directive-table">

指令 | 说明
----------|------------
**[abort](/docs/caddyfile/directives/abort)** | 中止HTTP请求
**[acme_server](/docs/caddyfile/directives/acme_server)** | 嵌入式ACME服务器
**[basicauth](/docs/caddyfile/directives/basicauth)** | 强制执行HTTP基本身份验证
**[bind](/docs/caddyfile/directives/bind)** | 自定义服务器的套接字地址
**[encode](/docs/caddyfile/directives/encode)** | 编码（通常是压缩）响应
**[error](/docs/caddyfile/directives/error)** | 触发错误
**[file_server](/docs/caddyfile/directives/file_server)** | 从磁盘提供文件
**[handle](/docs/caddyfile/directives/handle)** | 一组互斥的指令
**[handle_errors](/docs/caddyfile/directives/handle_errors)** | 定义路由的错误处理器
**[handle_path](/docs/caddyfile/directives/handle_path)** | 像处理器，但去掉路径前缀
**[header](/docs/caddyfile/directives/header)** | 设置或删除响应标头
**[import](/docs/caddyfile/directives/import)** | 包括片段或文件
**[log](/docs/caddyfile/directives/log)** | 启用访问/请求日志记录
**[map](/docs/caddyfile/directives/map)** | 将输入值映射到一个或多个输出
**[metrics](/docs/caddyfile/directives/metrics)** | 配置Prometheus指标展示端点
**[php_fastcgi](/docs/caddyfile/directives/php_fastcgi)** | 通过FastCGI服务PHP站点
**[push](/docs/caddyfile/directives/push)** | 使用HTTP/2服务器推送将内容推送到客户端
**[redir](/docs/caddyfile/directives/redir)** | 向客户端发出HTTP重定向
**[request_body](/docs/caddyfile/directives/request_body)** | 操作请求包
**[request_header](/docs/caddyfile/directives/request_header)** | 操作请求头
**[respond](/docs/caddyfile/directives/respond)** | 向客户端写入硬编码响应
**[reverse_proxy](/docs/caddyfile/directives/reverse_proxy)** | 强大且可扩展的反向代理
**[rewrite](/docs/caddyfile/directives/rewrite)** | 在内部重写请求
**[root](/docs/caddyfile/directives/root)** | 设置站点根目录的路径
**[route](/docs/caddyfile/directives/route)** | 将一组指令从字面上视为单个单元
**[templates](/docs/caddyfile/directives/templates)** | 对响应执行模板
**[tls](/docs/caddyfile/directives/tls)** | 自定义TLS设置
**[try_files](/docs/caddyfile/directives/try_files)** | 取决于文件的存在的重写
**[uri](/docs/caddyfile/directives/uri)** | 操作URL

</div>

## 语法

每个指令的语法如下所示：

```caddy-d
directive [<matcher>] <args...> {
	subdirective [<args...>]
}
```

`<>`表示要由实际值替换的标记。

`[]`表示可选参数。

`...`表示延续，即一个或多个参数，或者多行。

除非另有说明，否则子指令始终是可选的，即使它们没有出现在`[]`.


### 匹配器

大多数——但不是全部——指令接受[匹配器标记](/docs/caddyfile/matchers#syntax)，它可以让你过滤请求。匹配器标记通常是可选的。如果你在指令的语法中看到这一点：

```caddy-d
[<matcher>]
```

然后该指令接受一个匹配器令牌，让你过滤该指令适用于哪些请求。

由于匹配器标记的工作方式相同，因此不会在每一页上都描述匹配器令牌的各种可能性，以减少重复。如果想了解详情，请统一参考[匹配器文档](/docs/caddyfile/matchers)。


## 指令顺序

许多指令操纵HTTP处理程序链。这些指令的默认顺序已经被硬编码到Caddy中，评估其顺序是很重要的事情：

```caddy-d
map
root

header
request_body

redir

# URI manipulation
rewrite
uri
try_files

# middleware handlers; some wrap responses
basicauth
request_header
encode
push
templates

# special routing & dispatching directives
handle
handle_path
route

# handlers that typically respond to requests
abort
error
respond
metrics
reverse_proxy
php_fastcgi
file_server
acme_server
```

你可以使用[`order`全局选项](/docs/caddyfile/options)或[`route`指令](/docs/caddyfile/directives/route)覆盖/自定义该排序。
