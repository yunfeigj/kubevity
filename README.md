# Kubevity

> English | [中文](README_zh-CN.md)


There are three scenarios to use Kubevity.

* Install Kubernetes only
* Install Kubernetes first, then deploy Hadoop
* Manager kubernetes apps or hadoop apps


## Motivation

* Ansible-based installer has a bunch of software dependency such as Python. Kubevity is developed in Go language to get rid of the problem in a variety of environment so that increasing the success rate of installation.
* Kubevity uses Kubeadm to install K8s cluster on nodes in parallel as much as possible in order to reduce installation complexity and improve efficiency. It will greatly save installation time compared to the older installer.
* Kubevity supports for scaling cluster from allinone to multi-node cluster, even an HA cluster.
* Kubevity aims to install cluster as an object, i.e., CaaO.

## Supported Environment

### Linux Distributions

* **Ubuntu**  *16.04, 18.04, 20.04*
* **Debian**  *Buster, Stretch*
* **CentOS/RHEL**  *7*
* **SUSE Linux Enterprise Server** *15*


### <span id = "KubernetesVersions">Kubernetes Versions</span> 

* **v1.15**: &ensp; *v1.15.12*
* **v1.16**: &ensp; *v1.16.13*
* **v1.17**: &ensp; *v1.17.9*
* **v1.18**: &ensp; *v1.18.6*
* **v1.19**: &ensp; *v1.19.8*  (default)
* **v1.20**: &ensp; *v1.20.4*
> Looking for more supported versions [Click here](./docs/kubernetes-versions.md)

## Requirements and Recommendations

> /var/lib/docker is mainly used to store the container data, and will gradually increase in size during use and operation. In the case of a production environment, it is recommended that /var/lib/docker mounts a drive separately.

* OS requirements:
  * `SSH` can access to all nodes.
  * Time synchronization for all nodes.
  * `sudo`/`curl`/`openssl` should be used in all nodes.
  * `docker` can be installed by yourself or by KubeKey.
  * `Red Hat` includes `SELinux` in its `Linux release`. It is recommended to close SELinux or [switch the mode of SELinux](./docs/turn-off-SELinux.md) to `Permissive`
