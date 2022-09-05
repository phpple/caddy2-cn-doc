---
title: log (Caddyfile指令)
---

# 日志

启用并配置HTTP请求日志（也称为访问日志）。

<aside class="tip">
  如果你想配置Caddy的运行时日志，你要找的应该是<a href="/docs/caddyfile/options#log"><code>log</code>全局选项</a>。
</aside>

`log`指令适用于它出现在网站块的主机/端口，而不是网站地址的任何其他部分（如路径）。

- [语法](#syntax)
- [输出模块](#output-modules)
  - [stderr](#stderr)
  - [stdout](#stdout)
  - [discard](#discard)
  - [file](#file)
  - [net](#net)
- [格式化模块](#format-modules)
  - [console](#console)
  - [json](#json)
  - [filter](#filter)
    - [delete](#delete)
    - [rename](#rename)
    - [replace](#replace)
    - [ip_mask](#ip-mask)
    - [query](#query)
    - [cookie](#cookie)
    - [regexp](#regexp)
    - [hash](#hash)
- [例子](#examples)

从Caddy v2.5开始，默认情况下，带有潜在敏感信息的头文件（`Cookie`、`Set-Cookie`、`Authorization`和`Proxy-Authorization`）将以空值记录。这种行为可以通过[`log_credentials`](/docs/caddyfile/options#log-credentials)全局服务器选项禁用。

<h3 id="syntax">语法</h3>

```caddy-d
log {
	output <writer_module> ...
	format <encoder_module> ...
	level  <level>
}
```

- **output** 配置了写日志的地方。参见下面的[输出模块](#output-modules)。默认：`stderr`。
- **format** 描述了如何对日志进行编码，或格式化。参见下面的[格式模块](#format-modules)。默认：`console`如果`stdout`被检测到是一个终端，`json`否则。
- **level** 是记录的最小入口级别。默认值：`INFO`。注意，访问日志目前只发出`INFO`和`ERROR`级别的日志。

<h3 id="output-module">输出模块</h3>

**output** 子指令让你自定义日志的写入位置。它出现在一个`log`块中。

#### stderr

标准错误（控制台，默认）。

```caddy-d
output stderr
```

#### stdout

标准输出（控制台）。

```caddy-d
output stdout
```

#### 丢弃

没有输出。

```caddy-d
output discard
```

#### 文件

一个文件。默认情况下，日志文件被旋转（"滚动"），以防止磁盘空间耗尽。

```caddy-d
output file <filename> {
	roll_disabled
	roll_size     <size>
	roll_uncompressed
	roll_local_time
	roll_keep     <num>
	roll_keep_for <days>
}
```

- **&lt;filename&gt;** 是日志文件的路径。
- **roll_disabled** 禁用日志滚动。这可能导致磁盘空间耗尽，所以只在你的日志文件以其他方式维护时使用。
- **roll_size** 是滚动日志文件的大小。目前的实现支持兆字节的分辨率；小数值被四舍五入到下一个整兆字节。例如，`1.1MiB`被四舍五入为`2MiB`。默认值：`100MiB`。
- **roll_uncompressed** 关闭gzip日志压缩。默认值：启用gzip压缩。
- **roll_local_time** 将滚动设置为在文件名中使用本地时间戳。默认值：使用UTC时间。
- **roll_keep** 是在删除最旧的日志文件之前要保留多少个日志文件。默认值：`10`。
- **roll_keep_for** 是将滚动的文件作为[持续时间字符串]（/docs/conventions#durations）保留多长时间。目前的实现支持日的分辨率；小数的值被四舍五入到下一个整日。例如，`36h`（1.5天）被四舍五入为`48h`（2天）。默认值: `2160h` (90天)


#### net

一个网络套接字。如果该套接字发生故障，它将在尝试重新连接时向stderr转储日志。

```caddy-d
output net <address> {
  dial_timeout <duration>
}
```

- **&lt;address&gt;** 是写日志的[地址](/docs/conventions#network-addresses)。
- **dial_timeout** 是等待成功连接到日志套接字的时间。如果套接字发生故障，日志排放可能会被阻断，最长时间为这个时间。

### 格式化模块

**format**子指令让你自定义日志的编码方式（格式化）。它出现在一个`log`块中。

<aside class="tip">
<b>关于通用日志格式（CLF）的说明：</b> CLF与现代结构化日志发生冲突。要将你的访问日志转换为被废弃的通用日志格式，请使用<a href="https://github.com/caddyserver/transform-encoder" target="_blank"><code>transform-encoder</code>插件</a>。
</aside>

除了每个单独的编码器的语法外，这些通用属性可以在大多数编码器上设置。

```caddy-d
format <encoder_module> {
	message_key     <key>
	level_key       <key>
	time_key        <key>
	name_key        <key>
	caller_key      <key>
	stacktrace_key  <key>
	line_ending     <char>
	time_format     <format>
	duration_format <format>
	level_format    <format>
}
```

- **message_key** 日志条目的消息字段的键。默认：`msg`。
- **level_key** 日志条目的级别字段的键。默认值: `level`.
- **time_key** 日志条目的时间字段的键。默认值: `ts`.
- **name_key** 日志条目的名称字段的键（即记录器本身的名称）。默认值: `name`.
- **caller_key** 日志条目中调用者字段的键。
- **stacktrace_key** 日志条目的堆栈跟踪字段的键。
- **line_ending** 要使用的行结束语。
- **time_format** 时间戳的格式。可以是其中之一。
  - **unix_seconds_float**自Unix epoch以来的浮点数；这是默认的。
  - **unix_milli_float** 自Unix epoch以来的浮点数，即毫秒。
  - **unix_nano** 自Unix epoch以来的纳秒整数。
  - **iso8601** 示例：`2006-01-02T15:04:05.000Z0700`。
  - **rfc3339** 例如：`2006-01-02T15:04:05Z07:00`。
  - **rfc3339_nano** 例如：`2006-01-02T15:04:05.9999999Z07:00`。
  - **wall** 例如：`2006/01/02 15:04:05`。
  - **wall_milli** 例如：`2006/01/02 15:04:05.000`。
  - **wall_nano** 例如：`2006/01/02 15:04:05.000000000`。
  - **common_log** 例如：`02/Jan/2006:15:04:05 -0700`。
  - 或者，任何兼容的时间布局字符串；完整的细节见[Go documentation](https://pkg.go.dev/time#pkg-constants)。
- **duration_format** 持续时间的格式。可以是其中之一。
  - **seconds** 浮点数，即经过的秒数；这是默认的。
  - **nano** 经过的纳秒的整数。
  - **string** 使用Go的内置字符串格式，例如`1m32.05s`或`6.31ms`。
- **level_format** 级别的格式。可以是以下之一。
  - **lower** 小写；这是默认的。
  - **upper** 大写。
  - **color** 大写，有控制台颜色。


#### console

控制台编码器在保留一些结构的同时，对日志条目进行格式化，以便于人类阅读。

```caddy-d
format console
```

#### json

将每个日志条目格式化为JSON对象。

```caddy-d
format json
```

#### filter

包裹另一个编码器模块，允许每个字段的过滤。

```caddy-d
format filter {
	wrap <encode_module> ...
	fields {
		<field> <filter> ...
	}
}
```

嵌套字段可以通过用`>`表示一层嵌套而被引用。换句话说，对于像`{"a":{"b":0}}这样的对象，内部字段可以被引用为`a>b`。

以下字段是日志的基础，不能被过滤，因为它们是由底层日志库作为特例添加的。`ts`, `level`, `logger`, 和 `msg`.

这些是可用的过滤器。

##### delete

标志着一个字段被跳过编码。

```caddy-d
<field> delete
```

##### rename

重命名一个日志字段的键。

```caddy-d
<field> rename <key>
```

##### replace

标记一个字段，在编码时用提供的字符串替换。

```caddy-d
<field> replace <replacement>
```

##### ip_mask

使用CIDR掩码对字段中的IP地址进行掩码，即从左边开始，保留IP的字节数。对IPv4和IPv6地址有单独的配置。最常见的是，要过滤的字段是`request>remote_ip`。

```caddy-d
<field> ip_mask {
	ipv4 <cidr>
	ipv6 <cidr>
}
```

##### query

标记一个字段，执行一个或多个操作，以操作URL字段的查询部分。最常见的是，要过滤的字段是`uri`。可用的操作是。

```caddy-d
<field> query {
	delete  <key>
	replace <key> <replacement>
	hash    <key>
}
```

- **delete** 从查询中删除给定的键。
- **replace** 用**replacement**替换给定查询键的值。对插入一个编辑占位符很有用；你会看到查询键在URL中，但其值被隐藏了。
- **hash** 用SHA-256哈希值的前4个字节替换给定查询键的值，小写16进制。如果该值是敏感的，对隐藏该值很有用，同时能够注意到每个请求是否有不同的值。

##### cookie

标记一个字段，执行一个或多个操作，以操纵`Cookie`HTTP头的值。最常见的是，要过滤的字段是`request>headers>Cookie`。可用的操作是。

```caddy-d
<field> cookie {
	delete  <name>
	replace <name> <replacement>
	hash    <name>
}
```

- **delete** 从标头中删除给定的cookie的名称。
- **replace** 用**replacement**替换给定cookie的值。对插入一个节录的占位符很有用；你会看到该cookie在头中，但其值被隐藏。
- **hash** 用SHA-256哈希值的前4个字节替换给定的cookie的值，小写16进制。如果该值是敏感的，对隐藏该值很有用，同时能够注意到每个请求是否有不同的值。

如果为同一个cookie名称定义了许多行动，那么只有第一个行动将被应用。

##### regexp

标记一个字段，在编码时应用正则表达式替换。

```caddy-d
<field> regexp <pattern> <replacement>
```

使用的正则表达式语言是RE2，包含在Go中。参见 [RE2 语法参考](https://github.com/google/re2/wiki/Syntax) 和 [Go regexp 语法概述](https://pkg.go.dev/regexp/syntax)。

在替换字符串中，捕获组可以用`${group}`来引用，其中`group`是表达式中捕获组的名称或编号。捕获组`0`是完整的regexp匹配，`1`是第一个捕获组，`2`是第二个捕获组，以此类推。

##### hash

标记一个字段，在编码时用该值的SHA-256哈希值的前4个字节来替换。如果该值是敏感的，可以用来掩盖它，同时能够注意到每个请求是否有不同的值。

```caddy-d
<field> hash
```


<h3 id="examples">示例</h3>

启用访问日志(到控制台):

```caddy-d
log
```


将日志写到一个文件中（使用日志滚动，默认情况下启用）。

```caddy-d
log {
	output file /var/log/access.log
}
```


自定义日志滚动。

```caddy-d
log {
	output file /var/log/access.log {
		roll_size 1gb
		roll_keep 5
		roll_keep_for 720h
	}
}
```


从日志中删除授权请求标头。

```caddy-d
log {
	format filter {
		wrap console
		fields {
			request>headers>Authorization delete
		}
	}
}
```


对多个敏感的cookies进行修改。

```caddy-d
log {
	format filter {
		wrap console
		fields {
			request>headers>Cookie cookie {
				replace session REDACTED
				delete secret
			}
		}
	}
}
```


屏蔽请求中的远程地址，保留IPv4地址的前16位（即255.255.0.0），IPv6地址的前32位。(注意，在Caddy v2.5之前，这个字段被命名为`remote_addr'，但现在是`remote_ip'）。

```caddy-d
log {
	format filter {
		wrap console
		fields {
			request>remote_ip ip_mask {
				ipv4 16
				ipv6 32
			}
		}
	}
}
```
