---
title: jupyterhub-profilelist-custom
date: 2024-09-20 17:51:24
tags:
---

# jupyter hub 集群版（k8s）如何配置启动配置选择，比如可以选择 4G 内存和 8G 内存以及不同 nodeSelector 标签的

在 Kubernetes 上部署 JupyterHub 集群版时，你可以通过配置 JupyterHub 的 Helm chart 来实现启动配置选择，包括内存大小和不同的 `nodeSelector` 标签。以下是一个示例步骤，展示如何实现这些配置：

1. **安装 Helm（如果尚未安装）**:
    ```bash
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    ```

2. **添加 JupyterHub Helm 仓库**:
    ```bash
    helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
    helm repo update
    ```

3. **创建一个自定义的 `config.yaml` 文件**，用于配置 JupyterHub 的 Helm chart：

    ```yaml
    hub:
      extraConfig:
        customConfig: |
          c.KubeSpawner.profile_list = [
              {
                  'display_name': '4G Memory',
                  'description': 'Single user notebook with 4G memory',
                  'kubespawner_override': {
                      'cpu_limit': 1,
                      'mem_limit': '4G',
                      'node_selector': {
                          'disktype': 'ssd'
                      }
                  }
              },
              {
                  'display_name': '8G Memory',
                  'description': 'Single user notebook with 8G memory',
                  'kubespawner_override': {
                      'cpu_limit': 2,
                      'mem_limit': '8G',
                      'node_selector': {
                          'disktype': 'hdd'
                      }
                  }
              }
          ]
    ```

4. **安装 JupyterHub**:

    ```bash
    RELEASE=jhub
    NAMESPACE=jhub

    helm upgrade --install $RELEASE jupyterhub/jupyterhub \
        --namespace $NAMESPACE \
        --create-namespace \
        --version=1.2.0 \
        --values config.yaml
    ```

在上面的配置中，我们定义了两个不同的配置选项：

- **4G Memory**:
  - `cpu_limit`: 1
  - `mem_limit`: 4G
  - `node_selector`: `disktype=ssd`

- **8G Memory**:
  - `cpu_limit`: 2
  - `mem_limit`: 8G
  - `node_selector`: `disktype=hdd`

这些配置将出现在 JupyterHub 的用户界面中，用户可以在启动他们的 Jupyter Notebook 实例时选择相应的配置。

### 详细解释

- **`c.KubeSpawner.profile_list`**: 这是一个列表，包含了多个配置文件，每个配置文件代表一种资源配置选项。
- **`display_name`**: 在 JupyterHub UI 中显示的配置名称。
- **`description`**: 配置选项的描述。
- **`kubespawner_override`**: 用于覆盖默认的 KubeSpawner 配置，包括 CPU 和内存限制、节点选择器等。

### 其他配置项

除了上述基本配置，你还可以根据需要添加更多的配置选项，例如：

- **`cpu_guarantee`** 和 **`mem_guarantee`**：保证的 CPU 和内存资源。
- **`tolerations`**：为 Pod 指定容忍度，以便它们可以调度到特定的节点上。
- **`affinity`**：用于指定 Pod 的亲和性规则。

通过这种方式，你可以灵活地管理和分配集群资源，以满足不同用户的需求。


## 参考链接

- [JupyterHub Helm Chart](https://jupyterhub.github.io/helm-chart/)
- [https://z2jh.jupyter.org/en/latest/resources/reference.html](https://z2jh.jupyter.org/en/latest/resources/reference.html)
- [https://jupyterhub-kubespawner.readthedocs.io/en/latest/spawner.html#kubespawner.KubeSpawner.profile_list](https://jupyterhub-kubespawner.readthedocs.io/en/latest/spawner.html#kubespawner.KubeSpawner.profile_list)
- [https://jupyterhub-kubespawner.readthedocs.io/en/latest/spawner.html#kubespawner.KubeSpawner.mem_limit](https://jupyterhub-kubespawner.readthedocs.io/en/latest/spawner.html#kubespawner.KubeSpawner.mem_limit)