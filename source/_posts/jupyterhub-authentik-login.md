---
title: JupyterHub Authentik Login
date: 2024-05-31 16:18:19
tags: [jupyterhub, authentik, login]
---

[参考文档](https://z2jh.jupyter.org/en/stable/administrator/authentication.html#genericoauthenticator-openid-connect) 以及一些[配置](https://oauthenticator.readthedocs.io/en/stable/reference/api/gen/oauthenticator.oauth2.html)


下面之前的 LDAP 和 启用的新的 OAuth 配置，可以参考一下：

注意 allow_all 为 true，表示允许所有用户登录

```yaml
hub:
  activeServerLimit:
  allowNamedServers: false
  annotations: {}
  args: []
  authenticatePrometheus:
  baseUrl: "/"
  command: []
  concurrentSpawnLimit: 64
  config:
    Authenticator:
      admin_users:
        - "admin"
      allow_all: true
      auto_login: true
    GenericOAuthenticator:
      authorize_url: "https://authentik.example.com/application/o/authorize/"
      client_id: ""
      client_secret: ""
      login_service: "OAuth"
      oauth_callback_url: "https://jupyter.example.com/hub/oauth_callback"
      scope:
        - "openid"
        - "email"
      token_url: "https://authentik.example.com/application/o/token/"
      userdata_url: "https://authentik.example.com/application/o/userinfo/"
      username_claim: "sub"
    JupyterHub:
      admin_access: true
      authenticator_class: "generic-oauth"
    LDAPAuthenticator:
      allowed_groups: []
      bind_dn_template:
        - "uid={username},ou=foo,ou=People,dc=example,dc=com"
      escape_userdn: true
      lookup_dn: false
      server_address: "ldap.example.com"
      use_ssl: true
```
