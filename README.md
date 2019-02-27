# pg-operator
postgresql operator for k8s

## 架构设计

![https://crunchydata.github.io/postgres-operator/stable/OperatorReferenceDiagram.png](https://crunchydata.github.io/postgres-operator/stable/OperatorReferenceDiagram.png)


### 自定义资源定义
Kubernetes自定义资源定义用于PostgreSQL运算符的设计，以定义以下内容：

* 集群 - pgclusters
* 备份 - pgbackups
* 升级 - pgupgrades
* 政策 - pgpolicies
* 任务 - pgtasks
有关Postgres集群部署的元数据存储在这些CRD资源中，这些资源充当了运营商的真实来源。

在Postgres的运营商的设计采用了以下概念：

**事件听众**
在创建，删除或更新新资源时，会为操作员的CRD资源创建Kubernetes事件。运算符处理这些事件以执行异步操作。

捕获事件时，在操作员内执行控制器逻辑以执行大量操作员逻辑。

**REST API**
运营商的一个特征是提供REST API，用户或自定义应用程序可以在其上检查并引起运营商内的操作，例如配置资源或查看资源状态。

此API由RBAC（基于角色的访问控制）安全模型保护，其中每个API调用都具有分配给它的权限。定义API角色以向Operator服务提供精细授权。

**命令行界面**
运算符的一个独特功能是pgo命令行界面（CLI）。普通最终用户使用此工具来创建数据库或集群，或对现有数据库进行更改。

CLI与postgres-operator部署中部署的REST API交互。

**节点亲和力**
如果要使Kubernetes尝试将主群集安排到特定的Kubernetes节点，则可以让Operator将节点关联性部分添加到新的Cluster Deployment。

您可以通过运行以下命令来查看Kube群集上的节点：

`kubectl get node`

然后，您可以在创建集群时指定其中一个名称（例如kubeadm-node2）;

```pgo create cluster thatcluster --node-name=kubeadm-node2```

部署中插入的关联性规则使用首选 策略，以便在节点关闭或不可用时，Kubernetes将继续并在另一个节点上安排Pod。

当您扩展群集并添加副本时，缩放将考虑使用--node-name。如果它发现使用特定节点名创建了集群，则副本部署将添加一个关联规则以尝试计划

**故障转移**
操作员在单个Kubernetes集群中支持手动和自动故障转移。

手动故障转移由涉及查询的API操作执行 ，然后指定目标以选择故障转移副本目标。

操作员通过评估主要服务器的准备情况来执行自动故障转移。可以为所有群集或特定群集全局指定自动故障转移。

用户可以将操作员配置为使用新副本替换发生故障的主服务器（如果他们需要该行为）。

故障转移逻辑包括：

* 删除失败的主要部署
* 选择最好的副本成为新的主要
* 标签更改目标副本服务器以匹配主服务
* 在目标副本上执行PostgreSQL升级命令



## 安装步骤

* 创建一个项目结构
* 配置环境变量
* 配置运营商模板
* 创建安全资源
* 部署运营商
* 安装pgo CLI（最终用户命令工具）

参见文档以及当前仓库的配置（在原基础上已经做了修剪和改动）



## pg operator 常用操作

Operations
The following table shows the pgo operations currently implemented:

OPERATION | SYNTAX | DESCRIPTION
----------|--------|------------
apply | pgo apply mypolicy –selector=name=mycluster | Apply a SQL policy on a Postgres cluster(s)
backup | pgo backup mycluster | Perform a backup on a Postgres cluster(s)
create | pgo create cluster mycluster | Create an Operator resource type (e.g. cluster, policy, schedule, user)
delete | pgo delete cluster mycluster | Delete an Operator resource type (e.g. cluster, policy, user, schedule)
df | pgo df mycluster | Display the disk status/capacity of a Postgres cluster.
failover | pgo failover mycluster | Perform a manual failover of a Postgres cluster.
help | pgo help | Display general pgo help information.
label | pgo label mycluster –label=environment=prod | Create a metadata label for a Postgres cluster(s).
load | pgo load –load-config=load.json –selector=name=mycluster | Perform a data load into a Postgres cluster(s).
reload | pgo reload mycluster | Perform a pg_ctl reload command on a Postgres cluster(s).
restore | pgo restore mycluster | Perform a pgbackrest or pgdump restore on a Postgres cluster.
scale | pgo scale mycluster | Create a Postgres replica(s) for a given Postgres cluster.
scaledown | pgo scaledown mycluster –query | Delete a replica from a Postgres cluster.
show | pgo show cluster mycluster | Display Operator resource information (e.g. cluster, user, policy, schedule).
status | pgo status | Display Operator status.
test | pgo test mycluster | Perform a SQL test on a Postgres cluster(s).
update | pgo update cluster –label=autofail=false | Update a Postgres cluster(s).
upgrade | pgo upgrade mycluster | Perform a minor upgrade to a Postgres cluster(s).
user | pgo user –selector=name=mycluster –update-passwords | Perform Postgres user maintenance on a Postgres cluster(s).
version | pgo version | Display Operator version information.

详情参见： https://crunchydata.github.io/postgres-operator/stable/operator-cli



