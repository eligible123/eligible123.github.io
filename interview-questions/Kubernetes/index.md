---
title: Kubernetes面试题
date: 2026-06-07
type: page
---

# Kubernetes面试题

> 共 25 道面试题

## 1. Kubernetes 中的 DaemonSet 有什么作用？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

# DaemonSet 的作用

## 核心作用

DaemonSet 是 Kubernetes 中的一种工作负载控制器，确保**集群中每个（或指定的）节点上都运行一个 Pod 副本**。当新节点加入集群时，Pod 会自动部署到该节点；节点被移除时，对应的 Pod 也会被回收。

## 核心特点

- **节点级覆盖**：与 Deployment 指定副本数不同，DaemonSet 的副本数随节点数量自动调整
- **自动调度**：新增节点时自动创建 Pod，无需人工干预
- **灵活筛选**：可通过 `nodeSelector` 或 `nodeAffinity` 指定只在特定节点上运行

## 典型使用场景

1. **日志收集**：如 Fluentd、Filebeat，在每个节点采集容器日志
2. **节点监控**：如 Prometheus Node Exporter、cAdvisor，采集节点级指标
3. **网络插件**：如 Calico、Flannel、Cilium，确保每个节点都有网络组件
4. **存储守护进程**：如 Ceph、GlusterFS，在每个节点运行存储代理

## 与 Deployment 的区别

| 特性 | Deployment | DaemonSet |
|------|-----------|-----------|
| 副本策略 | 指定数量 | 每个节点一个 |
| 适用场景 | 无状态应用 | 节点级守护进程 |
| 扩缩容 | 手动/自动 | 随节点自动变化 |

简言之，DaemonSet 是实现**"每个节点都要跑"**这类需求的标准方案，是集群基础设施层组件部署的核心手段。

---

## 2. Kubernetes 中的 Deployment 和 StatefulSet 有什么区别？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Kubernetes 中的 **Deployment** 和 **StatefulSet** 都是用来管理 Pod 集合的工作负载资源，但它们面向的应用类型和管理方式有本质区别。

**Deployment** 主要用于管理**无状态应用**。它不关注每个 Pod 的身份和顺序，Pod 的名称、IP 地址都是随机的，可以随意替换。它擅长进行滚动更新、扩缩容和版本回滚，非常适合部署像 Web 服务器、微服务等不需要持久化存储或稳定网络标识的应用。

**StatefulSet** 则专为管理**有状态应用**而设计。它为每个 Pod 提供了一个稳定、唯一的网络标识（通过 hostname 和稳定的 DNS 记录）和持久化存储卷（即使 Pod 被重新调度，卷也会重新挂载）。Pod 的创建、更新和删除是**有序的**（例如，按名称顺序创建）。这些特性使其非常适合部署数据库（如 MySQL、Redis 集群）、分布式存储系统等需要稳定身份和持久化状态的应用。

简而言之，关键区别在于：Deployment 面向无状态、Pod 可互换；StatefulSet 面向有状态，为每个 Pod 保持唯一身份、顺序和稳定存储。

---

## 3. Kubernetes 中的 Ingress 资源有什么作用？如何配置？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

## Kubernetes Ingress 资源的作用与配置

### 一、Ingress 的作用

Ingress 是 Kubernetes 中管理**集群外部访问内部服务**的 API 资源，主要功能包括：

1. **HTTP/HTTPS 路由**：基于域名和路径将流量转发到不同 Service
2. **SSL/TLS 终止**：统一处理 HTTPS 证书，后端服务可使用 HTTP
3. **负载均衡**：将请求分发到多个 Pod 实例
4. **虚拟主机托管**：单个公网 IP 托管多个域名服务
5. **减少端口暴露**：只需开放 80/443 端口，提升安全性

### 二、配置方法

**1. 部署 Ingress Controller**（如 nginx-ingress、traefik）

**2. 创建 Ingress 资源**：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### 三、核心要点

- Ingress 资源本身只是**声明式配置**，实际工作由 Ingress Controller 完成
- 支持精确路径（Exact）、前缀匹配（Prefix）等路由策略
- 通过 Annotations 可实现限流、重写、CORS 等高级功能
- 生产环境建议配合 cert-manager 实现证书自动管理

