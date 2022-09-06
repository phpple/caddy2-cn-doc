---
title: Caddyfile支持
---

# Caddyfile支持

Caddy模块在[注册](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#RegisterModule)时，凭借其命名空间自动添加到[本地JSON配置](/docs/json/)中，使其既可使用又有记录。这使得对Caddyfile的支持纯粹是可选的，但它经常被那些喜欢Caddyfile的用户要求。

## Unmarshaler

要为你的模块添加Caddyfile支持，只需实现[`caddyfile.Unmarshaler`]（https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/caddyfile?tab=doc#Unmarshaler）接口。你可以通过解析标记的方式来选择你的模块的Caddyfile语法。

unmarshaler的工作是简单地设置你的模块类型，例如，通过填充它的字段，使用传递给它的[`caddyfile.Dispenser`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/caddyfile?tab=doc#Dispenser)。例如，一个名为`Gizmo`的模块类型可能有这样的方法。

```go
// UnmarshalCaddyfile实现了caddyfile.Unmarshaler。语法：
//
//     gizmo <name> [<option>]
//
func (g *Gizmo) UnmarshalCaddyfile(d *caddyfile.Dispenser) error {
	for d.Next() {
		if !d.Args(&g.Name) {
			// 没有足够的参数
			return d.ArgErr()
		}
		if d.NextArg() {
			// 可选的参数
			g.Option = d.Val()
		}
		if d.NextArg() {
			// 太多参数
			return d.ArgErr()
		}
	}
	return nil
}
```

在方法的godoc注释中记录语法是一个好主意。参见[`caddyfile`包的godoc](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/caddyfile?tab=doc)，了解更多关于解析Caddyfile的信息。

对于unmarshaler来说，接受其指令的多次出现也很重要（很少见，但在某些情况下可能发生）。由于第一个标记通常是模块的名称或指令（并且通常可以被unmarshaler跳过），这通常意味着将你的解析逻辑包裹在一个`for d.Next() { ... }`循环。

请确保检查是否有遗漏或多余的参数。

你还应该添加一个[接口防护](/docs/extending-caddy#interface-guards)，以确保接口得到正确满足。

```go
var _ caddyfile.Unmarshaler = (*Gizmo)(nil)
```

### 块

为了接受超过一行所能容纳的配置，你可能希望允许一个带有子指令的块。这可以用`d.NextBlock()`来完成, 并进行迭代, 直到返回到原来的嵌套级别。

```go
for nesting := d.Nesting(); d.NextBlock(nesting); {
	switch d.Val() {
		case "sub_directive_1":
			// ...
		case "sub_directive_2":
			// ...
	}
}
```


只要循环的每一次迭代都会消耗整个片段（行或块），那么这就是一种处理块的优雅方式。

## HTTP指令

HTTP Caddyfile是Caddy的默认Caddyfile适配器语法（或 "服务器类型"）。它是可扩展的，意味着你可以为你的模块[注册](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile?tab=doc#RegisterDirective)自己的 "顶级 "指令：

```go
func init() {
	httpcaddyfile.RegisterDirective("gizmo", parseCaddyfile)
}
```

如果你的指令只返回一个HTTP处理程序（这很常见），你可能会发现[`RegisterHandlerDirective`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile?tab=doc#RegisterHandlerDirective)更容易：

```go
func init() {
	httpcaddyfile.RegisterHandlerDirective("gizmo", parseCaddyfileHandler)
}
```

基本的想法是，你与指令相关联的[解析函数](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile?tab=doc#UnmarshalFunc)会返回一个或多个[`ConfigValue`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile?tab=doc#ConfigValue)值。(或者，如果使用 `RegisterHandlerDirective`，它只是直接返回填充的 `caddyhttp.MiddlewareHandler` 值) 。每个配置值都与一个["类"](#classes)相关联，这有助于HTTP Caddyfile适配器知道它可以用于最终JSON配置的哪一部分。所有的配置值都会被转储到一个堆中，适配器在构建最终的JSON配置时，会从中抽取。

这种设计允许你的指令为任何公认的类返回任何配置值，这意味着它可以影响HTTP Caddyfile适配器有指定类的配置的任何部分。

如果你已经实现了`UnmarshalCaddyfile()`方法，那么你的解析函数可以像这样简单：

```go
// parseCaddyfileHandler将h中的令牌解读为一个新的中间件处理值。
func parseCaddyfileHandler(h httpcaddyfile.Helper) (caddyhttp.MiddlewareHandler, error) {
	var g Gizmo
	err := g.UnmarshalCaddyfile(h.Dispenser)
	return g, err
}
```

关于如何使用`httpcaddyfile.Helper`类型的更多信息，请参阅[`httpcaddyfile`包godoc](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile?tab=doc)。

### 处理顺序

所有返回HTTP中间件/处理程序值的指令都需要以正确的顺序进行评估。例如，设置网站根目录的处理程序必须在访问根目录的处理程序之前，这样它就能知道目录路径是什么。

HTTP Caddyfile [对标准指令有一个硬编码的顺序](/docs/caddyfile/directives#directive-order)。这确保了用户不需要知道他们的网络服务器最常见的功能的实现细节，并使他们更容易写出正确的配置。鉴于Caddyfile的可扩展性，一个单一的、硬编码的列表也可以防止非确定性。

**当你注册一个新的处理指令时，它必须在使用前被添加到该列表中（在`路由`块之外）。** 这在配置中使用两种方法之一。

- [`order`全局选项](/docs/caddyfile/options)只为该配置修改标准顺序。例如：`order mydir before respond`将插入一个新的指令`mydir`，在`respond`处理程序之前被评估。然后该指令就可以正常使用了。
- 或者，在[`路由`块](/docs/caddyfile/directives/route)中使用该指令。因为路由块中的指令不会被重新排序，在路由块中使用的指令不需要出现在列表中。

请为你的用户记录你的指令在列表中的正确位置，以便他们能够正确地使用它。


### 类

这个表格描述了HTTP Caddyfile适配器所识别的每一个带有导出类型的类。

类的名称 | 预期的类型 | 说明
---------- | ------------- | -----------
bind | `[]string` | 服务器监听器绑定地址
tls.connection_policy | `*caddytls.ConnectionPolicy` | TLS连接策略
route | `caddyhttp.Route` | HTTP处理程序路线
error_route | `*caddyhttp.Subroute` | HTTP错误处理路由
tls.cert_issuer | `certmagic.Issuer` | TLS证书发放者
tls.cert_loader | `caddytls.CertificateLoader` | TLS证书加载器


## 服务类型

从结构上看，Caddyfile是一种简单的格式，因此可以有不同类型的Caddyfile格式（有时称为"服务类型"）来满足不同的需要。

默认的Caddyfile格式是HTTP Caddyfile，你可能对它很熟悉。这种格式主要配置[`http`应用程序](/docs/modules/http)，同时只可能在Caddy配置结构的其他部分洒下一些配置（例如，`tls`应用程序加载和自动化证书）。

要配置HTTP以外的应用程序，你可能想实现你自己的配置适配器，使用[你自己的服务器类型](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/caddyfile?tab=doc#Adapter)。Caddyfile适配器实际上将为你解析输入，并给你服务器块和选项的列表，由你的适配器来理解该结构并将其转化为JSON配置。