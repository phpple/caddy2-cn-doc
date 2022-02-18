---
title: basicauth (Caddyfile指令)
---

# basicauth

启用 HTTP 基本身份验证，可用于使用用户名和散列密码保护目录和文件。

**请注意，基本身份验证在普通HTTP上并不安全。** 在决定使用HTTP基本身份验证保护什么时，请谨慎使用。

当用户请求受保护的资源时，如果用户尚未提供用户名和密码，浏览器将提示用户输入用户名和密码。如果授权标头中存在正确的凭据，则服务器将授予对资源的访问权限。如果标头丢失或凭据不正确，服务器将响应 HTTP 401 Unauthorized。

Caddy 配置不接受明文密码；在将它们放入配置之前，你必须对它们进行哈希处理。该[`caddy hash-password`](/docs/command-line#caddy-hash-password)命令可以帮助解决这个问题。

成功验证后，`{http.auth.user.id}`占位符将可用，其中包含经过验证的用户名。


## 语法

```caddy-d
basicauth [<matcher>] [<hash_algorithm> [<realm>]] {
	<username> <hashed_password_base64> [<salt_base64>]
	...
}
```

- **&lt;hash_algorithm&gt;** 是在此配置中用于散列的密码散列算法（或 KDF）的名称。可以是`bcrypt`（默认）或者`scrypt`。
- **&lt;realm&gt;** 是自定义领域(realm)名称。
- **&lt;username&gt;** 是用户名或用户ID。
- **&lt;hashed_password_base64&gt;** 是散列密码的base-64编码。
- **&lt;salt_base64&gt;** 是密码盐的base-64编码，如果需要外部盐。


## 示例

保护 /secret 中的所有资源，以便只有Bob可以使用密码“hiccup”访问它们：

```caddy-d
basicauth /secret/* {
	Bob JDJhJDEwJEVCNmdaNEg2Ti5iejRMYkF3MFZhZ3VtV3E1SzBWZEZ5Q3VWc0tzOEJwZE9TaFlZdEVkZDhX
}
```

