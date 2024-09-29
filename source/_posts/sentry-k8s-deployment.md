---
title: Sentry K8s Deployment
date: 2024-05-31 09:58:29
tags: [sentry, k8s, deployment]
---

# Sentry K8s Deployment

Sentry 是一个开源的错误监控系统，可以用来监控应用程序的错误。本文介绍如何在 Kubernetes 集群中部署 Sentry。

官方没有提供 Sentry 的 Kubernetes 部署文档，但是 https://github.com/sentry-kubernetes/charts 提供了 Sentry 的 Helm Chart，可以方便地在 Kubernetes 集群中部署 Sentry。

## 关键配置 values.yaml

App Version: 24.5.0
Package Version: 23.1.0

`values.yaml` 配置如下：

```yaml
filestore:
  backend: filesystem
  filesystem:
    path: /var/lib/sentry/files
    persistence:
      accessMode: ReadWriteMany
      enabled: true
      existingClaim: ""
      persistentWorkers: true
      size: 10Gi
      storageClass: "efs"
nginx:
  ingress:
    annotations: { }
    enabled: true
    hostname: sentry.company.com
    ingressClassName: "alb"
    path: /
    pathType: Prefix
postgresql:
  postgresqlPassword: "examplePassword"
  postgresqlPostgresPassword: "examplePassword"
sourcemaps:
  enabled: true
user:
  create: true
  email: example@company.com
  password: examplePassword
zookeeper:
  autopurge:
    purgeInterval: 3
    snapRetainCount: 3
```

### 注意几点：

1. [filestore](https://github.com/sentry-kubernetes/charts?tab=readme-ov-file#persistence) , 需要配置 storageClass 为 ReadWriteMany 的存储类。
2. `nginx` 配置 annotation, 因为我们使用的是 AWS 的 ALB Ingress Controller。
3. `postgresql` 配置数据库密码， 防止启动时报错。[具体参见](https://github.com/sentry-kubernetes/charts/issues/571#issuecomment-1039616281), [清理方式](https://github.com/sentry-kubernetes/charts/issues/799)
4. `sourcemaps` 配置为 true，开启 sourcemaps 功能。
5. `user` 配置为 true，创建一个初始用户(admin)。
6. `zookeeper` 配置 autopurge, 避免 Zookeeper 数据过大。


### 重点注意：

部署成功后，会出现一个 `sentry-clickhouse` 的 pod，但是会一直处于 `CrashLoopBackOff` 状态

查看日志，发现如下错误：

```bash
snuba.clickhouse.errors.ClickhouseWriterError: Method write is not supported by storage Distributed with more than one shard and no sharding key provided

```

具体原因和解决方案 [参见](https://github.com/getsentry/snuba/issues/4897) 以及 [issue](https://github.com/sentry-kubernetes/charts/issues/1272) 和提及到的 [bug](https://github.com/sentry-kubernetes/charts/issues/1042)

解决方案：

```bash
kubectl exec -it sentry-clickhouse-0 -- bash
clickhouse-client -h sentry-clickhouse

DROP table default.metrics_raw_v2_dist ON CLUSTER 'sentry-clickhouse' SYNC;
CREATE TABLE default.metrics_raw_v2_dist ON CLUSTER 'sentry-clickhouse'
(
    `use_case_id` LowCardinality(String),
    `org_id` UInt64,
    `project_id` UInt64,
    `metric_id` UInt64,
    `timestamp` DateTime,
    `tags.key` Array(UInt64),
    `tags.value` Array(UInt64),
    `metric_type` LowCardinality(String),
    `set_values` Array(UInt64),
    `count_value` Float64,
    `distribution_values` Array(Float64),
    `materialization_version` UInt8,
    `retention_days` UInt16,
    `partition` UInt16,
    `offset` UInt64,
    `timeseries_id` UInt32
)
ENGINE = Distributed('sentry-clickhouse', 'default', 'metrics_raw_v2_local', sipHash64('timeseries_id'));

```


# 替代方案

[GlitchTip](https://glitchtip.com/) 是一个 Sentry 的替代方案，可以考虑使用。