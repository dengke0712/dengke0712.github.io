# Kubernetes基础



# Kubernetes体系架构

## 什么是Kubernetes

Kubernetes（K8s）是一个开源的容器编排平台，旨在简化和自动化容器化应用程序的部署、扩展和管理。它提供了一个强大的工具集，帮助我们能够更轻松地构建和运行容器化应用。

### 核心理念

- **容器化：** Kubernetes利用容器技术（如Docker）将应用及其所有依赖项打包成一个可移植的镜像，确保在任何环境中都能一致运行。

- **自动化：** Kubernetes旨在自动化应用的部署、伸缩、升级和故障恢复等操作，减轻开发人员和运维人员的负担。

- **弹性和可伸缩：** Kubernetes能够根据负载的变化自动调整应用程序的规模，确保系统具有弹性和高可用性。

- **声明式配置：** 用户通过声明式配置文件描述应用程序的期望状态，Kubernetes负责调整实际状态以匹配期望状态。

### 关键特性

- **容器编排：** Kubernetes是一个容器编排平台，负责管理和编排多个容器实例，确保它们协同工作。

- **自动伸缩：** Kubernetes能够根据负载自动调整应用程序的规模，提高资源利用率。

- **服务发现和负载均衡：** Kubernetes提供了内置的服务发现和负载均衡机制，确保应用程序能够可靠地相互通信。

- **持久化存储：** Kubernetes支持持久化存储，使应用程序可以在容器之间共享数据。

- **声明式配置：** 用户通过YAML文件声明应用程序的配置和期望状态，Kubernetes负责实现这些声明。

## Kubernetes集群

### 集群的架构体系

Kubernetes集群的架构体系主要包括Master节点和Worker节点。

- **Master节点：** 是集群的控制中心，负责管理和监控整个集群。主要包括以下核心组件：
   - **API Server：** 提供Kubernetes API，是集群通信的接口。
   - **Controller Manager：** 监视集群资源，确保实际状态与期望状态一致。
   - **Scheduler：** 负责Pod的调度，决定将Pod调度到哪个Worker节点。
   - **etcd：** 分布式键值存储，持久化存储集群的配置数据和状态信息。

- **Node节点：** 是集群中真正运行应用程序容器的节点。主要包括以下核心组件：
   - **Kubelet：** 与Master节点通信，管理Pod的生命周期。
   - **容器运行时：** 负责运行和管理容器。

### Kubernetes的核心组件

Kubernetes的核心组件是Master节点上的一组关键服务，它们共同协作以管理集群中的容器化应用。这些核心组件包括：

- **API Server：** 提供Kubernetes API，是集群通信的接口。
- **Controller Manager：** 监视集群中的资源对象，确保实际状态与期望状态一致。
- **Scheduler：** 负责将新创建的Pod调度到集群中的某个Worker节点上。
- **kubelet：** 管理node节点上运行的pod，并与master节点上的API Server进行交互。

## Kubernetes的对象

### 对象的管理

在Kubernetes中，通过API Server进行对象的管理，用户可以使用配置文件（JSON或YAML格式）来定义和部署对象。

简单的YAML部署示例：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```   

_Notes:_

- 创建了一个名为nginx-deployment的Deployment，由.metadata.name字段指定。这个名称将成为后续创建的ReplicaSets和Pods的基础。

- Deployment创建了一个ReplicaSet，该ReplicaSet创建了三个复制的Pod，由.spec.replicas字段指示。

- .spec.selector字段定义了创建的ReplicaSet如何找到要管理的Pod。在这种情况下，选择在Pod模板中定义的标签（app: nginx）。但是，只要Pod模板本身满足规则，就可以使用更复杂的选择规则。

- template字段包含以下子字段：

    - Pods使用.metadata.labels字段标记为app: nginx。

    - Pod模板的规范，或.template.spec字段，表示Pods运行一个名为nginx的容器，该容器使用Docker Hub上的nginx镜像，版本为1.14.2。

    - 创建一个容器并命名为nginx，使用.spec.template.spec.containers[0].name字段。

### 对象与命名空间
Kubernetes使用命名空间（Namespace）来隔离集群中的不同资源，以避免命名冲突和提供更好的资源管理。在部署完Kubernetes集群后，系统会自动创建一些常见的命名空间，包括：

- default： 默认的命名空间，如果未指定命名空间，则资源将被创建在default命名空间中。
- kube-system： 用于存储Kubernetes系统组件的命名空间，如kube-controller-manager、kube-scheduler等。用户不该使用。
- kube-public： 公开的属性，用于存储集群中所有节点都能访问的资源。
- kube-node-lease： 与所有node节点上的对象相关联，检测node节点状态。


# Kubernetes中的最小可部署对象Pod

Pod代表集群中正在运行的一个进程，是集群中的一个应用实例，由一个或多个容器组成。

- 单容器pod：pod中只运行一个容器，通过pod来管理这个容器。
- 多容器pod：pod中有多个容器一起协同工作，K8s调度器保证所有容器都运行在同一物理主机/虚机上，so容器间可以资源共享。

通过yaml文件可以描述一个pod

```yaml
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中  
kind: Pod #指定创建资源的角色/类型  
metadata: #资源的元数据/属性  
  name: first-pod #资源的名字，在同一个namespace中必须唯一  
  labels: #设定资源的标签
    k8s-app: apache  
    version: v1  
    kubernetes.io/cluster-service: "true"  
  annotations:            #自定义注解列表  
    - name: String        #自定义注解名字  
