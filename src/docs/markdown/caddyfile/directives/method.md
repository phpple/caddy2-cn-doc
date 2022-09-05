---
title: method (Caddyfile指令)
---

# method

改变请求中的HTTP方法。


## 语法

``caddy-d
method [<matcher>] <method>.
```

- **&lt;method&gt;** 是要改变请求的HTTP方法。


## 示例

将`/api`下的所有请求的方法改为`POST`。

``caddy-d
method /api* POST
```
