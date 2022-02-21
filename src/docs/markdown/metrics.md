---
title: 使用Prometheus监控Caddy
---

# 使用Prometheus监控Caddy

无论你是在云中运行数千个Caddy实例，还是在嵌入式设备上运行单个Caddy服务器，在某些时候你可能希望对Caddy正在做什么以及它持续了多长时间有一个高度的概览。
换句话说，你将希望能够 _监控_ Caddy。

## Prometheus

[Prometheus](https://prometheus.io)是一个监控平台，它通过在这些目标上抓取指标 HTTP 端点来收集来自被监控目标的指标。
除了帮助你使用[Grafana](https://grafana.com/docs/grafana/latest/getting-started/what-is-grafana/)等仪表板工具显示指标外，Prometheus还用于[报警](https://prometheus.io/docs/alerting/latest/overview/)。

与 Caddy 一样，Prometheus 是用 Go 编写的，并作为单个二进制文件分发。要安装它，请参阅[Prometheus安装文档](https://prometheus.io/docs/prometheus/latest/installation/)，或者在MacOS上运行`brew install prometheus`。

如果你是 Prometheus 的新手，请阅读[Prometheus文档](https://prometheus.io/docs/introduction/first_steps/)！

要将 Prometheus 配置为从 Caddy 抓取，你需要一个类似于以下的 YAML 配置文件：

```yaml
# prometheus.yaml
global:
  scrape_interval: 15s # default is 1 minute

scrape_configs:
  - job_name: caddy
    static_configs:
      - targets: ['localhost:2019']
```

然后你可以像这样启动Prometheus：

```console
$ prometheus --config.file=prometheus.yaml
```

## Caddy的指标

与使用 Prometheus 监控的任何进程一样，Caddy 公开了一个 HTTP 端点，该端点以[Prometheus展示格式](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format)进行响应。
Caddy 的 Prometheus 客户端还配置为在协商后以[OpenMetrics展示格式](https://pkg.go.dev/github.com/prometheus/client_golang@v1.7.1/prometheus/promhttp#HandlerOpts)
相应(即，如果`Accept`标头设置为`application/openmetrics-text; version=0.0.1`）。

默认情况下，[管理API](/docs/api)上的`/metrics`端点是可用的（即http://localhost:2019/metrics）。
但是如果管理API被禁用或者你希望在不同的端口或路径上监听，你可以使用[`metrics`处理器](/docs/caddyfile/directives/metrics)来配置它。

你可以使用任何浏览器或HTTP客户端查看指标，例如`curl`：

```console
$ curl http://localhost:2019/metrics
# HELP caddy_admin_http_requests_total Counter of requests made to the Admin API's HTTP endpoints.
# TYPE caddy_admin_http_requests_total counter
caddy_admin_http_requests_total{code="200",handler="metrics",method="GET",path="/metrics"} 2
# HELP caddy_http_request_duration_seconds Histogram of round-trip request durations.
# TYPE caddy_http_request_duration_seconds histogram
caddy_http_request_duration_seconds_bucket{code="308",handler="static_response",method="GET",server="remaining_auto_https_redirects",le="0.005"} 1
caddy_http_request_duration_seconds_bucket{code="308",handler="static_response",method="GET",server="remaining_auto_https_redirects",le="0.01"} 1
caddy_http_request_duration_seconds_bucket{code="308",handler="static_response",method="GET",server="remaining_auto_https_redirects",le="0.025"} 1
...
```

你会看到许多指标，大致分为 3 类：

- 运行时指标
- 管理API指标
- HTTP中间件指标

### 运行时指标

些指标涵盖了 Caddy 流程的内部，由 Prometheus Go 客户端自动提供。它们以`go_*`和`process_*`为前缀。

请注意，`process_*`指标仅在Linux和Windows系统上被收集。

请参阅[Go收集器](https://pkg.go.dev/github.com/prometheus/client_golang/prometheus#NewGoCollector)、进程收集器](https://pkg.go.dev/github.com/prometheus/client_golang/prometheus#NewProcessCollector)和[构建信息收集器](https://pkg.go.dev/github.com/prometheus/client_golang/prometheus#NewBuildInfoCollector)文档。

### 管理API指标

这些是有助于监控 Caddy 管理 API 的指标。每个管理端点都经过检测以跟踪请求计数和错误。

这些指标的前缀是`caddy_admin_*`。

例如：

```console
$ curl -s http://localhost:2019/metrics | grep ^caddy_admin
caddy_admin_http_requests_total{code="200",handler="admin",method="GET",path="/config/"} 1
caddy_admin_http_requests_total{code="200",handler="admin",method="GET",path="/debug/pprof/"} 2
caddy_admin_http_requests_total{code="200",handler="admin",method="GET",path="/debug/pprof/cmdline"} 1
caddy_admin_http_requests_total{code="200",handler="load",method="POST",path="/load"} 1
caddy_admin_http_requests_total{code="200",handler="metrics",method="GET",path="/metrics"} 3
```

#### `caddy_admin_http_requests_total`

管理端点处理的请求数的计数器，包括`admin.api.*`命名空间中的模块。

标签  | 描述
-------|------------
`code` | HTTP状态码
`handler` | 处理程序或模块名称
`method` | HTTP方法
`path` | 管理端点挂载到的URL路径

#### `caddy_admin_http_request_errors_total`

管理端点中遇到的错误数量的计数器，包括`admin.api.*`命名空间中的模块。

标签  | 描述
-------|------------
`handler` | 处理程序或模块名称
`method` | HTTP方法
`path` | 管理端点挂载到的URL路径

### HTTP中间件指标

所有 Caddy HTTP中间件处理程序都会自动检测，以确定请求延迟、首字节时间、错误和请求/响应正文大小。

<aside class="tip">
    因为所有中间件处理程序都经过检测，并且许多请求由多个处理程序处理，所以请确保不要简单地将所有计数器加在一起。
</aside>

对于下面的直方图指标，存储桶当前不可配置。
对于持续时间，使用默认的桶集[`prometheus.DefBuckets`](https://pkg.go.dev/github.com/prometheus/client_golang/prometheus#pkg-variables)（5ms、10ms、25ms、50ms、100ms、250ms、500ms、1s、2.5s、5s 和 10s）。
对于大小，桶是 256b、1kiB、4kiB、16kiB 、64kiB、256kiB、1MiB 和 4MiB。

#### `caddy_http_requests_in_flight`

衡量此服务器当前正在处理的请求数。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称

#### `caddy_http_request_errors_total`

处理请求时遇到的中间件错误计数器。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称

#### `caddy_http_requests_total`

发出的 HTTP(S) 请求的计数器。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称

#### `caddy_http_request_duration_seconds`

往返请求持续时间的直方图。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称
`code` | HTTP 状态码
`method` | HTTP 方法

#### `caddy_http_request_size_bytes`

请求的总（估计）大小的直方图。包括请求包体。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称
`code` | HTTP状态码
`method` | HTTP方法

#### `caddy_http_response_size_bytes`

返回的响应正文大小的直方图。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称
`code` | HTTP状态码
`method` | HTTP方法

#### `caddy_http_response_duration_seconds`

响应的第一个字节的时间直方图。

标签  | 描述
-------|------------
`server` | 服务器名称
`handler` | 处理程序或模块名称
`code` | HTTP状态码
`method` | HTTP方法

## 示例查询

一旦你让Prometheus抓取Caddy的指标，你就可以开始看到一些关于Caddy表现的有趣指标。

<aside class="tip">
    如果你已启动Prometheus服务器以使用上述配置抓取 Caddy，请尝试将这些查询粘贴到位于Prometheus UI的<a href="http://localhost:9090/graph">http://localhost:9090/graph</a>
</aside>

例如，要查看每秒请求率，平均超过 5 分钟：

```
rate(caddy_http_requests_total{handler="file_server"}[5m])
```

要查看超过 100 毫秒延迟阈值的速率：

```
sum(rate(caddy_http_request_duration_seconds_count{server="srv0"}[5m])) by (handler)
-
sum(rate(caddy_http_request_duration_seconds_bucket{le="0.100", server="srv0"}[5m])) by (handler)
```

要在`file_server`处理程序上查找占95%请求的持续时间，你可以使用如下查询：

```
histogram_quantile(0.95, sum(caddy_http_request_duration_seconds_bucket{handler="file_server"}) by (le))
```

或者查看`file_server`处理程序上`GET`请求成功的中位响应大小（以字节为单位）：

```
histogram_quantile(0.5, caddy_http_response_size_bytes_bucket{method="GET", handler="file_server", code="200"})
```