---

## 4. Kubernetes 中的 Job 和 CronJob 有什么区别？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Kubernetes 中 Job 和 CronJob 都是用于批处理任务的工作负载资源，核心区别在于**执行模式**和**调度方式**。

**Job** 用于执行**一次性**的批处理任务。它会创建一个或多个 Pod，并持续运行直至任务成功完成（指定数量的 Pod 成功终止）。Job 确保任务完成，即使 Pod 故障也会自动重启。适用于数据处理、报表生成、机器学习训练等非周期性任务。

**CronJob** 用于创建**定时**任务。它基于 **cron 表达式**周期性调度执行 Job。每个调度周期会创建一个新的 Job 对象来运行。适用于定期备份、日志轮转、发送汇总报告等周期性运维工作。

**关键区别总结**：
1.  **调度逻辑**：Job 手动创建或由其他控制器触发；CronJob 由时间调度自动触发。
2.  **执行频率**：Job 通常执行一次；CronJob 按计划重复执行。
3.  **资源对象**：CronJob 是 Job 的模板和调度器，负责按时生成 Job 实例。

简单说，**Job 解决“做一次”的问题，CronJob 解决“定时做”的问题**。两者协同，构成了 Kubernetes 完整的批处理任务管理能力。

---

## 5. Kubernetes 中的 Persistent Volume 和 Persistent Volume Claim 有什么区别？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

PV（Persistent Volume）是集群级别的存储资源，代表实际的物理或虚拟存储设备（如NFS、云盘），由管理员预先创建和配置。PVC（Persistent Volume Claim）是用户对存储的请求，通过声明所需容量、访问模式等条件，由系统动态绑定到合适的PV。

两者核心区别在于抽象层级：PV是存储资源的供应侧，描述“我有什么”；PVC是存储资源的需求侧，表达“我需要什么”。PV生命周期独立于Pod，由管理员管理；PVC则与用户应用绑定，实现存储资源与使用逻辑的解耦。

典型工作流程为：管理员创建PV资源池 → 用户提交PVC申请 → Kubernetes自动匹配并绑定PV与PVC → Pod通过挂载PVC使用存储。这种设计实现了存储的统一管理、动态分配和跨团队复用。

---

## 6. Kubernetes 中的 Pod 是什么？其作用是什么？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Pod 是 Kubernetes 中最小的可部署单元，它是一组共享网络和存储资源的容器集合。Pod 内的容器运行在同一主机上，共享相同的 IP 地址和端口空间，可以通过 localhost 通信。

Pod 的主要作用包括：
1. **部署单元**：作为应用部署的基本单位，一个 Pod 可以包含一个或多个紧密关联的容器（如主应用容器与辅助 sidecar 容器）。
2. **资源共享**：Pod 内容器共享 Volume 和网络命名空间，便于数据交换和服务协作。
3. **生命周期管理**：Kubernetes 以 Pod 为整体进行调度、健康检查和重启策略管理，确保应用高可用。
4. **环境抽象**：Pod 提供了一致的运行环境，屏蔽底层基础设施差异，简化应用部署。

实践中，通常通过 Deployment 等控制器管理 Pod 的副本数和滚动更新，直接创建独立 Pod 的场景较少。Pod 的临时性特点要求将状态数据存储在持久化卷中，避免因 Pod 重建导致数据丢失。

---

## 7. Kubernetes 中的 ReplicaSet 和 ReplicationController 有什么区别？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

**ReplicaSet** 与 **ReplicationController** 均用于确保指定数量的Pod副本持续运行，但二者存在关键区别：

1.  **选择器**：这是最核心的差异。ReplicationController仅支持基于等式的标签选择器（如 `environment = production`）。而ReplicaSet支持更强大的集合选择器（如 `environment in (production, qa)`），提供了更灵活、复杂的Pod匹配能力。

2.  **更新与推荐**：ReplicaSet是ReplicationController的下一代替代品。在Kubernetes中，**直接创建和管理ReplicaSet是当前推荐的做法**。虽然ReplicationController仍能工作，但新功能和优化都集中在ReplicaSet上。

