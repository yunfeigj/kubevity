# KubeKey

> [English](README.md) | 中文


有三种情况可以使用 Kubevity。

* 仅安装 Kubernetes
* 用一个命令中安装 Kubernetes 和 Hadoop
* 维护


## 优势

* 基于 Ansible 的安装程序具有大量软件依赖性，例如 Python。Kubevity 是使用 Go 语言开发的，可以消除在各种环境中出现的问题，从而提高安装成功率。
* Kubevity 使用 Kubeadm 在节点上尽可能多地并行安装 K8s 集群，以降低安装复杂性并提高效率。与较早的安装程序相比，它将大大节省安装时间。
* Kubevity 支持将集群从 all-in-one 扩展到多节点集群甚至 HA 集群。
* Kubevity 旨在将集群当作一个对象操作，即 CaaO。

## 支持的环境

### Linux 发行版

* **Ubuntu**  *16.04, 18.04, 20.04*
* **Debian**  *Buster, Stretch*
* **CentOS/RHEL**  *7*
* **SUSE Linux Enterprise Server** *15*


### <span id = "KubernetesVersions">Kubernetes 版本</span> 

* **v1.15**: &ensp; *v1.15.12*
* **v1.16**: &ensp; *v1.16.13*
* **v1.17**: &ensp; *v1.17.9*
* **v1.18**: &ensp; *v1.18.6*
* **v1.19**: &ensp; *v1.19.8*  (default)
* **v1.20**: &ensp; *v1.20.4*
> 查看更多支持的版本[点击这里](./docs/kubernetes-versions.md)

## 要求和建议

> /var/lib/docker 主要用于存储容器数据，在使用和操作过程中会逐渐增大。对于生产环境，建议 /var/lib/docker 单独挂盘。

* 操作系统要求：
  * `SSH` 可以访问所有节点。
  * 所有节点的时间同步。
  * `sudo`/`curl`/`openssl` 应在所有节点使用。
  * `docker` 可以自己安装，也可以通过 KubeKey 安装。
  * `Red Hat` 在其 `Linux` 发行版本中包括了`SELinux`，建议[关闭SELinux](./docs/turn-off-SELinux_zh-CN.md)或者将[SELinux的模式切换](./docs/turn-off-SELinux_zh-CN.md)为Permissive[宽容]工作模式

