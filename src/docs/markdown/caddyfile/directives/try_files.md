---
title: try_files (Caddyfile指令)
---

将请求的URI路径重写到网站根目录中存在的第一个文件。如果没有匹配的文件，则不进行重写。


## 语法

```caddy-d
try_files <files...>
```

- **<files...>**是要尝试的文件列表。URI将被改写为第一个存在的文件。要匹配目录，请在路径后面加上一个正斜杠`/`。所有的文件路径都是相对于网站[root]（/docs/caddyfile/directives/root）。每个参数也可以包含一个查询字符串，在这种情况下，如果查询字符串与该特定文件匹配，也会被改变。列表中的最后一项可以是以`=`为前缀的数字（例如`=404`），作为后备措施，它将发出一个带有该代码的错误；该错误可以被捕获，并用[`handle_errors`](/docs/caddyfile/directives/handle_errors)处理。


## 扩展的形式

`try_files`指令基本上是一个快捷方式：

```caddy-d
@try_files file <files...>
rewrite @try_files {http.matchers.file.relative}
```

注意，这个指令不接受匹配器标记。如果你需要更复杂的匹配逻辑，那么就使用上面的扩展形式作为基础。

参见[`file`匹配器](/docs/caddyfile/matchers#file)以了解更多细节。


## 示例

如果请求没有匹配任何静态文件，则重写到一个索引/路由文件：

```caddy-d
try_files {path} /index.php
```

同样的，但在查询字符串中加入原始路径：

```caddy-d
try_files {path} /index.php?{query}&p={path}
```

相同的，但也要匹配目录：

```caddy-d
try_files {path} {path}/ /index.php?{query}&p={path}
```

如果文件或目录存在，尝试重写，否则发出404错误（可以用[`handle_errors`](/docs/caddyfile/directives/handle_errors)捕捉和处理）。

```caddy-d
try_files {path} {path}/ =404
```
