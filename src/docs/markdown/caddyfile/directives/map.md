---
title: map (Caddyfile指令)
---

# map

设置根据输入值进行切换的自定义占位符的值。

它将源值与映射的输入端进行比较，如果匹配，则将输出值应用到每个目标。目标则替换成对应的占位符。也可以为每个目标指定默认的输出值。

映射的占位符在使用前不会被评估，所以即使是非常大的映射，这个指令也是相当有效的。

## Syntax

```caddy-d
map [<matcher>] <source> <destinations...> {
	[~]<input> <outputs...>
	default    <defaults...>
}
```

- **&lt;source&gt;**  是要切换的输入值。通常是一个占位符。

- **&lt;destinations...&gt;** 是要创建的容纳输出值的占位符。

- **&lt;input&gt;** 是要匹配的输入值。如果前缀带了`~`，它讲被作为正则表达式处理。

- **&lt;outputs...&gt;** 是一个或多个要存储在相关占位符中的输出值。 第一个输出写到第一个目标，第二个输出写到第二个目标，以此类推。

  作为一种特殊情况，Caddyfile分析器将字面连字符（`-`）的输出视为null/nil值。如果你想在给定输入的情况下，对该特定输出使用默认值，但又想对其他输出使用非默认值，这很有用。

  如果可能的话，输出将被转换类型；`true`和`false`将被转换为布尔类型，数字值将被相应地转换为整数或浮点数。为了避免这种转换，你可以用[引号](/docs/caddyfile/concepts#tokens-and-quotes)来包裹输出，它们将保持为字符串。

  每个映射的输出数量不得超过目标的数量；但是，为了方便起见，输出的数量可以少于目标的数量，任何缺失的输出将被隐含地填入。

  如果使用正则表达式作为输入，那么捕获组可以用`${group}`来引用，其中`group`是表达式中捕获组的名称或编号。捕获组`0`是完整的重构表达式匹配，`1`是第一个捕获组，`2`是第二个捕获组，以此类推。

- **&lt;default&gt;** 指定了在没有匹配输入的情况下要存储的输出值。




## 示例

下面的例子演示了这个指令的大部分内容。

```caddy-d
map {host}             {my_placeholder}  {magic_number} {
	example.com        "some value"      3
	foo.example.com    "another value"
	(.*)\.example.com  "${1} subdomain"  5

	~.*\.net$          -                 7
	~.*\.xyz$          -                 15

	default            "unknown domain"  42
}
```

这条指令切换到`{host}`的值，也就是请求的域名。

- 如果请求的是`example.com`，则将`{my_placeholder}`设`some value`，将`{magic_number}`设为`3`。
- 否则，如果请求的是`foo.example.com`，将`{my_placeholder}`设置为`another value`，并让`{magic_number}`默认为`42`。
- 否则，如果请求是针对`example.com`的任何一个子域，则将`{my_placeholder}`设置为包含正则表达式捕获的第一个组的字符串，即整个子域，并将`{magic_number}`设置为`5`。
- 否则，如果请求是针对任何以`.net`或`.xyz`结尾的主机，只需将`{magic_number}`分别设置为`7`或`15`。不设置`{my_placeholder}`。
- 否则（对于所有其他主机），将适用默认值。`{my_placeholder}`将被设置为`unknown domain`，`{magic_number}`将被设置为`42`。