spec: #specification of the resource content 指定该资源的内容  
  restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器  
  nodeSelector:     #节点选择，先给主机打标签kubectl label nodes kube-node1 zone=node1  
    zone: node1  
  containers:  
  - name: busybox-container #容器的名字  
    image: busybox:apache #容器使用的镜像地址  
    imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略，
                           # Always，每次都检查
                           # Never，每次都不检查（不管本地是否有）
                           # IfNotPresent，如果本地有就不检查，如果没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT  
    args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数  
    env: #指定容器中的环境变量  
    - name: str #变量的名字  
      value: "/etc/run.sh" #变量的值  
    resources: #资源管理
      requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行  
        cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m）
        memory: 32Mi #内存使用量  
      limits: #资源限制  
        cpu: 0.5  
        memory: 32Mi  
    ports:  
    - containerPort: 80 #容器开发对外的端口
      name: httpd  #名称
      protocol: TCP  
    livenessProbe: #pod内容器健康检查的设置
      httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常  
        path: / #URI地址  
        port: 80  
        #host: 127.0.0.1 #主机地址  
        scheme: HTTP  
      initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始  
      timeoutSeconds: 5 #检测的超时时间  
      periodSeconds: 15  #检查间隔时间    
    lifecycle: #生命周期管理  
      postStart: #容器运行之前运行的任务  
        exec:  
          command:  
            - 'sh'  
            - 'yum upgrade -y'  
      preStop: #容器关闭之前运行的任务  
        exec:  
          command: ['service httpd stop']  
    volumeMounts:
    - name: volume #挂载设备的名字，与volumes[*].name 需要对应    
      mountPath: /data #挂载到容器的某个路径下  
      readOnly: True  
  volumes: #定义一组挂载设备  
  - name: volume #定义一个挂载设备的名字  
    #meptyDir: {}  
    hostPath:  
      path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种