> * It's recommended that Your OS is clean (without any other software installed), otherwise there may be conflicts.  
> * A container image mirror (accelerator) is recommended to be prepared if you have trouble downloading images from dockerhub.io. [Configure registry-mirrors for the Docker daemon](https://docs.docker.com/registry/recipes/mirror/#configure-the-docker-daemon).
> * KubeKey will install [OpenEBS](https://openebs.io/) to provision LocalPV for development and testing environment by default, this is convenient for new users. For production, please use NFS / Ceph / GlusterFS  or commercial products as persistent storage, and install the [relevant client](docs/storage-client.md) in all nodes.
> * If you encounter `Permission denied` when copying, it is recommended to check [SELinux and turn off it](./docs/turn-off-SELinux.md) first 

* Dependency requirements:

KubeKey can install Kubernetes and KubeSphere together. The dependency that needs to be installed may be different based on the Kubernetes version to be installed. You can refer to the list below to see if you need to install relevant dependencies on your node in advance.

|             | Kubernetes Version ≥ 1.18 | Kubernetes Version < 1.18 |
| ----------- | ------------------------- | ------------------------- |
| `socat`     | Required                  | Optional but recommended  |
| `conntrack` | Required                  | Optional but recommended  |
| `ebtables`  | Optional but recommended  | Optional but recommended  |
| `ipset`     | Optional but recommended  | Optional but recommended  |

* Networking and DNS requirements:
  * Make sure the DNS address in `/etc/resolv.conf` is available. Otherwise, it may cause some issues of DNS in cluster.
  * If your network configuration uses Firewall or Security Group，you must ensure infrastructure components can communicate with each other through specific ports. It's recommended that you turn off the firewall or follow the link configuriation: [NetworkAccess](docs/network-access.md).

## Usage

### Get the Installer Executable File

* Build Binary from Source Code

    ```shell
    git clone https://github.com/yunfeigj/kubevity.git
    cd kubevity
    ./build.sh
    ```

> Note:
>
> * Docker needs to be installed before building.
> * If you have problem to access `https://proxy.golang.org/`, excute `build.sh -p` instead.

### Create a Cluster

#### Quick Start

Quick Start is for `all-in-one` installation which is a good start to get familiar with Kubevity

> Note: Since Kubernetes temporarily does not support uppercase NodeName, contains uppercase letters in the hostname will lead to subsequent installation error

##### Command

> If you have problem to access `https://storage.googleapis.com`, execute first `export KVZONE=cn`.

```shell script
./kv create cluster [--with-kubernetes version] [--with-hadoop version]
```

##### Examples

* Create a pure Kubernetes cluster with default version.

    ```shell script
    ./kv create cluster
    ```

* Create a Kubernetes cluster with a specified version ([supported versions](#KubernetesVersions)).

    ```shell script
    ./kv create cluster --with-kubernetes v1.19.8
    ```

* Create a Kubernetes cluster with Hadoop installed (e.g. `--with-hadoop v3.1.0`)

    ```shell script
    ./kv create cluster --with-hadoop [version]
    ```

#### Advanced

You have more control to customize parameters or create a multi-node cluster using the advanced installation. Specifically, create a cluster by specifying a configuration file.

> If you have problem to access `https://storage.googleapis.com`, execute first `export KVZONE=cn`.

1. First, create an example configuration file

    ```shell script
    ./kv create config [--with-kubernetes version] [--with-hadoop version] [(-f | --file) path]
    ```

   **examples:**

   * create an example config file with default configurations. You also can specify the file that could be a different filename, or in different folder.

    ```shell script
    ./kv create config [-f ~/myfolder/abc.yaml]
    ```

   * with Hadoop

    ```shell script
    ./kk create config --with-hadoop
    ```

2. Modify the file config-sample.yaml according to your environment
> Note:  Since Kubernetes temporarily does not support uppercase NodeName, contains uppercase letters in workerNode`s name will lead to subsequent installation error
> 
> A persistent storage is required in the cluster, when hadoop will be installed. The local volume is used default. If you want to use other persistent storage, please refer to [addons](./docs/addons.md).
3. Create a cluster using the configuration file

    ```shell script
    ./kv create cluster -f config-sample.yaml
    ```

### Enable Multi-cluster Management

By default, Kubevity will only install a **solo** cluster without Kubernetes federation. If you want to set up a multi-cluster control plane to centrally manage multiple clusters, you need to set the `ClusterRole` in [config-example.yaml](docs/config-example.md). For multi-cluster user guide, please refer to [How to Enable the Multi-cluster Feature](https://gitee.com/YunFeiGuoJi/community/tree/master/sig-multicluster/how-to-setup-multicluster-on-kubesphere).

### Enable Pluggable Components

Kubevity has decoupled some core feature components since v2.1.0. These components are designed to be pluggable which means you can enable them either before or after installation. By default, KubeSphere will be started with a minimal installation if you do not enable them.


### Add Nodes

Add new node's information to the cluster config file, then apply the changes.

```shell script
./kv add nodes -f config-sample.yaml
```

### Delete Nodes

You can delete the node by the following command，the nodeName that needs to be removed.

```shell script
./kv delete node <nodeName> -f config-sample.yaml
```

### Delete Cluster

You can delete the cluster by the following command:

* If you started with the quick start (all-in-one):

```shell script
./kv delete cluster
```

* If you started with the advanced (created with a configuration file):

```shell script
./kv delete cluster [-f config-sample.yaml]
```
### Upgrade Cluster
#### Allinone
Upgrading cluster with a specified version.
```shell script
./kv upgrade [--with-kubernetes version] [--with-hadoop version] 
```
* Support upgrading Kubernetes only.
* Support upgrading Hadoop only.
* Support upgrading Kubernetes and Hadoop.

#### Multi-nodes
Upgrading cluster with a specified configuration file.
```shell script
./kv upgrade [--with-kubernetes version] [--with-hadoop version] [(-f | --file) path]
```
* If `--with-kubernetes` or `--with-hadoop` is specified, the configuration file will be also updated.
* Use `-f` to specify the configuration file which was generated for cluster creation.

> Note: Upgrading multi-nodes cluster need a specified configuration file. If the cluster was installed without kubevity or the configuration file for installation was not found, the configuration file needs to be created by yourself or following command.

Getting cluster info and generating kubekey's configuration file (optional).
```shell script
./kv create config [--from-cluster] [(-f | --file) path] [--kubeconfig path]
```
* `--from-cluster` means fetching cluster's information from an existing cluster. 
* `-f` refers to the path where the configuration file is generated.
* `--kubeconfig` refers to the path where the kubeconfig. 
* After generating the configuration file, some parameters need to be filled in, such as the ssh information of the nodes.

## Documents

* [Configuration example](docs/config-example.md)
* [Addons](docs/addons.md)
* [Network access](docs/network-access.md)
* [Storage clients](docs/storage-client.md)
* [kubectl auto-completion](docs/kubectl-autocompletion.md)
* [kubekey auto-completion](docs/kubevity-autocompletion.md)
* [Roadmap](docs/roadmap.md)
* [Check-Renew-Certificate](docs/check-renew-certificate.md)
* [Developer-Guide](docs/developer-guide.md)

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):
<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!