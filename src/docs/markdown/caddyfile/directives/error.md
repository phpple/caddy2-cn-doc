---
title: error (Caddyfile 指令)
---

# error

触发HTTP处理链中的一个错误，有一个可选的消息和推荐的HTTP状态码。

这个处理程序并不会输出响应。相反，它应该与[`handle_errors`](handle_errors)指令搭配使用，以调用你的自定义错误处理逻辑。


## 语法

``caddy-d
error [<matcher>] <status>|<message> [<status>] {
message <text>
}
```

- **&lt;status&gt;**是要输出的HTTP状态代码。默认是`500`。
- **&lt;message&gt;**是错误信息。默认是没有错误信息。
- **message**是提供错误信息的另一种方式；多行的时候使用起来非常方便。

澄清一下，第一个非匹配器参数可以是一个3位数的状态代码，或者一个错误信息字符串。如果是一个错误信息，下一个参数可以是状态代码。

## 示例

在某些请求路径上触发一个错误, 并使用[`handle_errors`](handle_errors)来输出一个响应。

``caddy
example.com {
	root * /srv

	# 对某些路径触发错误
    error /private* "Unauthorized" 403
	error /hidden* "Not found" 404

    # 通过提供一个HTML页面来处理这个错误 
    handle_errors {
        rewrite * /{err.status_code}.html
		file_server
    }

	file_server
}
```