3.  **使用方式**：在实践中，通常不直接使用ReplicaSet。我们更常使用 **Deployment** 对象，它会自动创建和管理ReplicaSet，从而提供声明式更新、回滚等高级功能。因此，理解ReplicaSet是理解Deployment工作原理的基础。

**总结**：两者功能相似，但ReplicaSet通过支持集合选择器更具优势，是Kubernetes当前推荐的Pod副本管理控制器。在面试中，强调选择器的差异和ReplicaSet作为现代实践标准是关键。

---

## 8. Kubernetes 中的 Service 有哪几种类型？请分别简述。

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Kubernetes Service 是一种抽象，用于定义一组 Pod 的逻辑集合和访问策略，确保服务发现和负载均衡。主要有以下几种类型：

1. **ClusterIP**：默认类型，只分配一个集群内部的虚拟 IP 地址，仅在集群内部可访问。适用于内部服务间通信，提供简单的网络暴露。

2.

---

## 9. Kubernetes 中的网络策略（Network Policy）如何实现？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Kubernetes 中的网络策略通过 **NetworkPolicy** 资源实现，其核心是基于 Pod 标签定义**入站（Ingress）和出站（Egress）** 的访问控制规则。

实现分为两个层面：
1.  **策略声明**：用户通过 YAML 文件定义 NetworkPolicy，指定应用于哪些 Pod（`podSelector`）、允许的流量类型（`policyTypes`），以及具体的白名单规则（如来自或去往哪些特定 Pod、命名空间或 IP 段）。
2.  **策略执行**：Kubernetes API 本身不直接执行策略，而是需要**兼容的网络插件（CNI）** 来实现。常见的实现方式有：
    *   **通过 Linux 内核功能**：如 Calico 使用 iptables 或 eBPF；Weave Net 使用 iptables。
    *   **通过 SDN 控制器**：如 Calico 的 Felix、Cilium 等，这些控制器会 watch API Server 上的策略变更，并在节点上编程配置相应的网络规则（如 iptables、eBPF maps）。

**核心生效逻辑**：一旦某个 Namespace 中为特定 Pod 创建了 NetworkPolicy，该 Pod 将**默认拒绝所有未明确允许的流量**。策略按“与”逻辑累加。网络插件在节点上拦截、检查数据包，并根据策略规则决定是放行还是丢弃。

---

## 10. Kubernetes 中，ConfigMap 和 Secret 的作用是什么？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

ConfigMap 和 Secret 是 Kubernetes 中用于管理应用配置和敏感信息的核心资源对象，其主要作用是将配置数据与容器镜像解耦，提升应用部署的灵活性与安全性。

**ConfigMap** 用于存储非机密的键值对配置数据。它可以将应用所需的配置（如数据库地址、参数设置、特性开关等）以文件或环境变量的形式注入到 Pod 中。这使得同一镜像能够通过不同的 ConfigMap 适配开发、测试、生产等多种环境，实现了配置与代码的分离。

**Secret** 专门用于存储和管理敏感数据，例如密码、TLS 证书、API 密钥等。其内容在 etcd 中会被加密存储，并支持以文件或环境变量方式挂载到 Pod 内。与 ConfigMap 不同，Secret 的值通常进行 Base64 编码，为敏感信息提供了更安全的存储和传递机制。

两者都支持动态更新，当配置变更时，可以无需重新构建镜像或重启 Pod（对于挂载为卷的情况）而实现热更新，显著提高了运维效率。共同构成了 Kubernetes 中配置管理的最佳实践。

---

## 11. Kubernetes 的 Helm Charts 如何实现应用的版本控制？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Helm Charts 通过 Chart.yaml 文件和 Chart 版本管理机制实现应用版本控制。每个 Chart 的 Chart.yaml 文件中包含 `version`（Chart 本身的版本）和 `appVersion`（应用版本）字段，遵循语义化版本控制规范。

在部署时，Helm 为每个 Release 生成唯一的修订版本号，并记录部署历史，支持通过 `helm rollback` 回滚到任意历史版本。Chart 可被打包为 `.tgz` 文件并存储在 Chart 仓库中，用户可指定版本拉取特定 Chart，从而实现应用版本的可控管理和可重复部署。

