---
title: 让 JupyterHub 支持 LDAP
date: 2024-03-21 09:25:03
tags: [JupyterHub, LDAP]
---

JupyterHub 其实是支持的 LDAP 认证，但是模式是 LDAPS，对于没有 LDAP 非加密端口则不支持

下面是一段 Dockerfile, 用于修复这个问题
    
```Dockerfile
FROM jupyterhub/k8s-hub:latest

USER root 
# Fix LDAP authenticator use_ssl issue
RUN FILEPATH=`python -c "import pkg_resources; import os; print(os.path.join(pkg_resources.get_distribution('jupyterhub-ldapauthenticator').location, 'ldapauthenticator', 'ldapauthenticator.py'))"` && \
sed -i 's/ldap3.AUTO_BIND_NO_TLS if self.use_ssl else ldap3.AUTO_BIND_TLS_BEFORE_BIND/ldap3.AUTO_BIND_NO_TLS if not self.use_ssl else ldap3.AUTO_BIND_TLS_BEFORE_BIND/g' ${FILEPATH} 

USER jovyan
```
