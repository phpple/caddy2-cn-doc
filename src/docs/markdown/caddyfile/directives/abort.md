---
title: abort (Caddyfile指令)
---

# abort

通过立即中止HTTP处理链和关闭连接，阻止对客户端的任何响应。同一连接上的任何并发的、活动的HTTP流都会被中断。


## 语法

```caddy-d
abort [<matcher>]
```

## 示例

当使用通配符证书时，强行关闭收到的未定义域名的连接。

```caddy
*.example.com {
    @foo host foo.example.com
    handle @foo {
        respond "This is foo!" 200
    }

    # 未定义的域名会落到这里，但我们不想接收它们的请求
    handle {
        中止
    }
}
```