---

## 12. 在 Kubernetes 中，如何实现持久化存储（Persistent Storage）？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

在 Kubernetes 中，实现持久化存储的核心机制是将存储资源的**生命周期**与 Pod **解耦**。主要通过以下三个核心概念和工作流程实现：

1.  **核心抽象组件**：
    *   **PersistentVolume (PV)**：集群管理员预先或通过动态供应创建的一块**实际存储资源**，它独立于 Pod 存在，包含存储类型、容量、访问模式等详情。
    *   **PersistentVolumeClaim (PVC)**：由用户（开发者）提出的对存储资源的**请求**，它声明所需的容量和访问模式，但不关心具体来自哪个 PV。

2.  **使用与绑定流程**：
    *   用户创建一个 PVC。
    *   Kubernetes **控制平面**（特别是 `kube-controller-manager`）中的 PV 控制器会根据 PVC 的声明，在集群中寻找一个匹配（容量、访问模式等）的 PV。
    *   找到匹配的 PV 后，系统将该 PV **绑定** 到该 PVC，此时存储被“预定”。
    *   最后，在 Pod 的配置文件中通过 `volumes` 字段引用该 PVC，Kubernetes 会自动将绑定的 PV 挂载到 Pod 中的指定路径。即使 Pod 重启或迁移，存储卷依然存在。

3.  **高级实践（动态供应）**：
    *   在实际生产中，通过 **StorageClass** 可以定义存储类别（如 SSD、 HDD、 AWS EBS 等），并指定动态供应程序。
    *   当用户创建 PVC 时，可以指定 StorageClass。如果 PV 尚未就绪，**动态供应器**会根据 StorageClass 的定义，自动调用底层云平台或存储系统的 API 来**动态创建 PV** 并完成绑定，极大地简化了存储管理。

**总结**：通过 PV、PVC 的声明式绑定模型，Kubernetes 为应用提供了与存储实现无关的抽象接口，再结合 StorageClass 的动态供应，实现了灵活、可扩展的持久化存储管理。

---

## 13. 如何在 Kubernetes 中使用 Helm 部署应用？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

Helm 是 Kubernetes 的包管理工具，它将相关资源（Deployment、Service、ConfigMap 等）打包为一个 **Chart**，实现应用的模板化、版本化和可重复部署。

**核心步骤：**
1.  **准备 Chart**：可从公共仓库（如 Artifact Hub）获取，或用 `helm create` 命令自建。Chart 包含 `templates`（资源模板）和 `values.yaml`（默认配置）。
2.  **配置与安装**：通过 `-f` 或 `--set` 参数覆盖 `values.yaml` 中的默认值（如镜像标签、副本数），然后使用 `helm install [RELEASE_NAME] [CHART]` 命令部署。Helm 会渲染模板并向 API Server 发送请求，创建一个名为 **Release** 的部署实例。
3.  **管理与迭代**：使用 `helm upgrade` 更新应用（如镜像版本），`helm rollback` 回滚至历史版本，`helm uninstall` 卸载。这些操作均基于 Release 的历史记录进行管理。

**关键价值与最佳实践：**
*   **标准化与复用**：将复杂应用部署抽象为一个 Chart，便于跨环境（dev/staging/prod）一致性部署。
*   **配置解耦**：通过 `values.yaml` 管理配置，实现应用代码与部署配置分离。
*   **发布管理**：支持应用的整个生命周期管理，包括升级、回滚和审计。

在面试中强调 Helm 如何通过模板化、版本控制和声明式管理，解决了 Kubernetes 原生清单文件难以维护和复用的问题，是实现 DevOps 和 GitOps 的核心工具之一。

---

## 14. 如何在 Kubernetes 中实现服务的自动伸缩（autoscaling）？

🟡 **中等** | 标签：运维, DevOps, Kubernetes

在 Kubernetes 中，实现服务的自动伸缩主要依赖三种机制：

