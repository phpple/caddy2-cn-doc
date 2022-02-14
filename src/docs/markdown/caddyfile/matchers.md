---
title: 请求匹配器
---

<script>
$(function() {
	// We'll add links on the matchers in the code blocks
	// to their associated anchor tags.
	let headers = $('article h3').map((i, el) => el.id.replace(/-/g, "_")).toArray();
	$('pre.chroma .k')
		.filter((k, item) => headers.includes(item.innerText))
		.map(function(k, item) {
			let text = item.innerText.replace(/</g,'&lt;').replace(/>/g,'&gt;');
			let url = '#' + item.innerText.replace(/_/g, "-");
			$(item).html('<a href="' + url + '" style="color: inherit;" title="' + text + '">' + text + '</a>');
		});

	// Link matcher tokens based on their contents to the syntax section
	$('pre.chroma .nd')
		.map(function(k, item) {
			let text = item.innerText.replace(/</g,'&lt;').replace(/>/g,'&gt;');
			let anchor = "named-matchers"
			if (text == "*") anchor = "wildcard-matchers"
			if (text.startsWith('/')) anchor = "path-matchers"
			$(item).html('<a href="#' + anchor + '" style="color: inherit;" title="Matcher token">' + text + '</a>');
		});
});
</script>

# 请求匹配器

**请求匹配器** 可用于按特定标准过滤（或分类）请求。

### 菜单

