---
title: 如何启动私有 LDAP
date: 2024-09-29 15:24:44
tags:
- ldap
- docker
---

# 如何启动私有 LDAP

`docker-compose.yml` 配置文件

```yaml
version: '3.8'

services:
  openldap:
    image: osixia/openldap:1.5.0
    restart: always
    container_name: ldap
    ports:
      - "389:389"
      - "636:636"
    environment:
      - LDAP_ORGANISATION=foo
      - LDAP_DOMAIN=bar.com
      - LDAP_ADMIN_PASSWORD=foobar
      - LDAP_CONFIG_PASSWORD=foobar
    volumes:
      - ${PWD}/ldap:/var/lib/ldap
      - ${PWD}/slapd.d:/etc/ldap/slapd.d
```