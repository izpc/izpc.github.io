---
title: Grafana Authentik Login
date: 2024-05-31 15:43:36
tags: [grafana, authentik, login]
---

Grafana 支持多种登录方式，包括 LDAP, OAuth 等。本文介绍如何配置 Authentik 登录。

[参考1](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/)
[参考2](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/generic-oauth/)

# 配置 LDAP 登录
```yaml
grafana:
  ldap:
    config: "
      verbose_logging = true

      [[servers]]

      host="ldap.example.com"

      port=389

      use_ssl=false

      start_tls=false

      ssl_skip_verify=true

      bind_dn="cn=admin,dc=example,dc=com"

      bind_password="examplePassword"

      search_filter = "(uid=%s)"

      search_base_dns = ["ou=foo,dc=example,dc=com"]

      group_search_filter = "(objectClass=groupOfUniqueNames)"

      group_search_base_dns = ["ou=Group,dc=example,dc=com"]

      group_search_filter_user_attribute = "uid"

      [servers.attributes]

      name = "cn"

      surname = "sn"

      username = "uid"

      member_of = "memberOf"

      email =  "mail"

      [[servers.group_mappings]]

      group_dn = "cn=g-admin,ou=Group,dc=example,dc=com"

      org_role = "Editor""
    enabled: true
```

# 配置 Authentik 登录

`grafana.ini` 配置如下

```yaml
grafana:
  grafana.ini:
    auth:
      oauth_allow_insecure_email_lookup: true
    auth.generic_oauth:
      allow_assign_grafana_admin: true
      allow_sign_up: false
      allowed_organizations:
      api_url: "https://authentik.example.com/application/o/userinfo/"
      auth_url: "https://authentik.example.com/application/o/authorize/"
      auto_login: true
      client_id: ""
      client_secret: ""
      enabled: true
      name: "OAuth"
      scopes: "openid email profile offline_access"
      skip_org_role_sync: true
      team_ids:
      token_url: "https://authentik.example.com/application/o/token/"
      use_pkce: true
      use_refresh_token: true
    auth.ldap:
      allow_sign_up: true
      config_file: "/etc/grafana/ldap.toml"
      enabled: false
    server:
      root_url: "https://grafana.example.com/"
```

## 注意几点：


1. root_url: "https://grafana.example.com/", 需要配置为你的域名, 不然会出现跳转错误, [issue](https://github.com/goauthentik/authentik/issues/8673), [解决方案](https://community.grafana.com/t/redirect-uri-mismatch-error-in-google-oauth/35659/5)
2. oauth_allow_insecure_email_lookup: true, 允许不安全的邮箱查找, 由于我是通过 email 匹配的，所以需要设置为 true