> * 建议您的操作系统环境足够干净 (不安装任何其他软件)，否则可能会发生冲突。
> * 如果在从 dockerhub.io 下载镜像时遇到问题，建议准备一个容器镜像仓库 (加速器)。[为 Docker 守护程序配置镜像加速](https://docs.docker.com/registry/recipes/mirror/#configure-the-docker-daemon)。
> * 默认情况下，Kubevity 将安装 [OpenEBS](https://openebs.io/) 来为开发和测试环境配置 LocalPV，这对新用户来说非常方便。对于生产，请使用 NFS/Ceph/GlusterFS 或商业化存储作为持久化存储，并在所有节点中安装[相关的客户端](./docs/storage-client.md) 。
> * 如果遇到拷贝时报权限问题Permission denied,建议优先考虑查看[SELinux的原因](./docs/turn-off-SELinux_zh-CN.md)。

* 依赖要求:

Kube 可以同时安装 Kubernetes 和 Hadoop。根据 kubernetes 所安装版本的不同，您所需要安装的依赖可能也不同。请参考以下表格查看您是否需要提前在节点上安装有关的依赖。

|             | Kubernetes 版本 ≥ 1.18 | Kubernetes 版本 < 1.18 |
| ----------- | ---------------------- | ---------------------- |
| `socat`     | 必须安装               | 可选，但推荐安装       |
| `conntrack` | 必须安装               | 可选，但推荐安装       |
| `ebtables`  | 可选，但推荐安装       | 可选，但推荐安装       |
| `ipset`     | 可选，但推荐安装       | 可选，但推荐安装       |

* 网络和 DNS 要求：
  * 确保 `/etc/resolv.conf` 中的 DNS 地址可用。否则，可能会导致集群中出现某些 DNS 问题。
  * 如果您的网络配置使用防火墙或安全组，则必须确保基础结构组件可以通过特定端口相互通信。建议您关闭防火墙或遵循链接配置：[网络访问](docs/network-access.md)。

## 用法

### 获取安装程序可执行文件

* 从源代码生成二进制文件

    ```shell script
    git clone https://github.com/yunfeigj/kubevity.git
    cd kubevity
    ./build.sh
    ```

> 注意：
>
> * 在构建之前，需要先安装 Docker。
> * 如果无法访问 `https://proxy.golang.org/`，比如在大陆，请执行 `build.sh -p`。

### 创建集群

#### 快速开始

快速入门使用 `all-in-one` 安装，这是熟悉 Kubevity 的良好开始。

> 注意： 由于 Kubernetes 暂不支持大写 NodeName， hostname 中包含大写字母将导致后续安装过程无法正常结束

##### 命令

> 如果无法访问 `https://storage.googleapis.com`, 请先执行 `export KVZONE=cn`.

```shell script
./kv create cluster [--with-kubernetes version] [--with-hadoop version]
```

##### 例子

* 使用默认版本创建一个纯 Kubernetes 集群

    ```shell script
    ./kv create cluster
    ```

* 创建指定一个（[支持的版本](#KubernetesVersions)）的 Kubernetes 集群

    ```shell script
    ./kv create cluster --with-kubernetes v1.19.8
    ```

* 创建一个部署了 Hadoop 的 Kubernetes 集群 （例如 `--with-hadoop v3.1.0`）

    ```shell script
    ./kv create cluster --with-hadoop [version]
    ```

#### 高级用法

您可以使用高级安装来控制自定义参数或创建多节点集群。具体来说，通过指定配置文件来创建集群。

> 如果无法访问 `https://storage.googleapis.com`, 请先执行 `export KVZONE=cn`.

1. 首先，创建一个示例配置文件

    ```shell script
    ./kv create config [--with-kubernetes version] [--with-hadoop version] [(-f | --file) path]
    ```

   **例子：**

   * 使用默认配置创建一个示例配置文件。您也可以指定文件名称或文件所在的文件夹。

        ```shell script
        ./kv create config [-f ~/myfolder/config-sample.yaml]
        ```

   * 同时安装 hadoop

        ```shell script
        ./kv create config --with-kubesphere
        ```

2. 根据您的环境修改配置文件 config-sample.yaml
> 注意： 由于 Kubernetes 暂不支持大写 NodeName， worker 节点名中包含大写字母将导致后续安装过程无法正常结束
>
> 当指定安装KubeSphere时，要求集群中有可用的持久化存储。默认使用localVolume，如果需要使用其他持久化存储，请参阅 [addons](./docs/addons.md) 配置。
3. 使用配置文件创建集群。

      ```shell script
      ./kv create cluster -f ~/myfolder/config-sample.yaml
      ```

### 启用多集群管理

默认情况下，Kubevity 将仅安装一个 Solo 模式的单集群，即未开启 Kubernetes 多集群联邦。如果您希望将 KubeSphere 作为一个支持多集群集中管理的中央面板，您需要在 [config-example.yaml](docs/config-example.md) 中设置 `ClusterRole`。关于多集群的使用文档，请参考 [如何启用多集群](https://gitee.com/YunFeiGuoJi/community/tree/master/sig-multicluster/how-to-setup-multicluster)。

### 开启可插拔功能组件

Kubevity 对 Installer 的各功能组件进行了解耦，快速安装将默认仅开启最小化安装（Minimal Installation），Installer 支持在安装前或安装后自定义可插拔的功能组件的安装。使最小化安装更快速轻量且资源占用更少，也方便不同用户按需选择安装不同的功能组件。

### 添加节点

将新节点的信息添加到集群配置文件，然后应用更改。

```shell script
./kv add nodes -f config-sample.yaml
```
### 删除节点

通过以下命令删除节点，nodename指需要删除的节点名。

```shell script
./kv delete node <nodeName> -f config-sample.yaml
```

### 删除集群

您可以通过以下命令删除集群：

* 如果您以快速入门（all-in-one）开始：

```shell script
./kv delete cluster
```

* 如果从高级安装开始（使用配置文件创建的集群）：

```shell script
./kv delete cluster [-f config-sample.yaml]
```

### 集群升级
#### 单节点集群
升级集群到指定版本。
```shell script
./kv upgrade [--with-kubernetes version] [--with-hadoop version] 
```
* `--with-kubernetes` 指定kubernetes目标版本。
* `--with-hadoop` 指定hadoop目标版本。

#### 多节点集群
通过指定配置文件对集群进行升级。
```shell script
./kv upgrade [--with-kubernetes version] [--with-hadoop version] [(-f | --file) path]
```
* `--with-kubernetes` 指定kubernetes目标版本。
* `--with-hadoop` 指定hadoop目标版本。
* `-f` 指定集群安装时创建的配置文件。

> 注意: 升级多节点集群需要指定配置文件. 如果集群非kubekey创建，或者创建集群时生成的配置文件丢失，需要重新生成配置文件，或使用以下方法生成。

Getting cluster info and generating kubevity's configuration file (optional).
```shell script
./kv create config [--from-cluster] [(-f | --file) path] [--kubeconfig path]
```
* `--from-cluster` 根据已存在集群信息生成配置文件. 
* `-f` 指定生成配置文件路径.
* `--kubeconfig` 指定集群kubeconfig文件. 
* 由于无法全面获取集群配置，生成配置文件后，请根据集群实际信息补全配置文件。

### 启用 kubectl 自动补全

Kubevity 不会启用 kubectl 自动补全功能。请参阅下面的指南并将其打开：

**先决条件**：确保已安装 `bash-autocompletion` 并可以正常工作。

```shell script
# 安装 bash-completion
apt-get install bash-completion

# 将 completion 脚本添加到你的 ~/.bashrc 文件
echo 'source <(kubectl completion bash)' >>~/.bashrc

# 将 completion 脚本添加到 /etc/bash_completion.d 目录
kubectl completion bash >/etc/bash_completion.d/kubectl
```

## 相关文档

* [配置示例](docs/config-example.md)
* [自定义插件安装](docs/addons.md)
* [网络访问](docs/network-access.md)
* [存储客户端](docs/storage-client.md)
* [路线图](docs/roadmap.md)
* [查看或更新证书](docs/check-renew-certificate.md)
* [开发指南](docs/developer-guide.md)

## 贡献者 ✨

欢迎任何形式的贡献! 感谢这些优秀的贡献者，是他们让我们的项目快速成长。

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
  </tr>

</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->