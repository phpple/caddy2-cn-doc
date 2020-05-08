# Caddyfile快速入门

创建一个文本文件，命名为Caddyfile（没有后缀）。

第一个要写入Caddyfile的内容就是你的站点网址：

```caddy
localhost
```

> 如果HTTP和HTTPS端口(分别为80和443)是操作系统上的特权端口，则需要以提升的特权运行或使用更大的端口号。要使用更大的端口号，只需将地址更改为类似`localhost:2015`，并使用Caddyfile中的`http_port`选项 Caddyfile选更改HTTP端口。

然后按回车，输入你想让它做什么，它看起来像这样：

```caddy
localhost

respond "Hello, world!"
```

保存这个并从包含Caddyfile文件的目录运行Caddy：

```bash
caddy start
```

你可能会被要求输入密码，因为默认情况下，Caddy通过HTTPS为所有网站(甚至本地网站)提供服务。（密码提示应该只在第一次出现！）

> 对于本地HTTPS, Caddy自动为你生成证书和唯一的私钥。根证书被添加到系统的信任存储区，这就是会出来密码提示符的原因。它允许你在本地通过HTTPS进行开发，而不报证书的错误。

如果你看到权限错误，可能需要使用提升的特权运行。

```bash
curl https://localhost
Hello, world!
```

你可以在一个Caddyfile中定义多个站点，方法是将它们包装在花括号{}中。将你的Caddyfile改为：

```caddy
localhost {
	respond "Hello, world!"
}

localhost:2016 {
	respond "Goodbye, world!"
}
```

你可以给Caddy更新配置的两种方式，要么与API直接：

```bash
curl localhost:2019/load \
	-X POST \
	-H "Content-Type: text/caddyfile" \
	--data-binary @Caddyfile
```

或者使用reload命令，它为你执行相同的API请求：

```bash
caddy reload
```

在浏览器或curl中尝试一下新的“goodbye”站点，以确保它能正常工作：

```bash
curl https://localhost:2016
Goodbye, world!
```

当你使用完caddy，可以运行命令关闭它：

```bash
caddy stop
```