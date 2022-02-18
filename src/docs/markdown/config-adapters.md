---
title: 配置适配器
---

# 配置适配器

Caddy 的原生配置语言是[JSON](https://www.json.org/json-en.html)，但手动编写 JSON 可能很乏味且容易出错。这就是Caddy支持通过**配置适配器**配置其他语言的原因。这些适配器是可以将你喜欢的格式输出成[Caddy JSON](/docs/json/)格式的Caddy插件。

例如，配置适配器可以[将NGINX配置转化成Caddy JSON](https://github.com/caddyserver/nginx-adapter)。

## 已知的配置适配器

以下配置适配器当前可用（一些是第三方项目）：

- [**caddyfile**](/docs/caddyfile) (标准)
- [**nginx**](https://github.com/caddyserver/nginx-adapter)
- [**jsonc**](https://github.com/caddyserver/jsonc-adapter)
- [**json5**](https://github.com/caddyserver/json5-adapter)
- [**yaml**](https://github.com/abiosoft/caddy-yaml)
- [**cue**](https://github.com/caddyserver/cue-adapter)
- [**toml**](https://github.com/awoodbeck/caddy-toml-adapter)
- [**hcl**](https://github.com/francislavoie/caddy-hcl)

（此列表是已知适配器的临时位置，直到我们的新网站完成。）

## 使用配置适配器

你可以通过在命令行上使用大多数接受配置的子命令上的`--adapter` 标志来指定它来使用配置适配器：

<pre><code class="cmd bash">caddy run --config caddy.yaml --adapter yaml</code></pre>

或者通过[`/load`端点](/docs/api#post-load)的APi：

<pre><code class="cmd bash">curl localhost:2019/load \
	-X POST \
	-H "Content-Type: application/yaml" \
	--data-binary @caddy.yaml</code></pre>

如果你只想获取输出JSON而不运行它，可以使用以下[`caddy adapt`](/docs/command-line#caddy-adapt)命令：

<pre><code class="cmd bash">caddy adapt --config caddy.yaml --adapter yaml</code></pre>

## 注意事项

并非所有配置语言都与 Caddy 100% 兼容；某些功能或行为根本无法很好地转换或尚未编程到适配器或 Caddy 本身。

一些适配器会进行1-1转换，例如YAML->JSON或TOML->JSON。其他是专门为Caddy设计的，例如Caddyfile。通常，这些适配器将始终有效。

然而，并不是所有的适配器都能一直工作。配置适配器尽最大努力将你的输入转换为具有最高保真度和正确性的Caddy JSON。因为不能保证这个转换过程始终是完整和正确的，所以我们不称他们为“转换器”或“翻译者”。它们是“适配器”，因为它们至少会给你一个很好的起点来完成你的最终JSON配置。

配置适配器可以输出生成的JSON、警告和错误。如果没有发生错误，则结果为JSON。当输入有问题（例如，语法错误）时会发生错误。当适应出现问题但不一定是致命的（例如，不支持的功能）时，会发出警告。如果使用带有警告的配置，建议小心。