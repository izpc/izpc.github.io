---
title: eks-no-nodes-fix
date: 2024-09-18 18:10:40
tags:
---

# EKS Nodes not showing EKS

EKS 使用管理源无法看到节点机器，这个问题需要配置 aws-auth ConfigMap 来解决。

```yaml

mapRoles:
----
- groups:
  - system:bootstrappers
  - system:nodes
  rolearn: arn:aws:iam::111122223333:role/my-node-role
```

## 参考：

- [https://repost.aws/questions/QUwwHQCZgxQH2pNRDVZYsKWQ/eks-nodes-not-showing](https://repost.aws/questions/QUwwHQCZgxQH2pNRDVZYsKWQ/eks-nodes-not-showing)

- [https://docs.aws.amazon.com/eks/latest/userguide/auth-configmap.html](https://docs.aws.amazon.com/eks/latest/userguide/auth-configmap.html)