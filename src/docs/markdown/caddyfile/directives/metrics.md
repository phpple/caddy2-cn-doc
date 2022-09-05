---
title: metrics (Caddyfile指令)
---

# metrics

配置一个Prometheus度量标准展示端点，这样采集的指标可以被暴露出来以供抓取。

注意，[admin API](/docs/api)上也有一个`/metrics`端点。 但它不能进行配置，而且当管理API被禁用时也不能使用。

这个端点将以[Prometheus数据传输格式](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format)返回指标。如果经过协商，也可以使用[OpenMetrics数据传输格式](https://pkg.go.dev/github.com/prometheus/client_golang@v1.9.0/prometheus/promhttp#HandlerOpts)
(`application/openmetrics-text`)。

另见[用Prometheus度量标准监控Caddy](/docs/metrics)。

## 语法

```caddy-d
metrics [<matcher>] {
	disable_openmetrics
}
```

- **disable_openmetrics** 禁用OpenMetrics格式。这个指令通常没有必要，除非需要解决解析错误。

## 示例

在默认的`/metrics`路径下显示指标。

```caddy-d
  metrics /metrics
```

在另一个路径上显示指标。

```caddy-d
metrics /foo/bar/baz
```

在一个单独的子域中提供指标。

```caddy
metrics.example.com {
	metrics
}
```

停用OpenMetrics格式。

```caddy-d
metrics /metrics {
	disable_openmetrics
}
```