**1. 水平 Pod 自动伸缩**
这是最常用的方式，基于 CPU、内存或自定义指标（如每秒请求数）自动调整 Pod 的副本数量。当指标超过阈值时，HPA 控制器会增加 Pod；负载降低后则会缩容。需先部署 Metrics Server 以提供指标数据。

**2. 垂直 Pod 自动伸缩**
VPA 自动调整 Pod 的 CPU 和内存请求/限制，而非副本数。它通过历史指标推荐资源配置，并可在新 Pod 创建时应用，适用于有状态或单实例服务。

**3. 集群节点自动伸缩**
当现有节点资源不足时，Cluster Autoscaler 会自动添加新节点；当节点资源长期闲置时，则移除节点。需配合云厂商的节点池功能使用。

**实践建议：**  
- HPA 是首选方案，适合无状态服务。  
- VPA 适用于资源需求波动大、不便水平扩展的场景。  
- 生产环境中通常组合使用 HPA 与 Cluster Autoscaler，实现 Pod 和节点的协同伸缩。  
- 务必设置合理的伸缩边界，避免因指标波动导致频繁伸缩。

---

## 15. 描述在 Kubernetes 中如何进行日志管理，并解释常用的方法。

🟡 **中等** | 标签：运维, Kubernetes

在Kubernetes中，日志管理至关重要，其核心挑战在于容器应用生命周期短暂、Pod调度动态变化。常用方法主要围绕日志的**采集、存储和分析**展开。

主要方法分为三类：
1.  **节点级日志代理**：在每个节点上部署守护进程（如Fluentd、Filebeat），通过DaemonSet统一收集该节点上所有容器的标准输出（stdout/stderr）日志。这是最基础、最常见的方式，架构简单。
2.  **Sidecar容器**：在每个应用Pod中部署一个专用的日志收集容器（如Filebeat），将应用容器的日志文件转发至后端存储。适用于应用日志需复杂处理或写入特定文件的场景，但会增加资源开销。
3.  **应用直接输出**：应用代码直接集成日志库，将日志发送到集中式日志服务（如Elasticsearch、Loki）。耦合度最高，控制力强，但修改应用代码成本较大。

实践中，常采用 **“节点级代理 + 中心化存储分析”** 的成熟方案。例如，使用DaemonSet部署Fluentd收集所有节点日志，发送至Elasticsearch进行存储和索引，最后通过Kibana进行可视化查询与报警。该方案平衡了运维成本、资源开销与管理效率，是生产环境的常见选择。选择何种方法需根据应用架构、团队习惯和资源规模综合权衡。

---

## 16. 描述在 Kubernetes 中如何进行滚动更新和回滚操作。

🟡 **中等** | 标签：运维, Kubernetes

在 Kubernetes 中，滚动更新和回滚主要通过 Deployment 资源进行管理，其核心是确保应用在更新过程中保持可用性。

**滚动更新：**
滚动更新是 Deployment 的默认更新策略。当我们修改 Deployment 的 Pod 模板（如更新镜像版本）并应用更改后，Kubernetes 会自动启动一个逐步替换旧 Pod 的过程。它通过 `maxSurge` 和 `maxUnavailable` 两个参数控制更新节奏。`maxSurge` 决定了可以超出期望副本数创建的 Pod 数量，`maxUnavailable` 决定了在更新过程中允许不可用的最大 Pod 数。例如，一个有3个副本的Deployment，设置 `maxSurge: 1, maxUnavailable: 0` 时，Kubernetes 会先创建1个新Pod，待其就绪后，再终止1个旧Pod，如此循环直至所有Pod更新完成，确保服务始终有足够副本可用。

**回滚操作：**
如果更新后发现问题，可以使用回滚。Deployment 会保留更新历史记录。通过 `kubectl rollout undo deployment/<deployment-name>` 命令即可回滚到上一个版本。如果需要回滚到特定历史版本，可以使用 `--to-revision=<revision>` 参数指定。执行回滚后，Kubernetes 会按照滚动更新的策略，将 Pod 逐步恢复到指定的旧版本状态。

**常用命令：**
- `kubectl rollout status`：查看更新状态。
- `kubectl rollout history`：查看历史版本记录。
- `kubectl rollout pause` / `resume`：暂停或恢复更新过程。

