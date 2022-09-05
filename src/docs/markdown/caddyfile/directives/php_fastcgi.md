---
title: php_fastcgi (Caddyfile指令)
---

# php_fastcgi

一个专用的指令，可以将请求代理给PHP FastCGI服务器，如php-fpm。

- [语法](#syntax)
- [扩展的形式](#expanded-form)
  - [解释](#explanation)
- [示例](#examples)

Caddy的[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)能够为任何FastCGI应用程序提供服务，但这个指令是专门为PHP应用程序定制的。这个指令实际上只是一个方便的方法，用来使用一个更长、更常见的配置（如下）。

它默认将网站根部的`index.php`作为一个路由器。如果不希望这样，可以重新配置`try_files`选项来修改默认的重写行为，或者使用下面的[扩展格式](#expanded-form)为基础，根据自己的需要进行定制。

它支持[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)的所有子指令，并将其传递给底层的`reverse_proxy`处理程序，另外还有一些专门定制FastCGI传输的子指令。

**大多数现代的PHP应用在没有额外的子指令或定制的情况下都能正常工作。** 子指令通常只用于某些边缘情况或传统的PHP应用。

<h3 id="syntax">语法</h3>

```caddy-d
php_fastcgi [<matcher>] <php-fpm_gateways...> {
	root <path>
	split <substrings...>
	env [<key> <value>]
	index <filename>|off
	try_files <files...>
	resolve_root_symlink
	dial_timeout  <duration>
	read_timeout  <duration>
	write_timeout <duration>

	<any other reverse_proxy subdirectives...>
}
```

- **<php-fpm_gateways...>** 是FastCGI服务器的[地址]（/docs/conventions#network-addresses）。
- **root** 设置网站的根文件夹。默认：[`root`指令]（/docs/caddyfile/directives/root）。
- **split** 设置用于将URI分成两部分的子串。第一个匹配的子串将被用来从路径中分割出 "路径信息"。第一部分以匹配的子串为后缀，将被认为是实际的资源（CGI脚本）名称。第二块将被设置为PATH_INFO，供CGI脚本使用。默认：`.php`。
- **env** 设置一个额外的环境变量为给定值。可以为多个环境变量指定多次。
- **index** 指定文件名作为目录索引文件。这影响到[扩展形式](#expanded-form)中的文件匹配器。默认：`index.php`。可以设置为`off`，当没有找到匹配的文件时，禁止重写到`index.php`。
- **try_files** 指定了对默认的try-files重写的覆盖。详情见[`try_files`指令]（/docs/caddyfile/directives/try_files）。默认：`{path} {path}/index.php index.php`。
- **resolve_root_symlink** 可以通过评估符号链接（如果存在的话）将`root`目录解析为其实际值。
- **dial_timeout** 是连接到上游套接字时需要等待的时间。接受[持续时间值]（/docs/conventions#durations）。默认值：`3s'。
- **read_timeout** 是指从FastCGI服务器读取数据时要等待多长时间。接受[持续时间值](/docs/conventions#durations)。默认值：无超时。
- **write_timeout** 是向FastCGI服务器发送时要等待多长时间。接受[持续时间值](/docs/conventions#durations)。默认：没有超时。


由于该指令是对反向代理的专用包装，你可以使用[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy#syntax)的任何子指令来定制它。


<h3 id="expanded-form">扩展形式</h3>

`php_fastcgi`指令（没有子指令）与以下配置相同。大多数现代的PHP应用都能很好地使用这个预设。如果你的没有，可以随意借用，并根据需要进行定制，而不是使用`php_fastcgi`快捷方式。

```caddy-d
route {
	# 为目录请求添加尾部斜线
	@canonicalPath {
		file {path}/index.php
		not path */
	}
	redir @canonicalPath {path}/ 308

	# 如果请求的文件不存在，尝试索引文件
	@indexFiles file {
		try_files {path} {path}/index.php index.php
		split_path .php
	}
	rewrite @indexFiles {http.matchers.file.relative}

	# 将 PHP 文件代理给 FastCGI 应答器
	@phpFiles path *.php
	reverse_proxy @phpFiles <php-fpm_gateway> {
		transport fastcgi {
			split .php
		}
	}
}
```


<h3 id="explanation">解释</h3>

- 第一部分涉及到请求路径的规范化。其目的是确保针对磁盘上的目录的请求实际上在请求路径中添加了尾部的斜线`/`，因此只有一个URL对该目录的请求有效。

  这是通过使用一个请求匹配器来实现的，该匹配器只匹配不以斜线结尾的请求，并且映射到磁盘上包含`index.php`文件的目录，如果匹配，则执行HTTP 308重定向，并添加尾部斜线。因此，举例来说，如果磁盘上存在`/foo/index.php`，它就会将对路径`/foo`的请求重定向到`/foo/`（附加`/`，以规范该目录的路径）。

- 下一节涉及到根据磁盘上是否存在匹配的文件来执行路径重写。这也有一个副作用，就是记住路径中`.php`之后的部分（如果请求路径中有`.php`）。这对Caddy正确设置FastCGI的环境变量很重要。

  - 首先，它检查`{path}`是否是一个存在于磁盘上的文件。如果是，它就重写到该路径。这本质上是对其他部分的短路，并确保对磁盘上确实存在的文件的请求不会被重写（见下面的步骤）。因此，如果你在磁盘上有一个`/js/app.js`文件，那么对该路径的请求将被保持不变。

  - 其次，它检查`{path}/index.php`是否是一个存在于磁盘上的文件。如果是，它就会重写到该路径。对于向`/foo/`目录的请求，它将寻找`/foo//index.php`（它将被规范化为`/foo/index.php`），如果它存在，将重写请求到该路径。如果你在另一个子目录下运行一个PHP应用程序，这种行为有时很有用。

  - 最后，如果该文件存在，它将重写到`index.php`（对于现代的PHP应用程序，它几乎总是应该存在）。这使得你的PHP应用程序可以通过使用`index.php`脚本作为其入口，来处理任何没有映射到磁盘上文件的路径请求。

- 最后，最后一节实际上是将请求代理给PHP FastCGI（或PHP-FPM）服务，以实际运行你的PHP代码。请求匹配器只匹配以".php "结尾的请求，因此，任何不是PHP脚本的文件，如果在磁盘上确实存在，将不会被这个指令所处理，而会被忽略。

`php_fastcgi`指令本身通常是不够的。它几乎总是要和[`root`指令]（/docs/caddyfile/directives/root）配对，以设置文件在磁盘上的位置（对于现代PHP应用程序，这可能是`/var/www/html/public`，其中`public`目录是包含`index. php`），以及[`file_server`指令]（/docs/caddyfile/directives/file_server）来提供你的静态文件（你的JS、CSS、图片等），这些文件并没有被这个指令处理，而是被遗忘了。



<h3 id="examples">示例</h3>

将所有的PHP请求代理给一个在127.0.0.1:9000监听的FastCGI响应器。

```caddy-d
php_fastcgi 127.0.0.1:9000
```

同样，但只针对`/blog/`下的请求。

```caddy-d
php_fastcgi /blog/* 127.0.0.1:9000
```

当使用php-fpm通过unix socket监听时。

```caddy-d
php_fastcgi unix/run/php/php7.4-fpm.sock
```

[`root`指令](/docs/caddyfile/directives/root)通常用于指定包含PHP脚本的目录，而[`file_server`指令](/docs/caddyfile/directives/file_server)用于提供静态文件。

```caddy-d
root * /var/www/html/public
php_fastcgi 127.0.0.1:9000
file_server
```

对于不使用`index.php`作为入口的PHP站点，可以退而求其次，发出一个`404`错误。这个错误可以用[`handle_errors`指令](/docs/caddyfile/directives/handle_errors)处理。

```caddy-d
php_fastcgi localhost:9000 {
	try_files {path} {path}/index.php =404
}
```
