---
title: tracing (Caddyfile指令)
---

# tracing

它提供了与OpenTelemetry追踪设施的整合。

当启用时，它将传播一个现有的跟踪上下文或初始化一个新的。

它基于[github.com/open-telemetry/opentelemetry-go]（https://github.com/open-telemetry/opentelemetry-go）。

它使用[gRPC](https://github.com/grpc/)作为输出协议，使用W3C[tracecontext](https://www.w3.org/TR/trace-context/)和[baggage](https://www.w3.org/TR/baggage/)作为传播器。

## 语法

```caddy-d
tracing {
	[span <span_name>]
}
```

- **&lt;span_name&gt;** - 是一个span的名称。请参阅 span 命名[指南](https://github.com/open-telemetry/opentelemetry-specification/blob/v1.7.0/specification/trace/api.md)。

## 配置

### 环境变量

可以使用[OpenTelemetry]定义的环境变量对其进行配置。
OpenTelemetry环境变量规范](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/sdk-environment-variables.md)。

关于导出器的配置细节，请
见[spec](https://github.com/open-telemetry/opentelemetry-specification/blob/v1.7.0/specification/protocol/exporter.md)。

比如说：

```bash
export OTEL_EXPORTER_OTLP_HEADERS="myAuthHeader=myToken,anotherHeader=value"
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=https://my-otlp-endpoint:55680
```

## 示例

下面是一个**Caddyfile**的例子：

```
handle /myHandler {
	tracing {
		span my-span
	}
	reverse_proxy 127.0.0.1:8081
}
```
