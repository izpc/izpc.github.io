---
title: Airflow Authentik Login
date: 2024-05-31 14:22:19
tags: [airflow, authentik, login]
---

Airflow Helm Chart

`App Version: 2.8.3`

`Package Version: 1.13.1`


配置 `webserver_config.py` 如下

# Airflow LDAP login

可以参考[文档](https://airflow.apache.org/docs/apache-airflow/1.10.13/security.html#ldap)


```python
from flask_appbuilder.security.manager import AUTH_LDAP

AUTH_TYPE = AUTH_LDAP

AUTH_LDAP_SERVER = "ldap://ldap.example.com"

AUTH_LDAP_USE_TLS = False

# registration configs

AUTH_USER_REGISTRATION = True  # allow users who are not already in the FAB DB

# Required if not mapping from LDAP DN

AUTH_USER_REGISTRATION_ROLE = "Admin"

# AUTH_LDAP_FIRSTNAME_FIELD = "uid"

AUTH_LDAP_LASTNAME_FIELD = "cn"

# if null in LDAP, email is set to: "{username}@email.notfound"
AUTH_LDAP_EMAIL_FIELD = "mail"


# search configs

# the LDAP search base
AUTH_LDAP_SEARCH = "ou=foo,ou=People,dc=example,dc=com"

AUTH_LDAP_UID_FIELD = "uid"  # the username field

AUTH_LDAP_BIND_USER = "cn=admin,dc=example,dc=com"

AUTH_LDAP_BIND_PASSWORD = "adminPassword"


# a mapping from LDAP DN to a list of FAB roles

AUTH_ROLES_MAPPING = {

    "cn=g-admin,ou=Group,dc=example,dc=com": ["Admin"],

}


# the LDAP user attribute which has their role DNs

AUTH_LDAP_GROUP_FIELD = "memberOf"


# if we should replace ALL the user's roles each login, or only on registration

AUTH_ROLES_SYNC_AT_LOGIN = True


# force users to re-auth after 30min of inactivity (to keep roles in sync)

PERMANENT_SESSION_LIFETIME = 86400
```


# Airflow OAuth login

参考 [文档](https://airflow.apache.org/docs/apache-airflow/2.8.3/security/webserver.html)

具体 [参见](https://airflow.apache.org/docs/apache-airflow-providers-fab/1.1.1/auth-manager/webserver-authentication.html)

## Authentik 认证

有趣的是 Airflow 对于集成 Authentik，有人提供了一个 [PR](https://github.com/apache/airflow/issues/35131), 但这个过于复杂，代码可以[参考](https://github.com/apache/airflow/blob/b8a83b2293f16523b40fab6035fed5f5431076af/airflow/providers/fab/auth_manager/security_manager/override.py#L2268)

我采用了一个简单的方式，通过 OAuth 认证，问题的关键点是在于 userinfo/ 而不是 userinfo , 具体配置如下：

```python
from flask_appbuilder.security.manager import AUTH_OAUTH

from airflow.auth.managers.fab.security_manager.override import
FabAirflowSecurityManagerOverride


AUTH_TYPE = AUTH_OAUTH

AUTH_USER_REGISTRATION = True

AUTH_USER_REGISTRATION_ROLE = "User"

OAUTH_PROVIDERS = [
    {
        "name": "OAuth",
        "icon": "fa-circle-o",
        "token_key": "access_token",
        "remote_app": {
            "client_id": ""
            "api_base_url": "https://authentik.example.com/application/o/",
            "client_secret": "",
            "client_kwargs": {"scope": "openid email profile"},
            "server_metadata_url": "https://authentik.exmpale.com/application/o/oauth/.well-known/openid-configuration",
        },
    }
]

PERMANENT_SESSION_LIFETIME = 259200


class CustomSecurityManager(FabAirflowSecurityManagerOverride):
    def get_oauth_user_info(self, provider, resp):

        if provider != "OAuth":

            return {}

        # Be careful with the URL here, it must be userinfo/ and not user_info
        me = self.appbuilder.sm.oauth_remotes[provider].get("userinfo/")

        data = me.json()

        print(f"data: {data}")

        return {
            "email": data["email"],
            "username": data.get("name", ""),
            "first_name": data.get("given_name", ""),
            "role_keys": data.get("groups", []),
        }


SECURITY_MANAGER_CLASS = CustomSecurityManager

```

## Debug 方式

遗留的一些调试代码：

```python
import sys

remote_app = self.appbuilder.sm.oauth_remotes[provider]

print(f"remote_app: {remote_app}, vars: {vars(remote_app)}")

try:
    id_token = resp["id_token"]
    me = self._get_dingtalk_token_info(id_token)
    print(f"me: {me}")


    me = remote_app.get("userinfo")
    # print get source code
    import inspect
    print(inspect.getsource(remote_app.get))

    print(f"me: {me}, vars: {vars(me)}")


    user_data = me.json()
except Exception as e:
    print(f"Error getting user info: {e}")
    print(f"{sys.exc_info()}")
    traceback.print_exc()

    raise e

print(f"user_data: {user_data}")
```


# Airflow API Auth

API 认证可以通过配置文件进行配置，具体 [参见](https://airflow.apache.org/docs/apache-airflow/2.8.3/security/api.html)

可以通过帐号密码进行认证，不同于 OAuth 认证, 具体配置如下：

```yaml
config:
  api:
    auth_backends: "airflow.api.auth.backend.basic_auth"
```