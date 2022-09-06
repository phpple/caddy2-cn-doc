---
title: 编写配置适配器
---

# 编写配置适配器

由于各种原因，你可能希望使用一种不是[JSON](/docs/json/)的格式来配置Caddy。Caddy通过[配置适配器](/docs/config-adapters)对此有一流的支持。

如果你喜欢的语言/语法/格式还不存在，你可以写一个!

## 模板

这里有一个模板，你可以开始使用：

```go
package myadapter

import (
	"fmt"

	"github.com/caddyserver/caddy/v2/caddyconfig"
)

func init() {
	caddyconfig.RegisterAdapter("adapter_name", MyAdapter{})
}

// MyAdapter adapts ____ to Caddy JSON.
type MyAdapter struct{
}

// Adapt adapts the body to Caddy JSON.
func (a MyAdapter) Adapt(body []byte, options map[string]interface{}) ([]byte, []caddyconfig.Warning, error) {
	// TODO: parse body and convert it to JSON
	return nil, nil, fmt.Errorf("not implemented")
}
```

- 参见 godoc for [`RegisterAdapter()`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig?tab=doc#RegisterAdapter)
- 参见godoc for ['Adapter'](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig?tab=doc#Adapter) 接口

返回的JSON不应该***缩进；它应该始终是紧凑的。如果调用者愿意，他们可以随时对其进行美化。

请注意，虽然配置适配器是Caddy的_插件，但它们不是Caddy的_模块，因为它们没有集成到配置的某个部分（但为了方便，它们会显示在`list-modules`中）。因此，它们没有`Provision()`或`Validate()`方法，也不遵循模块生命周期的其他部分。它们只需要实现`Adapter'接口并注册为适配器。

当填充配置中`json.RawMessage`类型的字段（即模块字段）时，使用`JSON()`和`JSONModuleObject()`函数。

- [`caddyconfig.JSON()`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig?tab=doc#JSON)是用来处理没有嵌入模块名称的模块值。(通常用于ModuleMap字段，其中模块名称是地图的关键。)
- [`caddyconfig.JSONModuleObject()`](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig?tab=doc#JSONModuleObject)用于处理模块值，并在对象中加入模块名称。(其他地方几乎都在使用。)


## Caddyfile服务器类型

也可以实现一个自定义的Caddyfile格式。Caddyfile适配器是一个单一的适配器实现，其默认的 "服务器类型"是HTTP，但它在注册时支持替代的 "服务器类型"。例如，HTTP Caddyfile是这样注册的。

```go
func init() {
	caddyconfig.RegisterAdapter("caddyfile",  caddyfile.Adapter{ServerType: ServerType{}})
}
```

你将实现[`caddyfile.ServerType`接口](https://pkg.go.dev/github.com/caddyserver/caddy/v2/caddyconfig/caddyfile?tab=doc#ServerType)并相应地注册你自己的适配器。
