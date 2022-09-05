---
title: import (Caddyfile指令)
---
#导入

包括一个[snippet](/docs/caddyfile/concepts#snippets)或文件，用该片段或文件的内容替换该指令。

这个指令是一个特例：它在结构解析之前被评估，而且它可以出现在Caddyfile的任何地方。

## 语法

```caddy-d
import <pattern> [<args...>]
```

- **&lt;pattern&gt;** 是文件名、glob模式或[snippet](/docs/caddyfile/concepts#snippets)的名称，要包括在内。它的内容将取代这一行，就像该文件的内容一开始就出现在这里一样。如果找不到一个特定的文件是一个错误，但是一个空的glob模式并不是一个错误。如果这个模式是一个文件名或glob，它总是相对于`import`出现的文件而言的。
- **&lt;args...&gt;** 是一个可选的参数列表，用于传递给导入的标记。它们可以和一个形式为`{args.N}`的占位符一起使用，其中`N`是参数的基于0的位置索引。这个占位符是一种特殊情况，在解析时而不是运行时进行评估。


## 示例

导入相邻站点启用的文件夹中的所有文件。

```caddy-d
import sites-enabled/*
```

使用导入参数导入一个设置CORS头文件的片段。

```caddy
(cors) {
	@origin header Origin {args.0}
	header @origin Access-Control-Allow-Origin "{args.0}"
	header @origin Access-Control-Allow-Methods "OPTIONS,HEAD,GET,POST,PUT,PATCH,DELETE"
}

example.com {
	import cors example.com
}
```
