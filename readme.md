# Caddy欢迎你

Caddy是一款基于Go语言编写的强大且可扩展的平台，可以给你的站点、服务和应用程序提供服务。虽然大多数人使用它作为web服务器或代理，但当你需要下述服务时也是很好的选择：
* web服务器
* 反向代理
* sidecar代理
* 负载均衡器
* API网关
* 入口控制器
* 系统管理
* 进程监控
* 任务调度
* 任何常驻进程

Caddy主要在[OSI模型](https://en.wikipedia.org/wiki/OSI_model)的L4(传输层)和L7(应用层)上运行，当然，它也能在其他层一起配合使用。

Caddy有两种方式不再需要配置文件：动态配置；基于[API](referrence/api.md)配置。Caddy的本地配置语言是[JSON](referrence/json-config-structure.md)，同时也支持很多[配置适配器](article/config-adapters.md)格式。

Caddy能在所有主流平台编译，没有其他外部依赖。