这种机制使得应用更新变得安全可控，是保障生产环境服务连续性的重要手段。

---

## 17. 描述在 Kubernetes 中如何配置资源配额，并解释其作用。

🟡 **中等** | 标签：运维, Kubernetes

在 Kubernetes 中，资源配额（ResourceQuota）用于控制一个命名空间（Namespace）内所有工作负载所能消耗的总资源量。其核心作用是防止某个团队或应用独占集群资源，确保资源公平分配，并有助于容量规划和成本控制。

配置资源配额通常涉及三个步骤：
1.  **创建 ResourceQuota 对象**：定义配额的具体规则。
2.  **指定限制的资源类型**：可以包括计算资源（如CPU、内存）、存储资源（如持久卷声明数量、总存储量）以及对象数量（如Pod、Service、ConfigMap的数量）。
3.  **应用到特定的命名空间**：ResourceQuota 的生效范围是它所在的命名空间。

以下是一个配置示例：
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-project-quota
  namespace: dev-team # 应用到此命名空间
spec:
  hard:
    requests.cpu: "10"    # 所有Pod的CPU请求总和不能超过10核
    requests.memory: "20Gi" # 内存请求总和不超过20GiB
    limits.cpu: "20"      # CPU限制总和不超过20核
    limits.memory: "40Gi" # 内存限制总和不超过40GiB
    pods: "50"            # 最多创建50个Pod
