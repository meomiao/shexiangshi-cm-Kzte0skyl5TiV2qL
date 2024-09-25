
# 前言


### 1\. `scope` 参数的作用


* **定义权限**：`scope` 用于声明请求访问的资源和权限。常见的值包括 `openid`、`profile`、`email` 等。
* **影响返回的数据**：如果你在授权请求中指定了某些 `scope`，在后续的 token 请求中，Keycloak 会根据这些 `scope` 返回相应的信息。



> * openid用于指示请求者希望使用 OpenID Connect 进行身份验证
> * 获取 ID Token：当你在授权请求中包含 openid 时，Keycloak 会返回一个 ID Token，包含用户的身份信息，反之，scope里不加openid，则不会生成ID Token这个字段。



```


```

### 2\. 在不同阶段使用 `scope`


* **授权请求阶段** (`/protocol/openid-connect/auth`)：


	+ 可以传递 `scope` 参数，以便在用户同意授权时，明确所请求的权限。
* **令牌请求阶段** (`/protocol/openid-connect/token`)：


	+ 也可以在此阶段传递 `scope`，但通常情况下，如果在授权请求中已指定 `scope`，则不需要在此再次指定；
	+ 在授权中指定了`scope`，这里再指定是无效的，以`授权中指定的值`为准


### 3\. 示例


以下是一个获取授权码的请求示例：



```
GET /auth/realms/{realm}/protocol/openid-connect/auth?
response_type=code&
client_id={client_id}&
redirect_uri={redirect_uri}&
scope=openid profile

```

### 4\. 总结


* **建议**：虽然 `scope` 在授权请求中是可选的，但为了确保获得正确的权限和数据，建议在请求中包含 `scope` 参数。
* **注意**：在实际开发中，根据你的应用需求合理配置 `scope` 是非常重要的。


# 实践


## oauth2授权码认证的过程


* 用户在网站发起登录请求，请求地址为认证中心地址，参数会有scope,client\_id,response\_type,redirect\_uri等
* 认证中心检查到用户未登录，去表单页，让用户完成认证
* 表单认证成功后，需要让用户选择公开的字段，对应认证中心的scope
* 选择scope确认后，根据scope生成code，并重定向到来源页
* 网站在自己的回调接口中，处理code，并通过code获取认证中心颁发的token
* 网站通过token，可以获取认证中心的用户信息，信息范围为scope枚举中指定的字段


## 1\. 配置客户端及scope模板


* 默认模板，无论应用是否传scope，默认模板里的权限都会被启用
* 可选模板，由用户自己选择，通过scope来体现，它会追加到默认模板后面


![](https://images.cnblogs.com/cnblogs_com/lori/2399824/o_240925023430_clientScope.png)


## 2\. 获取授权码


* scope中体现了获取用户的email，这一步是由用户自己选择的公开的信息
* 地址：/auth/realms/{realm}/protocol/openid\-connect/auth?client\_id\=dahengshuju\&scope\=profile email\&redirect\_uri\=http://www.baidu.com\&response\_type\=code


## 3\. 表单认证


* 提示用户输入账号密码进行登录
* 登录成功后，重定向到来源页，带上code码
* code授权码使用一次后，立即过期


## 4\. 获取token


* 通过步骤3，获取到的code，它通过scope来限制授权的范围【即token和获取用户信息中包含的字段集合】
* 步骤2指定了scope，这一步再指定scope是无效的，二选一即可
* 地址：/auth/realms/{realm}/protocol/openid\-connect/token
* 请求表类型：x\-www\-form\-urlencoded
* 请求参数



```
grant_type:authorization_code
code:3be438fe-8651-4a84-8141-976b76e671e1.75cab95f-a1ec-4b9b-9a6e-8f1ecb651cd6.61d819de-33e4-4006-ae66-dd7609ea2d3e
client_id:dahengshuju
client_secret:9e3de70f-d5cd-4d11-a8aa-85fd3af13265
scope:profile

```

* 这是scope为profile email的token



```
{
    "exp": 1727233162,
    "iat": 1727231362,
    "auth_time": 1727229121,
    "jti": "bb296d9d-d521-45b1-aab9-8cb6bea0ddc3",
    "iss": "https://xx.xx.com/auth/realms/xx",
    "sub": "347c9e9e-076c-45e3-be74-c482fffcc6e5",
    "typ": "Bearer",
    "azp": "dahengshuju",
    "session_state": "75cab95f-a1ec-4b9b-9a6e-8f1ecb651cd6",
    "acr": "0",
    "scope": "email profile",
    "email_verified": false,
    "preferred_username": "test",
    "locale": "zh-CN",
    "email": "bfyxzls@gmail.com"
}

```

* 这是scope为profile的token，里面是没有email信息的



```
{
    "exp": 1727233521,
    "iat": 1727231721,
    "auth_time": 1727229121,
    "jti": "f7de8ad9-7558-4f4a-8761-8724f685febb",
    "iss": "https://xx.xx.com/auth/realms/xx",
    "sub": "347c9e9e-076c-45e3-be74-c482fffcc6e5",
    "typ": "Bearer",
    "azp": "dahengshuju",
    "session_state": "75cab95f-a1ec-4b9b-9a6e-8f1ecb651cd6",
    "acr": "0",
    "scope": "profile",
    "preferred_username": "test",
    "locale": "zh-CN"
}

```

## 5\. 通过access\_token获取用户信息


* 用户信息主要是对token中的内容进行解析
* 地址：/auth/realms/{realm}/protocol/openid\-connect/userinfo
* 请求头：Authorization: Bearer



```
{
    "sub": "347c9e9e-076c-45e3-be74-c482fffcc6e5",
    "email_verified": false,
    "preferred_username": "test",
    "locale": "zh-CN",
    "email": "xxx@gmail.com"
}

```

 本博客参考[楚门加速器p](https://tianchuang88.com)。转载请注明出处！
