---
title: 扩展Caddy
---

# 扩展Caddy

Caddy很容易扩展，因为它是模块化结构。大多数类型的Caddy扩展（或插件）被称为_modules_，如果它们扩展或插入Caddy的配置结构。为了清楚起见，Caddy的模块与[Go模块](https://github.com/golang/go/wiki/Modules)不同（但它们也是Go模块）。

**先决条件：**
- 对[Caddy的架构](/docs/architecture)的基本了解
- 熟练掌握Go语言
- [`go`](https://golang.org/doc/install)
- [`xcaddy`](https://github.com/caddyserver/xcaddy)


## 快速入门

Caddy模块是任意一种命名类型，其包被导入时将自己注册为Caddy模块。最重要的是，一个模块总是实现[caddy.Module](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#Module)接口，提供它的名字和构造函数。

在一个新的Go模块中，将以下模板粘贴到Go文件中，并定制你的包名、类型名和Caddy模块ID。

```go
package mymodule

import "github.com/caddyserver/caddy/v2"

func init() {
	caddy.RegisterModule(Gizmo{})
}

// Gizmo只是一个例子；可以是你自己的类型
type Gizmo struct {
}

// 通过CaddyModule方法返回Caddy模块的信息
func (Gizmo) CaddyModule() caddy.ModuleInfo {
	return caddy.ModuleInfo{
		ID:  "foo.gizmo",
		New: func() caddy.Module { return new(Gizmo) },
	}
}
```

然后从你的项目目录中运行这个命令，则在列表中应该能看到你的模块：

```bash
xcaddy list-modules
...
foo.gizmo
...
```

<aside class="tip">
[`xcaddy`命令](https://github.com/caddyserver/xcaddy)是每个模块开发者工作流程中的一个重要部分。它用你的插件编译Caddy，然后用给定的参数运行它。它每次都会丢弃临时二进制文件（类似于`go run`）。
</aside>


恭喜你，你的模块在Caddy注册了，可以在[Caddy的配置文件](/docs/json/)中的任意位置使用具有相同命名空间的模块。

基于这套机制，`xcaddy`只是制作了一个依赖Caddy和你的插件的新的Go模块（用适当的`replace`来使用你的本地开发版本），然后添加一个导入，确保它被编译到。

```go
import _ "github.com/example/mymodule"
```

## 模块基础知识

Caddy模块：

1. 实现`caddy.Module`接口，提供一个ID和构造函数。
2. 在适当的命名空间里有一个独特的名字
3. 通常实现一些对该命名空间的主模块有意义的接口。

**主模块**（或母模块）是用于加载/初始化其他模块的模块。它们通常为访客模块定义命名空间。

**访客模块**（或_子模块_）是被加载或初始化的模块。所有模块都是访客模块。


## 模块ID

每个Caddy模块都有一个唯一的ID，由命名空间和名称组成。

- 一个完整的ID看起来像`foo.bar.module_name`。
- 命名空间是`foo.bar`。
- 名称是`module_name`，在其命名空间中必须是唯一的。

模块ID必须使用`snake_case`惯例。

### 命名空间

命名空间就像类一样，也就是说，一个命名空间定义了一些功能，这些功能在它的所有模块中是通用的。例如，我们可以预期在`http.handlers`命名空间中的所有模块都是HTTP处理程序。因此，宿主模块可以将该命名空间中的客体模块从 "interface{}"类型转为更具体、更有用的类型，如 "caddyhttp.MiddlewareHandler"。

客体模块必须有正确的命名空间，才能被宿主模块识别，因为宿主模块会要求Caddy在某一命名空间内提供宿主模块所需的功能。例如，如果你要写一个叫`gizmo'的HTTP处理程序模块，你的模块的名字将是`http.handlers.gizmo'，因为`http'应用程序会在`http.handlers'命名空间中寻找处理程序。

换句话说，Caddy模块被期望实现[某些接口]（/docs/extending-caddy/namespaces），这取决于它们的模块名称空间。有了这个约定，模块开发者可以说一些直观的东西，比如："`http.handlers`命名空间中的所有模块都是HTTP处理程序。" 更为技术性的是，这通常意味着："`http.handlers`命名空间中的所有模块都实现了`caddyhttp.MiddlewareHandler`接口"。因为这个方法集是已知的，所以更具体的类型可以被断言和使用。

**[查看所有标准Caddy命名空间与它们的Go类型的映射表。]（/docs/extending-caddy/namespaces）**

`caddy'和`admin'命名空间是保留的，不能作为应用程序的名称。

要编写插入第三方主机模块的模块，请查阅这些模块的命名空间文档。

###名称

命名空间中的名字很重要，对用户来说非常明显，但并不特别重要，只要它是唯一的、简洁的，并且对它的作用有意义。

## 应用程序模块

应用程序是具有空的命名空间的模块，它习惯上成为自己的顶层命名空间。应用程序模块实现了[caddy.App](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#App)接口。

这些模块出现在Caddy配置的顶层的[`"apps"`](/docs/json/#apps)属性中。

```json
{
	"apps": {}
}
```

例如[apps](/docs/json/apps/)是`http`和`tls`。他们的是空命名空间。

为这些应用程序编写的访客模块应该在一个由应用程序名称衍生的命名空间中。例如，HTTP处理程序使用`http.handlers`命名空间，TLS证书加载器使用`tls.certificates`命名空间。

## 模块实现

一个模块几乎可以是任何类型，但结构体是最常见的，因为它们可以保存用户配置。


### 配置

大多数模块需要一些配置。只要你的类型与JSON兼容，Caddy会自动处理这个问题。因此，如果一个模块是一个结构类型，它将需要在其字段上使用结构标签，根据Caddy的惯例，应该使用`snake_casing`。

```go
type Gizmo struct {
	MyField string `json:"my_field,omitempty"`
	Number  int    `json:"number,omitempty"`
}
```

以这种方式使用结构标签将确保配置属性在所有Caddy中的命名是一致的。

当一个模块被初始化时，它的配置已经填好了。也可以在模块初始化后执行额外的[provisioning](#provisioning)和[validation](#validating)步骤。


### 模块生命周期

一个模块的生命从它被主机模块加载时开始。会发生以下情况。

1. [`New()`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#ModuleInfo.New)被调用，以获得一个模块的值的实例。
2. 2. 模块的配置被解密到该实例中。
3. 3. 如果该模块是一个[caddy.Provisioner](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#Provisioner)，则调用`Provision()`方法。
4. 4. 如果该模块是[caddy.Validator](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#Validator)，则调用`Validate()`方法。
5. 5.在这一点上，宿主模块被赋予加载的客体模块作为`interface{}`值，所以宿主模块通常会对客体模块进行类型确认，使其成为更有用的类型。检查宿主模块的文档，了解其命名空间对客体模块的要求，例如，需要实现哪些方法。
6. 当一个模块不再需要时，如果它是一个[caddy.CleanerUpper](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#CleanerUpper)，就会调用`Cleanup()`方法。

请注意，你的模块的多个加载实例可能会在某一特定时间重叠! 在配置改变期间，新的模块会在旧的模块停止之前启动。一定要小心使用全局状态。使用[caddy.UsagePool](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#UsagePool)类型来帮助管理跨模块加载的全局状态。如果你的模块在套接字上监听，使用`caddy.Listen*()`来获得一个支持重叠使用的套接字。

### 额外配置(provisioning)

一个模块的配置将被自动解密到其值中。这意味着，例如，结构字段将为你填好。

但是，如果你的模块需要额外的配置步骤，你可以实现（可选）[caddy.Provisioner]（https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#Provisioner）接口。

```go
// Provision sets up the module.
func (g *Gizmo) Provision(ctx caddy.Context) error {
	// TODO: set up the module
	return nil
}
```

这通常是宿主模块将加载他们的客人/子模块的地方，但它可以用于几乎任何东西。模块的配置是以任意的顺序进行的。

一个模块可以通过调用`ctx.App()`访问其他应用程序，但模块不能有循环依赖关系。换句话说，如果`http`应用加载的模块依赖于`tls`应用，那么`tls`应用加载的模块就不能依赖于`http`应用。(与 Go 中禁止导入循环的规则非常相似）。

此外，你应该避免在`Provision`中执行昂贵的操作，因为即使配置只是被验证，也要进行配置。当处于供应阶段时，不要期望模块会被实际使用。

#### 日志

如果你的模块需要记录日志，不要使用Go标准库中的`log.Print*()`。换句话说，**不要使用Go的全局日志器**。Caddy使用高性能、高度灵活、结构化的日志[zap]（https://github.com/uber-go/zap）。

要发射日志，在你模块的Provision方法中获得一个日志器：

```go
func (g *Gizmo) Provision(ctx caddy.Context) error {
	g.logger = ctx.Logger(g) // g.logger is a *zap.Logger
}
```

然后你可以使用`g.logger`发送结构化的、分层的日志。详见[zap的go文档](https://pkg.go.dev/go.uber.org/zap?tab=doc#Logger)。

### 验证

想验证其配置的模块可以通过满足（可选）[`caddy.Validator`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#Validator)接口来进行验证。

```go
// Validate验证模块是否有可用的配置。
func (g Gizmo) Validate() error {
	// TODO: 验证模块的设置。
	return nil
}
```

Validate应该是一个只读的函数。它在`Provision()`方法之后运行。


### 接口守护

Caddy模块的行为是隐性的，因为Go接口是隐性满足的。只需在你的模块类型中添加正确的方法，就可以使你的模块正确与否。因此，打错字或弄错方法签名会导致意外的（缺乏）行为。

幸运的是，有一个简单的、无开销的、编译时的检查，你可以添加到你的代码中，以确保你已经添加了正确的方法。这些被称为接口防护。

```go
var _ InterfaceName = (*YourType)(nil)
```

将`InterfaceName`替换为你打算满足的接口，将`YourType`替换为你的模块的类型名称。

例如，一个HTTP处理程序，如静态文件服务器，可能满足多个接口：

```go
// Interface guards
var (
	_ caddy.Provisioner           = (*FileServer)(nil)
	_ caddyhttp.MiddlewareHandler = (*FileServer)(nil)
)
```

这样，如果`*FileServer`不满足这些接口，程序就无法编译。

没有接口防护，混乱的 bug 就会溜进来。例如，如果你的模块必须在使用前提供自己，但你的`Provision()`方法有一个错误（例如拼写错误或签名错误），提供将永远不会发生，导致挠头。接口防护是非常简单的，可以防止这种情况。它们通常放在文件的底部。

## 主机模块

当一个模块加载它自己的客户模块时，它就成为一个主机模块。如果一个模块的功能可以用不同的方式实现，这就很有用。

一个主机模块几乎总是一个结构。通常情况下，支持客体模块需要两个结构域：一个用于保存其原始JSON，另一个用于保存其解码值。

```go
type Gizmo struct {
	GadgetRaw json.RawMessage `json:"gadget,omitempty" caddy:"namespace=foo.gizmo.gadgets inline_key=gadgeter"`

	Gadget Gadgeter `json:"-"`
}
```

第一个字段（本例中的`GadgetRaw'）是可以找到客人模块的原始的、未被提供的JSON形式的地方。

第二个字段(`Gadget')是最终配置的值将被存储的地方。由于第二个字段不是面向用户的，我们用结构标签将其从JSON中排除。(如果其他软件包不需要它，你也可以取消导出，这样就不需要结构标签了。)

### Caddy结构标签

原始模块字段上的`caddy`结构标签帮助Caddy知道要加载的模块的名称空间和名称（包括完整的ID）。它也用于生成文档。

该结构标签有一个非常简单的格式。`key1=val1 key2=val2 ...`。

对于模块字段，结构标签将看起来像：

```go
`caddy:"namespace=foo.bar inline_key=baz"`
```

`namespace=`部分是必须的。它定义了寻找模块的命名空间。

`inline_key=`部分只在模块名称与模块本身并列时使用；这意味着值是一个对象，其中一个键是_inline key_，其值是模块的名称。如果省略，那么字段类型必须是[`caddy.ModuleMap`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#ModuleMap)或`[]caddy.ModuleMap`，其中映射键是模块名称。


###加载客户模块

要加载一个客户模块，在提供阶段调用[`ctx.LoadModule()`](https://pkg.go.dev/github.com/caddyserver/caddy/v2?tab=doc#Context.LoadModule)。

```go
// Provision sets up g and loads its gadget.
func (g *Gizmo) Provision(ctx caddy.Context) error {
	if g.GadgetRaw != nil {
		val, err := ctx.LoadModule(g, "GadgetRaw")
		if err != nil {
			return fmt.Errorf("loading gadget module: %v", err)
		}
		g.Gadget = val.(Gadgeter)
	}
	return nil
}
```

注意，`LoadModule()`调用需要一个指向结构的指针和一个字符串的字段名。很奇怪，对吗？为什么不直接传递结构字段呢？这是因为根据配置的布局，有几种不同的方式来加载模块。这个方法签名允许Caddy使用反射来找出加载模块的最佳方式，最重要的是，读取其结构标签。

如果客体模块必须由用户明确设置，那么在尝试加载模块之前，如果Raw字段为nil或空，你应该返回一个错误。

注意加载的模块是如何进行类型确认的。`g.Gadget = val.(Gadgeter)` - 这是因为返回的`val`是一个`interface{}`类型，不是很有用。然而，我们期望在声明的命名空间中的所有模块（在我们的例子中来自结构标签的`foo.gizmo.gadgets`）实现`Gadgeter`接口，所以这个类型断言是安全的，然后我们可以使用它

如果你的主机模块定义了一个新的命名空间，一定要为开发者记录该命名空间和它的Go类型[就像我们在这里做的]（/docs/extending-caddy/namespaces）。

## 完整的例子

让我们假设我们想写一个HTTP处理模块。这将是一个为演示目的而设计的中间件，在每个HTTP请求中把访问者的IP地址打印成一个流。

我们还希望它可以通过Caddyfile进行配置，因为大多数人喜欢在非自动情况下使用Caddyfile。我们通过注册一个Caddyfile处理程序指令来做到这一点，这是一种可以向HTTP路由添加处理程序的指令。我们还实现了`caddyfile.Unmarshaler`接口。通过添加这几行代码，这个模块就可以用Caddyfile进行配置了! 例如：`visitor_ip stdout`。

以下是这样一个模块的代码，并附有解释说明：

```go
package visitorip

import (
	"fmt"
	"io"
	"net/http"
	"os"

	"github.com/caddyserver/caddy/v2"
	"github.com/caddyserver/caddy/v2/caddyconfig/caddyfile"
	"github.com/caddyserver/caddy/v2/caddyconfig/httpcaddyfile"
	"github.com/caddyserver/caddy/v2/modules/caddyhttp"
)

func init() {
	caddy.RegisterModule(Middleware{})
	httpcaddyfile.RegisterHandlerDirective("visitor_ip", parseCaddyfile)
}

// 中间件实现了一个HTTP处理程序，将访问者的IP地址写入
// 访客的IP地址写到文件或流中。
type Middleware struct {
	// 要写入的文件或流。可以是 "stdout"或 "stderr"。
	Output string `json:"output,omitempty"`

	w io.Writer
}

// CaddyModule返回Caddy模块的信息。
func (Middleware) CaddyModule() caddy.ModuleInfo {
	return caddy.ModuleInfo{
		ID:  "http.handlers.visitor_ip",
		New: func() caddy.Module { return new(Middleware) },
	}
}

// Provision实现了caddy.Provisioner。
func (m *Middleware) Provision(ctx caddy.Context) error {
	switch m.Output {
	case "stdout":
		m.w = os.Stdout
	case "stderr":
		m.w = os.Stderr
	default:
		return fmt.Errorf("an output stream is required")
	}
	return nil
}

// Validate实现了caddy.Validator。
func (m *Middleware) Validate() error {
	if m.w == nil {
		return fmt.Errorf("no writer")
	}
	return nil
}

// ServeHTTP 实现了 caddyhttp.MiddlewareHandler。
func (m Middleware) ServeHTTP(w http.ResponseWriter, r *http.Request, next caddyhttp.Handler) error {
	m.w.Write([]byte(r.RemoteAddr))
	return next.ServeHTTP(w, r)
}

// UnmarshalCaddyfile实现了caddyfile.Unmarshaler。
func (m *Middleware) UnmarshalCaddyfile(d *caddyfile.Dispenser) error {
	for d.Next() {
		if !d.Args(&m.Output) {
			return d.ArgErr()
		}
	}
	return nil
}

// parseCaddyfile从h中解读令牌到一个新的中间件。
func parseCaddyfile(h httpcaddyfile.Helper) (caddyhttp.MiddlewareHandler, error) {
	var m Middleware
	err := m.UnmarshalCaddyfile(h.Dispenser)
	return m, err
}

// Interface guards
var (
	_ caddy.Provisioner           = (*Middleware)(nil)
	_ caddy.Validator             = (*Middleware)(nil)
	_ caddyhttp.MiddlewareHandler = (*Middleware)(nil)
	_ caddyfile.Unmarshaler       = (*Middleware)(nil)
)
```