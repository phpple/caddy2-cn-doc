---
title: vars (Caddyfile指令)
---

# vars

将一个或多个变量设置为一个特定的值，以便在以后的请求处理链中使用。

访问变量的主要方式是使用占位符，其形式为`{vars.variable_name}`，或者使用[`vars`](/docs/caddyfile/matchers#vars)和[`vars_regexp`](/docs/caddyfile/matchers#vars_regexp) 请求匹配器。

## 语法

```caddy-d
vars [<matcher>] [<name> <value>] {
    <name> <value>
    ...
}
```

- **&lt;name&gt;**是要设置的变量名称。

- **&lt;value&gt;**是该变量的值。

  如果可能的话，该值将进行类型转换；`true`和`false`将被转换为布尔类型，数字值将被相应地转换为整数或浮点数。为了避免这种转换，你可以用[引号](/docs/caddyfile/concepts#tokens-and-quotes)来包裹输出，它们将保持为字符串。

## 示例

设置一个单一的变量，该值是基于请求路径的条件，然后用该值进行响应：

```caddy-d
vars /foo* isFoo "yep"
vars isFoo "nope"

respond {vars.isFoo}
```

要设置多个变量，每个变量都转换为适当的标量类型：

```caddy-d
vars {
	# boolean
	abc true

	# integer
	def 1

	# float
	ghi 2.3

	# string
	jkl "example"
}
```