```

当用户在 `dev-team` 命名空间创建资源（如Deployment）时，API Server 会计算该命名空间内所有已存在资源加上新请求资源的总和。如果总和超过配额，则新请求将被拒绝。通过这种方式，资源配额强制执行了命名空间级别的资源治理策略，是多租户集群环境中的关键管理工具。同时，管理员需配合使用 `LimitRange` 对象为每个Pod设置默认请求/限制值，以确保配额策略的有效性。监控配额使用情况并定期调整，是优化集群资源利用的重要环节。

---

## 18. 描述如何在 Kubernetes 中创建一个 Pod，并给出示例配置文件。

🟡 **中等** | 标签：运维, Kubernetes

在 Kubernetes 中，Pod 是最小的可部署单元，一个 Pod 可包含一个或多个紧密关联的容器。创建 Pod 主要有两种方式：使用命令行（kubectl）或声明式配置文件（YAML）。

**1. 命令行方式（快速创建）**
使用 `kubectl run` 命令可以直接创建一个单容器的 Pod：
```bash
kubectl run my-nginx --image=nginx --port=80
```

**2. 配置文件方式（推荐、可重复）**
通过编写 YAML 文件来定义 Pod，这是生产环境推荐的做法。
**示例配置文件 `pod-example.yaml`：**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
**核心字段说明：**
*   **metadata**: 设置 Pod 名称和标签。
*   **spec.containers**: 定义容器列表。
    *   **image**: 指定容器镜像。
    *   **ports**: 暴露容器端口。
    *   **resources**: 设置 CPU 和内存的资源请求与限制，这对调度和稳定性至关重要。

创建时执行命令：
```bash
kubectl apply -f pod-example.yaml
```
之后可通过 `kubectl get pods` 和 `kubectl describe pod my-nginx` 查看状态和详情。

---

## 19. 解释什么是 Kubernetes，并描述其主要组件及其作用。

🟡 **中等** | 标签：运维, Kubernetes

Kubernetes（K8s）是一个开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。它将集群中的机器抽象为一个统一的资源池，解决了容器在分布式环境中的调度、网络、存储和自愈等核心问题。

其主要组件分为控制平面和工作节点两大部分：

1.  **控制平面（Master）**：集群的大脑，负责决策与管理。
    *   **API Server**：所有操作的统一入口，提供 RESTful 接口。
    *   **Scheduler**：监听未调度的 Pod，根据资源策略将其分配到合适的节点。
    *   **Controller Manager**：运行各种控制器（如节点控制器、副本控制器），确保集群状态与期望状态一致。
    *   **etcd**：高可用的键值存储，保存整个集群的配置和状态数据。

2.  **工作节点（Worker Node）**：集群的四肢，负责运行容器。
    *   **kubelet**：在每个节点上运行的代理，负责管理 Pod 的生命周期，确保容器健康运行。
    *   **kube-proxy**：维护节点上的网络规则，实现服务（Service）的负载均衡和网络访问。
    *   **容器运行时（如 Docker、containerd）**：负责实际运行容器。

简言之，Kubernetes 通过控制平面的智能决策，驱动工作节点高效、可靠地管理容器化应用，是实现云原生基础设施的核心。

---

## 20. 请描述 Kubernetes 中的 Helm 的作用，并解释其使用场景。

🟡 **中等** | 标签：运维, Kubernetes

Helm 是 Kubernetes 的包管理器，类似于 Linux 系统中的 apt 或 yum，其主要作用是简化 Kubernetes 应用的定义、安装和升级过程。

Helm 的核心作用体现在：通过 Chart（一个包含预配置 Kubernetes 资源文件的目录）来封装应用，并利用模板化机制（Go templates）实现配置的灵活定制。它解决了直接管理大量 YAML 文件时遇到的版本控制、依赖管理和发布回滚等复杂问题。

典型使用场景包括：
1. **应用部署标准化**：将微服务、数据库等应用及其依赖资源打包成 Chart，实现一键部署。
2. **多环境配置管理**：使用 values 文件为开发、测试、生产等不同环境提供差异化的配置参数。
3. **复杂应用生命周期管理**：方便地进行应用的升级、回滚和版本控制。
4. **共享与复用**：通过公共仓库（如 Artifact Hub）分发和复用成熟的 Chart，提升部署效率与一致性。

总之，Helm 通过抽象和封装 Kubernetes 资源，显著提升了应用在云原生环境中的管理效率和可靠性。

---

## 21. 请描述 Kubernetes 中的 Namespace 的作用，并解释其使用场景。

🟡 **中等** | 标签：运维, Kubernetes

Kubernetes中的Namespace是一种用于将集群资源划分为多个虚拟集群的逻辑隔离机制。它允许不同团队、项目或应用在同一个物理集群中独立工作，实现资源的逻辑隔离和访问控制。

核心作用包括：一是资源隔离，防止不同环境或团队的资源命名冲突；二是访问控制，通过RBAC策略限制用户对特定Namespace的操作权限；三是资源配额管理，可为每个Namespace设置CPU、内存等资源的使用上限。

典型使用场景包括：1）多团队协作，为每个团队分配独立的Namespace，避免互相干扰；2）多环境部署，通过Namespace区分开发、测试、生产环境；3）项目级资源管理，为特定项目设置专属的资源配额和策略。例如，电商平台可将订单、支付、用户服务分别部署在独立的Namespace中，实现服务解耦和独立扩缩容。

---

## 22. 请解释 Kubernetes 中的 ConfigMap 和 Secret 的作用及其使用场景。

🟡 **中等** | 标签：运维, Kubernetes

**ConfigMap**  
**作用**：用于存储非敏感的配置信息，如应用设置、环境变量或配置文件。  
**使用场景**：将配置数据与容器镜像解耦，支持动态更新（如挂载到Pod或作为环境变量）。适用于数据库连接URL、应用参数等可公开的配置。

**Secret**  
**作用**：存储敏感数据，如密码、证书、API密钥。数据以Base64编码存储（可加密），提供比ConfigMap更高的安全性。  
**使用场景**：传递敏感信息至Pod，例如数据库密码、TLS证书、SSH密钥。需配合RBAC权限控制访问。

**核心区别**：  
1. **数据性质**：ConfigMap存明文配置；Secret存编码/加密的敏感数据。  
2. **安全性**：Secret可通过KMS加密，且有严格权限控制；ConfigMap无加密机制。  
3. **典型用例**：ConfigMap适用于应用配置；Secret用于凭证、密钥等需保密的数据。  

**最佳实践**：避免在Secret中存储超大数据（建议≤1MB），并定期轮换敏感信息。

---

## 23. 请解释 Kubernetes 中的 Service 和 Ingress 的区别，并描述各自的用途。

🟡 **中等** | 标签：运维, Kubernetes

Service 和 Ingress 是 Kubernetes 中处理网络流量的不同层次资源，核心区别在于它们工作的网络层级和主要用途不同。

**Service** 是一种**四层（TCP/UDP）** 的抽象，用于定义一组 Pod 的访问策略。它的核心作用是提供稳定的访问端点（ClusterIP）和负载均衡，使得集群内部或外部能够通过一个固定的虚拟IP访问到后端不断变化的 Pod 集群。它主要解决**服务发现和内部通信**问题。

**Ingress** 是一种**七层（HTTP/HTTPS）** 的 API 资源，作为集群的智能路由入口。它根据用户定义的规则（如主机名或URL路径），将外部流量定向到集群内不同的 Service。它通常与 Ingress 控制器（如 Nginx、Traefik）配合工作，负责处理**反向代理、SSL终止、基于域名的虚拟主机**等高级流量管理功能。

**总结**：Service 负责在**集群内部**或四层网络上暴露和负载均衡一组Pod；Ingress 负责在**集群边缘**进行七层智能路由和流量治理。实际使用中，流量通常从 Ingress 进入，再由 Ingress 转发给对应的 Service，最后由 Service 负责到达具体的 Pod。两者是互补关系，共同构建了 Kubernetes 的网络访问体系。

---

## 24. 描述在 Kubernetes 中如何进行安全配置，并解释常用的方法。

🔴 **困难** | 标签：运维, Kubernetes

Kubernetes安全配置需从架构层面系统性地构建防御体系，核心方法包括：

**1. 认证与授权**  
采用多因素认证机制（如X.509证书、OIDC令牌），通过RBAC实施最小权限原则，精确控制用户与服务账户对资源的访问权限。

**2. 工作负载加固**  
为Pod配置安全上下文，限制容器特权（如禁止特权模式、只读根文件系统）、设置资源限额，并使用Pod安全策略（或准入控制器）强制执行安全基线。

**3. 网络安全**  
通过NetworkPolicy定义Pod间流量隔离规则，使用Calico等CNI插件实现网络分段，结合Service Mesh（如Istio）进行服务间双向TLS加密与流量审计。

**4. 数据与密钥保护**  
静态数据加密（启用etcd加密配置）、传输层TLS全覆盖，使用Sealed Secrets或外部密钥管理系统（如HashiCorp Vault）安全管理凭证。

**5. 持续监控与审计**  
启用Kubernetes审计日志记录关键操作，集成Prometheus与Falco监控异常行为，定期扫描镜像漏洞（如Trivy），并遵循基础设施即代码原则进行版本化配置管理。

**6. 集群运维安全**  
及时更新组件修复漏洞，严格控制集群管理端口暴露，通过堡垒机与VPN访问控制平面，并定期轮换证书与令牌。

---

## 25. 描述在Kubernetes中如何配置持久化存储，并解释其工作原理。

🔴 **困难** | 标签：运维, Kubernetes

Kubernetes中配置持久化存储主要通过PersistentVolume（PV）和PersistentVolumeClaim（PVC）机制实现。核心步骤如下：

1. **准备存储后端**：管理员首先在集群外配置实际存储资源，如NFS、云盘、本地磁阵等。

2. **创建PV**：管理员定义PersistentVolume资源，声明存储容量、访问模式（ReadWriteOnce等）及后端具体信息。这是集群级别的存储资源抽象。

3. **定义PVC**：用户或开发者创建PersistentVolumeClaim，声明所需存储的大小和访问模式，无需关心底层细节。

4. **绑定与使用**：Kubernetes控制平面自动将PVC与满足条件的PV绑定。随后在Pod的volumes字段中引用该PVC，并在容器中通过mountPath挂载，应用即可持久化读写数据。

**工作原理**：  
- **静态供应**：管理员预先创建PV池，系统按需绑定。
- **动态供应**：通过StorageClass自动按PVC需求动态创建PV，实现按需供给。
- 当Pod销毁时，PVC与PV的绑定关系及数据得以保留，确保存储持久化。回收策略（Retain/Delete/Recycle）则控制PVC删除后PV的后续处理。

---