- [语法](#syntax)
    - [例子](#examples)
    - [通配符匹配器](#wildcard-matchers)
    - [路径匹配器](#path-matchers)
    - [命名匹配器](#named-matchers)
- [标准匹配器](#standard-matchers)
    - [expression](#expression)
    - [file](#file)
    - [header](#header)
    - [header_regexp](#header-regexp)
    - [host](#host)
    - [method](#method)
    - [not](#not)
    - [path](#path)
    - [path_regexp](#path-regexp)
    - [protocol](#protocol)
    - [query](#query)
    - [remote_ip](#remote-ip)


## 语法

在Caddyfile中，紧跟在指令后面的**匹配器标记**可以限制该指令的范围。匹配器标记可以是以下形式之一：

1. **`*`**匹配所有请求（通配符；默认）。
2. **`/path`**以正斜杠开头以匹配请求路径。
3. **`@name`**指定一个命名匹配器。

匹配器标记[通常是可选](/docs/caddyfile/directives#matchers)的。如果省略匹配器标记，则它与通配符匹配器（`*`）相同。


#### 例子

该指令适用于[所有](#wildcard-matchers)HTTP 请求：

```caddy-d
reverse_proxy localhost:9000
```

这是一样的：

```caddy-d
reverse_proxy * localhost:9000
```

但此指令仅适用于以`/api/`开头的[路径](#path-matchers)：

```caddy-d
reverse_proxy /api/* localhost:9000
```

要匹配路径以外的任何内容，请定义一个[命名匹配器](#named-matchers)并使用`@name`引用它：

```caddy-d
@postfoo {
	method POST
	path /foo/*
}
reverse_proxy @postfoo localhost:9000
```


### 通配符匹配器

通配符（或“catch-all”）匹配器`*`匹配所有请求，并且仅在需要匹配器标记时才需要。例如，如果你要给出指令的第一个参数也恰好是路径，那么它看起来就像一个路径匹配器！因此，你可以使用通配符匹配器来消除歧义，例如：

```caddy-d
root * /home/www/mysite
```

否则，这个匹配器不经常使用。尽可能省略它很方便；只是一个偏好问题。

### 路径匹配器

因为按路径匹配非常普遍，所以可以内联单个路径匹配器，如下所示：

```caddy-d
redir /old.html /new.html
```

路径匹配器标记必须以正斜杠`/`开头。

**[路径匹配](/docs/caddyfile/matchers#path) 默认为精确匹配；**你必须附加`*`以进行快速前缀匹配。请注意，`/foo*`将匹配`/foo`、`/foo/`和`/foobar`；你可能实际是想要`/foo/*`。

### 命名匹配器

所有不是路径或通配符匹配器的匹配器都必须命名为匹配器。这是一个在任何特定指令之外定义的匹配器，并且可以重用。

定义具有唯一名称的匹配器为你提供了更大的灵活性，允许你将[任何可用的匹配器](#standard-matchers)组合成一个集合：

```caddy-d
@name {
	...
}
```

或者，如果集合中只有一个匹配器：

```caddy-d
@name ...
```

然后你可以像这样使用匹配器：`@name`

例如：

```caddy-d
@websockets {
	header Connection *Upgrade*
	header Upgrade    websocket
}
reverse_proxy @websockets localhost:6001
```

这仅代理具有名为“Connection”的标头字段（包含“Upgrade”）和另一个名为“Upgrade”且值为“websocket”的字段的请求。

如果匹配器集仅包含一个匹配器，则单行语法也适用：

```caddy-d
@post method POST
reverse_proxy @post localhost:6001
```

与指令一样，命名匹配器定义必须放在使用它们的站点块内。

一个命名的匹配器定义构成一个_匹配器集_。集合中的匹配器是“与”的关系，也就是说，必须所有的都被匹配。例如，如果集合中有`aheader`和`path`匹配器，则两者都必须匹配。

相同类型的国歌匹配器可以被(AND/OR)组合起来（例如，同一个集合中的多个`path`匹配器），如下面的相应部分所述。



## 标准匹配器

完整的匹配器文档可以[在每个匹配器模块的文档中](/docs/json/apps/http/servers/routes/match/)找到。



### expression

⚠️ _此模块仍处于试验阶段，因此可能会发生重大变化。_

```caddy-d
expression <cel...>
```

通过任何[CEL(通用表达式语言)](https://github.com/google/cel-spec) 表达式返回`true`或者`false`。

作为一种特殊情况，可以在这些CEL表达式中使用Caddy[占位符](/docs/conventions#placeholders) (或[Caddyfile缩写](/docs/caddyfile/concepts#placeholders))，因为它们在被CEL环境解释之前经过预处理并转换为常规CEL函数调用。

#### 示例：

匹配以`P`开头的请求方法，例如`PUT`或`POST`。

```caddy-d
expression {method}.startsWith("P")
```

处理返回`404`错误状态代码的请求，且与[`handle_errors`指令](/docs/caddyfile/directives/handle_errors)结合起来处理。

```caddy-d
expression {http.error.status_code} == 404
```

---
### 文件

```caddy-d
file {
	root       <paths>
	try_files  <files...>
	try_policy first_exist|smallest_size|largest_size|most_recent_modified
	split_path <delims...>
}
```

通过文件进行匹配。

- `root`定义在其中查找文件的目录。默认是当前工作目录，或者`root`[变量](/docs/modules/http.handlers.vars) (`{http.vars.root}`)对应的位置 (可以通过[`root`指令](/docs/caddyfile/directives/root)设置)。
- `try_files`检查其列表中与重试策略(try_policy)匹配的文件。如果`try_policy`是`first_exist`，那么列表中的最后一项可能是一个以`=`(比如`=404`)开头的数字，作为后备，将触发以这个数字作为错误码的回调; 该错误也可以使用[`handle_errors`](/docs/caddyfile/directives/handle_errors)捕获和处理错误。
- `try_policy`指定如何选择文件。默认为`first_exist`.
    - `first_exist`检查文件是否存在。选择存在的第一个文件。
    - `smallest_size`选择大小最小的文件。
    - `largest_size`选择最大的文件。
    - `most_recent_modified`选择最近修改的文件。
- `split_path`将导致路径在每个要尝试的文件路径中找到的列表中的第一个分隔符处拆分。对于每个拆分值，拆分的左侧（包括分隔符本身）将是尝试的文件路径。例如，`/remote.php/dav/`使用`.php`作为分隔符，将尝试文件`/remote.php`。每个分隔符必须出现在 URI 路径组件的末尾，才能用作拆分分隔符。这是一个小众设置，主要用于为 PHP 站点提供服务。

因为`try_files`策略`first_exist`如此普遍，所以有一条捷径：

```caddy-d
file <files...>
```

一个空`file` 匹配器（后面没有列出任何文件），将通过URI从相对于[站点根目录](/docs/caddyfile/directives/root)的位置逐字比对查找文件是否存在。

由于基于磁盘上文件的存在进行重写非常普遍，因此还有一个[`try_files`指令](/docs/caddyfile/directives/try_files)，它是`file`匹配器和[`rewrite`处理器](/docs/caddyfile/directives/rewrite)的快捷方式。

匹配后，将可以使用两个新的占位符：

- `{http.matchers.file.relative}`文件的根相对路径。这在重写请求时通常很有用。
- `{http.matchers.file.absolute}`匹配文件的绝对路径。

#### 示例:

匹配路径是存在文件的请求。

```caddy-d
file
```

匹配请求，其中后跟的路径`.html`是存在的文件，或者如果不存在，则路径是存在的文件。

```caddy-d
file {
	try_files {path}.html {path} 
}
```

与上面相同，除了使用单行快捷方式，如果找不到文件，则回退到发出404错误。

```caddy-d
file {path}.html {path} =404
```

---
### header

```caddy-d
header <field> [<value>]
```

通过请求头字段进行匹配。

- `<field>`是要检查的 HTTP 标头字段的名称。
    - 如果以`!`为前缀，则该字段必须不存在才能匹配 (省略`value`参数).
- `<value>`是字段必须匹配的值。
    - 如果前缀是`*`，则执行快速后缀匹配。
    - 如果后缀为`*`，则执行快速前缀匹配。
    - 如果用`*`括起来，它将执行快速子字符串匹配。
    - 否则，它是快速精确匹配。

统一集合的不同header字段是“和”的关系。每个字段的多个值之间是“或”的关系。

#### 示例：

匹配请求`Connection`标头字段包含`Upgrade`的请求：

```caddy-d
header Connection *Upgrade*
```

匹配`Foo`标头字段包含`bar`或者`baz`的请求：

```caddy-d
@foo {
	header Foo bar
	header Foo baz
}
```

匹配根本没有`Foo`标头字段的请求：

```caddy-d
@not_foo {
	header !Foo
}
```


---
### header_regexp

```caddy-d
header_regexp [<name>] <field> <regexp>
```

和`header`类似，但是支持正则表达式。 捕获组可以通过[占位符](/docs/caddyfile/concepts#placeholders)访问，如`{re.name.capture_group}`，其中`name`是正则表达式的名称（可选，单推荐），且`capture_group`是表达式中捕获组的名称或编号。捕获组`0`是完整的正则表达式匹配，`1`是第一个捕获组，`2`是第二个捕获组，依此类推。

使用的正则表达式语言是GO语言中内置的`RE2`。具体请查阅[RE2语法参考](https://github.com/google/re2/wiki/Syntax)和[Go正则语法概述](https://pkg.go.dev/regexp/syntax)。

每个标头字段仅支持一个正则表达式。多个不同的字段是“与”的关系。

#### 示例：

将 Cookie 标头包含`login_`后跟十六进制字符串的请求，包含了一个可以通过`{re.login.1}`访问的捕获组。

```caddy-d
header_regexp login Cookie login_([a-f0-9]+)
```

---
### host

```caddy-d
host <hosts...>
```

通过请求的`Host`标头字段匹配请求。在 Caddyfile 中使用它并不常见，因为大多数站点块已经在站点地址中明确主机。此匹配器主要用于未定义特定主机名的站点块。

多个`host`匹配器之间是“或”的关系。

#### 示例：

```caddy-d
host sub.example.com
```



---
### method

```caddy-d
method <verbs...>
```

通过HTTP请求的方法（动词）匹配请求。动词应该是大写的，比如`POST`。可以匹配一种或多种方法。

多个`method`匹配之间是“或”的关系。

#### 示例：

匹配请求方法为`GET`的请求。

```caddy-d
method GET
```

匹配请求方法为`PUT`或者`DELETE`的请求。

```caddy-d
method PUT DELETE
```


---
### not

```caddy-d
not <any other matcher>
```

或者，要同时否定多个匹配器，请打开一个块：

```caddy-d
not {
	<any other matchers...>
}
```

包含起来的匹配器的结果将被否定。

#### 示例：

匹配路径不以`/css/`或`/js/`开头的请求。

```caddy-d
not path /css/* /js/*
```

匹配两者关系为`NEIGHER`的请求：
- 路径的前缀是`/api/`的，或非（NOR）
- 请求方法为`POST`

也就是说，必须__没有任何符合这些条件__(none of these)的请求才能被匹配：

```caddy-d
not path /api/*
not method POST
```

匹配两者关系为`WITHOUT BOTH`的请求：
- 路径的前缀是`/api/`，和(AND)
- 请求方法为`POST`

也就是说，必须__都不或者任何一个不__(neither or either)才能匹配：

```caddy-d
not {
	path /api/*
	method POST
}
```

---
### path

```caddy-d
path <paths...>
```

通过请求路径进行匹配，表示请求URI的路径部分。路径匹配是精确的，但也可以使用通配符`*`：

- 放在最后，进行前缀匹配 (`/prefix/*`)
- 放在开始，进行后缀匹配 (`*.suffix`)
- 在两边，进行子字符串匹配 (`*/contains/*`)
- 在中间，进行球状匹配 (`/accounts/*/info`)

请求路径在匹配前经过URL解码、小写（不区分大小写）和清理（折叠双斜杠和目录遍历点）。例如`/foo*`也会匹配`/FOO`、`//foo`和`/%2F/foo`。

多个`path`匹配之间是“或”的关系。

---
### path_regexp

```caddy-d
path_regexp [<name>] <regexp>
```

和`path`类似，但是支持正则表达式。捕获组可以通过[占位符](/docs/caddyfile/concepts#placeholders)访问，如`{re.name.capture_group}`，其中`name`是正则表达式的名称（可选，单推荐），且`capture_group`是表达式中捕获组的名称或编号。捕获组`0`是完整的正则表达式匹配，`1`是第一个捕获组，`2`是第二个捕获组，依此类推。

请求路径在匹配前经过URL解码、小写（不区分大小写）和清理（折叠双斜杠和目录遍历点）。例如`/foo*`也会匹配`/FOO`、`//foo`和`/%2F/foo`。

使用的正则表达式语言是GO语言中内置的`RE2`。具体请查阅[RE2语法参考](https://github.com/google/re2/wiki/Syntax)和[Go正则语法概述](https://pkg.go.dev/regexp/syntax)。

每个命名匹配器只能有一个`path_regexp`匹配器。

#### 示例：

匹配路径以6个字符的十六进制字符串结尾的请求，后跟`.css`或`.js`作为文件扩展名，捕获组可以分别用`{re.static.1}`和`{re.static.2}`访问包含在`( )`中间的每个部分。

```caddy-d
path_regexp static \.([a-f0-9]{6})\.(css|js)$
```

---
### protocol

```caddy-d
protocol http|https|grpc
```

通过请求协议进行匹配。

每个命名匹配器只能有一个`protocol`。


---
### query

```caddy-d
query <key>=<val>...
```

通过查询字符串参数匹配请求。应该是`key=value`的键值对。键完全匹配，区分大小写。值可以包含占位符。值完全匹配，但也支持`*`匹配任何值。

每个命名匹配器可以有多个`query`匹配器，具有相同键的对之间是“或”的关系。


#### 示例：

匹配带有`sort`查询参数且值为`asc`的请求。

```caddy-d
query sort=asc
```

---
### remote_ip

```caddy-d
remote_ip [forwarded] <ranges...>
```

通过远程（客户端）IP地址。接受确切的`IP`或`CIDR` 范围。如果第一个参数是`forwarded`，则`X-Forwarded-For`请求标头中的第一个IP（如果存在）将被首选作为参考IP，而不是默认的直接对等点(immediate peer)的IP。

多个`remote_ip`匹配器是“或”的关系。

#### 示例：

匹配来自私有 IPv4 地址的请求。

```caddy-d
remote_ip 192.168.0.0/16 172.16.0.0/12 10.0.0.0/8
```