```

_Notes:_

1. **labels:** 资源的标签是一对KV，对用户而言非常有意义的，但对K8S本身而言没有直接意义的。Label可以在创建对象时指定，也可以在后期修改，每个对象可以拥有多个标签，但key值必须是唯一的。<u>通过Label进行对象弱关联，灵活地分类和选择不同服务或业务，让用户根据自己特定的组织结构以松耦合方式进行服务部署。</u>
2. **resources 资源管理:** 默认情况下，kubernetes不会限制pod等资源对象使用系统资源，单个pod或者容器可以无限制使用系统资源。kubernetes的资源管理分为资源请求（request）和资源限制（limit），资源请求能够保证Pod有足够的资源来运行，而资源限制则是防止某个Pod无限制地使用资源，导致其他Pod崩溃。
3. **应用程序健康检查:** 分为livenessProbe和readinessProbe，两者相似。livenessProbe在服务运行过程中检查应用程序是否运行正常，不正常将杀掉进程；而readnessProbe是用于检测应用程序启动完成后是否准备好对外提供服务，不正常继续检测，直到返回成功为止。
4. **volumeMounts：** 在K8S中，当Pod重建的时候，数据是会丢失的，K8S也是通过数据卷挂载来提供Pod数据的持久化的。K8S数据卷是对Docker数据卷的扩展，K8S数据卷是Pod级别的，可以用来实现Pod中容器的文件共享。



## pod中的容器

- 基础容器：维护整个pod的网络空间

- 初始化容器：需要在"spec"中添加"initContainers"字段。晚于基础容器运行，早于业务容器运行。一个pod可以有多个初始化容器，可以提前安装在业务容器中要使用的工具，或运行一些初始化脚本。

- 临时容器：在pod中临时运行，以完成用户发起的操作（故障排查、性能诊断），最大用途是调试其他容器。

- 业务容器：实际运行应用的容器。

# 通过 Service 访问 Pod

Service为应用提供统一的访问地址。它提供了一个稳定的网络端点，以便其他应用或服务可以通过该端点与 Pod 进行通信。

对集群内部，它不断跟踪pod的变化，更新endpoint中对应pod的对象，提供了ip不断变化的pod的服务发现机制；

对集群外部，他类似负载均衡器，可以在集群外部对pod进行访问。在Kubernetes中，Pod的IP地址和service的ClusterIP仅可以在集群网络内部做用，对于集群外的应用是不可见的。


## 定义service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

这里定义了一个service，它通过标签选择器 app: my-app 与具有相同标签的 Pod 关联。Service 在端口 80 上监听请求，并将流量转发到 Pod 的端口 8080。

检查service状态:
```bash
kubectl get services
```

## Service的类型

ClusterIP
默认类型，将 Service 暴露在集群内部，集群内部其他 Pod 可以通过 ClusterIP 访问。

NodePort
在每个节点上开放一个静态端口，使得外部可以通过节点的 IP 和该端口访问 Service。

**缺点：** 因为每个端口只能是一种服务，端口范围只能是 30000-32767。当有几十上百的服务在集群中运行时，NodePort的端口管理就是个灾难。

LoadBalancer
通过设置LoadBalancer映射到云服务商提供的LoadBalancer地址。这种用法仅用于在公有云服务提供商的云平台上设置Servic的场景。受限于云平台，且通常在云平台部署LoadBalancer还需要额外的费用。

ExternalName
将 Service 映射到外部服务的 DNS 名称。

### 集群外部请求访问集群内应用的最佳方式 — Ingress

只需一个或者少量的公网IP和LB，即可同时将多个HTTP服务暴露到外网，七层反向代理。可以简单理解为service的service，它其实就是一组基于域名和URL路径，把用户的请求转发到一个或多个service的规则。


## Service的使用

a. 集群内部访问pod

```bash
kubectl exec -it <pod-name> -- /bin/sh
curl <service-name>
```

b. 外部访问（NodePort）

在浏览器中访问节点的 IP 地址和指定的 NodePort。


# 常用命令
第三部分：常用指令

1. 获取Pod信息

```bash
kubectl -n ccos-ops-app get pod
kubectl -n ccos-ops-app get pod | grep jobm
kubectl -n ccos-ops-app get pod -owide | grep jobm

获取特定部署的 Pod 信息
kubectl -n ccos-ops-app get deploy jobm
```

2. 在容器内执行特定命令
```bash
kubectl exec -n ccos-ops-app -it jobm-5b6fdbd5d7-55dqs -- /bin/sh
kubectl exec cmdb-fafsf-kfasf -n ccos-ops-region -it -- bash
-> curl
```

3. 删除 Pod
```bash
kubectl delete pod <pod-name> -n <namespace>
kubectl delete pod <pod-name> -n <namespace> --force
```

4. 查看日志
```bash
kubectl -n ccos-ops-app logs -f <pod-name> --tail=300
```

5. 编辑pod
```bash
kubectl edit pod <pod-name> -n <namespace>
```

6. 修改服务

```bash
kubectl -n ccos-mysql edit svc ccos-mysql
type 从 ClusterIP 改成 NodePort
kubectl get svc -n ccos-mysql
```

```bash
kubectl expose pod -n ccos-clickhouse ck-logging-0-0-0
kubectl -n ccos-clickhouse edit svc ck-logging-0-0-0
kubectl -n ccos-clickhouse get svc
```

7. 操作Cronjob
```bash
kubectl apply -f 0000_90_jobm_041_cronjob.yaml
kubectl -n ccos-ops-app get cronjob
kubectl -n ccos-ops-app edit cronjob
kubectl delete cronjob cronjob-name
kubectl get job
```

8. 登录到节点进行 sudo 操作：
```bash
ssh -i id_rsa ccadmin@
```



# 结语

通过阅读《Docker+Kubernetes容器实战派》这本书，让我深入了解了这个强大的容器编排系统，对于容器化应用的部署和管理有了更深层次的理解。
通过每一章节的学习，我逐渐掌握了 Kubernetes 的基本概念，了解了它的体系架构、以及各种对象的管理方式。学会了如何部署 Kubernetes 集群，以及如何通过 Service 访问 Pod，同时也涉及了持久化存储等重要主题。
当然，这只是一个开始，我意识到还有很多高级概念和特性等待我深入研究，比如安全认证、自动化部署、集成与运维管理等，未来我还会继续追求更深入的知识。

