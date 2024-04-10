---
title: authentik-use-dingtalk-login
date: 2024-04-10 13:44:53
tags: [authentik, dingtalk, aws, serverless, python]
---

# Authentik 使用钉钉登录

## 介绍

Authentik 是一个开源的身份验证和授权服务，它提供了一种简单的方式来添加用户身份验证和授权功能到您的应用程序中。Authentik 支持多种身份验证方式，包括用户名密码、社交登录、LDAP 等。

钉钉是一款企业级即时通讯和协作工具，它提供了丰富的 API 和 SDK，可以用于集成企业内部系统和第三方应用。

在这个示例中，我们将演示如何使用 Authentik 集成钉钉登录，以便用户可以使用他们的钉钉账户登录到您的应用程序中。

钉钉是有 OAuth2 授权机制的，我们将使用 Authentik 的 OAuth2 提供程序来实现钉钉登录。

Authentik 的 OAuth2 提供程序允许您将任何符合 OAuth2 标准的身份验证服务集成到 Authentik 中，以便用户可以使用这些服务登录到您的应用程序中。[参见文档](https://docs.goauthentik.io/integrations/sources/oauth/)

## 步骤

### 1. 创建钉钉应用

可以参见 Gitlab 的文档 [DingTalk 集成](https://docs.gitlab.com/ee/integration/ding_talk.html) 或 Casdoor 的文档 [DingTalk OAuth](https://casdoor.org/zh/docs/provider/oauth/DingTalk/) 来创建一个钉钉应用。

#### 1.1 OAuth 接口说明

参见文档：[获取登录用户的访问凭证](https://open.dingtalk.com/document/orgapp/obtain-identity-credentials)

在 github 找到一个[示例](https://github.com/directus/directus/discussions/11881)

<details>
<summary>示例代码</summary>

```
Dingtalk is an enterprise communication tool similar to Jira/Confluence/WhatApp. It provides OAuth 2 service. The protocol is as below.

OAuth2 protocol, Dingtalk version
Auth
GET https://login.dingtalk.com/oauth2/auth?
redirect_uri=https%3A%2F%2Fwww.baidu.com%2F&response_type=code&client_id=dingyourclientid&scope=openid&prompt=consent

The redirect callback after Auth will be with below format:
https://www.baidu.com/?authCode=6b427e8bfab83e93bedd13f16a430702

Get token
POST https://api.dingtalk.com/v1.0/oauth2/userAccessToken
Content-Type:application/json

{
  "clientId" : "ding your id",
  "clientSecret" : "your secret",
  "code" : "6b427e8bfab83e93bedd13f16a430702",
  "grantType" : "authorization_code"
}
The response will be

{
  "expireIn": 7200,
  "accessToken": "a8f4e3215a703ce9a7164e91dbab53c0",
  "refreshToken": "b13e5a61b421342d95d86c9e64c275c6"
}
Retrieve user info
GET https://api.dingtalk.com/v1.0/contact/users/me
x-acs-dingtalk-access-token:a8f4e3215a703ce9a7164e91dbab53c0
Content-Type:application/json

Response will be

{
  "nick": "AWIS ME",
  "unionId": "D578iS5hxxxx",
  "avatarUrl": "https://static-legacy.dingtalk.com/media/lADPGT5i9m5ZyXDNA4LNAtA_720.jpg",
  "openId": "WySPOpXqxE",
  "mobile": "1350xxxxxxxx",
  "stateCode": "86",
  "email": "xxxu@xxx.com"
}
```

</details>


以及一些其他服务集成的代码样例 [apiproxy](https://github.com/xu4wang/apiproxy/blob/main/src/handlers/oauth_dingtalk.ts) 和 [kratos](https://github.com/ory/kratos/blob/eb67bed1f26d2c7ff10e5481b679b2213b44676d/selfservice/strategy/oidc/provider_dingtalk.go)

### 2. 配置 Authentik

可以参见配置 Twtich 的文档 [Twitch 集成](https://docs.goauthentik.io/integrations/sources/twitch/)

记得要选择 OpenID Connect 作为身份验证类型。

注意 `Scopes` 要填写 *openid, `Authorization URL` 要填写 https://login.dingtalk.com/oauth2/auth?prompt=consent

至于 `Token URL` 和 `User Info URL` 可以则需要自定义转化服务的 URL， 这是因为钉钉的 OAuth2 接口是非标准的（命名方法不一样，参数内容也有一些细微差异），所以需要自己实现一个 OAuth2 提供程序。可参见[这篇文章](https://zhuanlan.zhihu.com/p/666423994)


### 3. 实现 OAuth2 提供程序

我是使用 aws serverless 来实现的，代码如下：

<details>
<summary> /auth/dingtalk/token </summary>

```python
import requests
import json
from base64 import b64decode
from urllib.parse import parse_qs


TOKEN_URL = 'https://api.dingtalk.com/v1.0/oauth2/userAccessToken'


def parse_form_data_to_json(form_data):
    parsed_data = parse_qs(form_data)
    result = {k: v[0] for k, v in parsed_data.items()}
    return result


def main(event, context):
    print(f"event:\n{event}")
    s = event.get("body")
    if event.get("isBase64Encoded") and s:
        s = b64decode(s).decode("utf-8")
    body = parse_form_data_to_json(s)

    headers = {"Content-Type": "application/json"}
    response = requests.post(TOKEN_URL, json={
        'clientId': body.get('client_id'),
        'clientSecret': body.get('client_secret'),
        'code': body.get('code'),
        'grantType': body.get('grant_type'),
    }, headers=headers)
    response.raise_for_status()
    res = response.json()

    result = {
        # 'refresh_token': result.get('refreshToken'),
        'access_token': res.get('accessToken'),
        'expires_in': res.get('expiresIn'),
        'token_type': 'Bearer',
    }

    return {"statusCode": 200, "body": json.dumps(result)}

```

</details>

<details>
<summary> /auth/dingtalk/profile </summary>

```python
import requests
import json

URL = 'https://api.dingtalk.com/v1.0/contact/users/me'


def main(event, context):
    print(f"event:\n{event}")
    access_token = event.get('headers', {}).get('authorization', '')
    access_token = access_token.replace('Bearer ', '')
    print(access_token)

    headers = {
        "Content-Type": "application/json",
        'x-acs-dingtalk-access-token': access_token,
    }
    response = requests.get(URL, headers=headers)
    response.raise_for_status()
    user_info = response.json()
    print(user_info)

    result = {
        # 'issuer': userInfoURL,
        # 'picture': user_info.get('avatarUrl'),
        'sub': user_info['openId'],
        'nickname': user_info['nick'],
        'name': user_info['nick'],
        'email': user_info['email']
    }
    return {"statusCode": 200, "body": json.dumps(result)}

```

</details>


#### 错误排查

如果出现 `Could not determine id.` 的错误，可以参见 [源码解决](https://github.com/goauthentik/authentik/blob/main/authentik/sources/oauth/types/oidc.py) 来解决， 我当时处理少了 sub 参数。报错实现的[具体代码](https://github.com/goauthentik/authentik/blob/main/authentik/sources/oauth/views/callback.py#L59)可以定位。

在日志中可以根据关键字查找到源码， 日志如下

```
{"auth_via": "unauthenticated", "domain_url": "example.com", "event": "Authentication Failure", "host": "example.com", "level": "warning", "logger": "authentik.sources.oauth.views.callback", "pid": 4721, "reason": "Could not determine id.", "request_id": "28a8d8818c63441da41051455c32d437", "schema_name": "public", "timestamp": "2024-04-09T10:30:48.464283"}
```
