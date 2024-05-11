---
title: 笔记-K8s
categories: 好的学习笔记
tags: K8s
---


# K8s-基本指令

### 前提

**基础知识**：

* configmap==cm，deployment==deploy，service==vc，namespace==ns，pod==po

- kubectl get：查询资源

  - -n：n 代表 namespace，正常来说每条 pod 指令都要带上
  - -o：o 代表 optional，通用的就是 wide 和 yaml，wide 用于列表，yaml 用于详情

- kubectl logs：打印 pod 的日志

- kubectl descirbe：查看资源信息

  - kubectl describe node
  - kubectl describe pod
  - kubectl describe service

- kubectl exec：在 pod 中执行命令

___

### 查询资源列表

> #Get：
>
> kubectl get namespaces -o wide
>
> kubectl get nodes -o wide
>
> kubectl get services -o wide -n test-sit
>
> kubectl get pods -o wide -n test-sit
>
> kubectl get deployments -o wide -n test-sit 	//查看已经部署了的所有应用，可以看到容器，以及容器所用的镜像，标签等信息
>
> kubectl get replicaset -o wide -n test-sit	//查看目前所有的replicaset，显示了所有的pod的副本数，以及他们的可用数量以及状态等信息
>
> kubectl get configmap -o wide -n test-sit	// 查看配置



### 查询资源详情信息

> #Get：
>
> kubectl get namespace/node ‘具体资源’ -o yaml
>
> kubectl get service/pod/deployments/replicaset/configure '具体资源' -o yaml -n test-sit
>
> 
>
> #Describe：
>
> kubectl describe namespace/node ‘具体资源’ 
>
> kubectl descirbe service/pod/deployments/replicaset/configure '具体资源'  -n test-sit
>
> 
>
> #Log
>
> kubectl logs -f "具体资源"



### 编辑和应用资源

> #Edit
>
> kubectl edit -f
>
> 
>
> #Apply
>
> kubectl apply -f



可以使用【kubectl edit】直接修改，但是更好的是找到资源文件【.yaml】进行【kubectl apply】。edit可以当临时使用。

![image-20230309141510988](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230309141510988.png)



![image-20230309141639921](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230309141639921.png)



### 资源的拷贝

> #CP
>
> // CP容器内->容器外
>
> kubectl cp  pod名:文件绝对路径 文件目标位置
>
> 
>
> // 容器外->容器内
>
> kubectl cp 路径 pod名:容器内绝对路径



# K8s-资源和系统架构

### 前提

> [Kubernetes高级使用功能](https://mp.weixin.qq.com/s/0NwX0wRmg1Sayve2k_w9RQ?vid=1688855323406858&deviceid=e49ee5d7-811c-463e-a2d5-92350cd767ad&version=4.1.0.6011&platform=win)

### Kubernetes 资源架构

下图中的都是可以创建和查询到的资源：

![image-20230302173541519](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302173541519.png)

### Kubernetes 创建资源工作流程

下图是创建资源的工作流程：

![image-20230302173448745](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302173448745.png)



1. 客户端发出创建 Deployment 请求
2. API Server 接收到客户端的调用
3. API Server 通知 Controller Manager 创建一个 deployment 资源。
4. Scheduler 执行调度任务，将两个副本 Pod 分发到 node1 和 node2。
5. node1 和 node2 上的 kubelet 在各自的节点上创建并运行 Pod。



# K8s-Pod是什么？

### Pod

Pod 是一个或一个以上的容器（例如Docker容器）组成的，且具有共享存储/网络/UTS/PID的能力，以及运行容器的规范。并且在kubernetes 中，Pod 是最小的可被调度的原子单位。

按照容器的设计理念，每个容器只运行单个进程。而要想实现多个 container 被绑定在一起进行管理的需求。我们需要一种高级别的概念来实现这个。在 kubernetes 中，这就是 Pod。 在 Pod 里面，container 之间可以共享网络（IP/Port）、共享存储（`Volume`）、共享Hostname。



### 总结

pod 管理多个container，service 是运行再 container 中的。之前单独使用 Docker 的时候，各个服务运行在 Dokcer 环境中，服务之间也是可以共享网络、共享存储的。kubernetes  引入 pod 来实现。

但是注意，Kubernetes 通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。



# K8s-Service 是什么？

### Service

Kubernetes Service 定义了外界访问一组特定 Pod 的方式。Service 有自己的 IP 和端口，为 Pod 的访问提供了负载均衡。

拥有固定 IP 的服务，从逻辑上代表了一组 Pod。Pod 的 IP 地址会变化，但是 Service 的 IP 是不变的。客户端只需要访问 Service 的 IP。



![image-20230302115807716](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302115807716.png)



### Service 使用流程分析：

![image-20230302115940457](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302115940457.png)

1. 管理员或用户创建 Service。
2. Kube-proxy 监听到 Service 变化，创建相应的 iptables 规则，同步到所有 Node 节点上。
3. 用户访问 Service 的 ip 和端口号，Service 将流量转向 Node 节点的 Pod 的端口。
4. Node 节点根据 iptables 规则，转发流量到 Pod 的端口上。
5. 流量转到 Pod 里的 docker 中



# K8s-Controller 是什么？

### 前提

> [Kubernetes高级使用功能](https://mp.weixin.qq.com/s/0NwX0wRmg1Sayve2k_w9RQ?vid=1688855323406858&deviceid=e49ee5d7-811c-463e-a2d5-92350cd767ad&version=4.1.0.6011&platform=win)

![image-20230302120011729](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302120011729.png)



### Controller

Kubernetes 通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。负责管理和协调 Pod 的创建、复制、伸缩、更新和删除等操作。

Controller 中定义了 Pod 的部署特性，比如有几个副本，在什么样的 Node 上运行等。为了满足不同的业务场景，Kubernetes 提供了多种 Controller，包括 Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job 等。

1. Deployment 是最常用的 Controller，Deployment 可以管理 Pod 的多个副本，并确保 Pod 按照期望的状态运行。
2. ReplicaSet 实现了 Pod 的多副本管理。使用 Deployment 时会自动创建 ReplicaSet，也就是说 Deployment 是通过 ReplicaSet 来管理 Pod 的多个副本，通常不需要直接使用 ReplicaSet。
3. DaemonSet 用于每个 Node 最多只运行一个 Pod 副本的场景。DaemonSet 通常用于运行守护进程的服务。

4. 你可能会注意到Replica Set的名字总是`<Deployment的名字>-<pod template的hash值>`

![image-20230228113548270](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230228113548270.png)



如下图所示：

![image-20230530155517792](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230530155517792.png)

![image-20230530155434143](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230530155434143.png)





# K8s-ConfigMap 是什么？

### ConfigMap

使用 Docker 容器时，我们通过挂载包含该文件的卷进行配置管理和修改。而在 k8s 中，我们要讲一种更好的方式，即 ConfigMap，这种资源对象的出现，更是极大的方便了应用程序的配置管理。

**用法：**

* 生成容器内的环境变量，在 pod 中可以通过 `spec.env` 或者 `spec.envFrom` 进行引用。
* 设置容器启动命令的启动参数，前提是设置为环境变量。
* 以卷 volume 的方式挂载到容器内部的文件或目录，通过 `spec.volumes` 引用。



![image-20230228160016996](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230228160016996.png)



![image-20230228160232481](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230228160232481.png)



![image-20230228155759374](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230228155759374.png)





# K8s-Ingress 是什么？

### 前提

[Kubernetes高级使用功能](https://mp.weixin.qq.com/s/0NwX0wRmg1Sayve2k_w9RQ?vid=1688855323406858&deviceid=e49ee5d7-811c-463e-a2d5-92350cd767ad&version=4.1.0.6011&platform=win)



**暴露service的三种方式：**

* ClusterIP 的方式只能在集群内部访问。
* NodePort 方式的话，测试环境使用还行，当有几十上百的服务在集群中运行时，NodePort 的端口管理是灾难。
* LoadBalance 方式受限于云平台，且通常在云平台部署 ELB 还需要额外的费用。

> 单独用 service 暴露服务的方式， 在实际生产环境中不太合适



**K8s-service 的作用：**

service的作用体现在两个方面，对集群内部，它不断跟踪 pod 的变化，更新 endpoint 中对应 pod 的对象，提供了 ip 不断变化的 pod 的服务发现机制，对集群外部，他类似负载均衡器，可以在集群内外部对pod进行访问。

service 的功能是将 TCP/IP 的四层流量转发到后端 pod，service 对于 HTTP 服务来说，无法完成不同 URL 对应不同后端服务的的功能。所以如果想要完成根据 URL 转发到不同后端，可以使用 Ingress 完成。



### Ingress

> Ingress 的工作就是根据配置文件，修改 nginx.conf, 让 Nginx 灵活指向不同 Service。

它是k8s提供的一种集群维度暴露服务的方式，简单理解为service的service，他通过独立的ingress对象来制定请求转发的规则，把请求路由到一个或多个service中。这样就把服务与请求规则解耦了，可以从业务维度统一考虑业务的暴露，而不用为每个service单独考虑。



Ingress 一般有三个组件组成：

1. Ingress 是 Kubernetes 的一个资源对象，用于编写定义规则。
2. 反向代理负载均衡器，通常以 Service 的 Port 方式运行，接收并按照 Ingress 定义的规则进行转发，通常为 nginx，haproxy，traefik 等。
3. Ingress-Controller，监听 apiserver，获取服务新增，删除等变化，并结合 Ingress 规则动态更新到反向代理负载均衡器上，并重载配置使其生效。



### Ingress 工作原理图

![image-20230307143103578](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230307143103578.png)





# K8s-K8s 和 Srping Cloud

### 前提

> [拥抱Kubernetes,再见了Spring Cloud](https://zhuanlan.zhihu.com/p/339736610)
>
> [从Spring Cloud到Kubernetes的微服务迁移实践](https://zhuanlan.zhihu.com/p/128571899)



### k8s 和 Spring Cloud 的激烈冲突

> k8s 的捆绑销售和springcloud的暧昧解决（spring-cloud-starter-kubernete） --------------------------> 开始互相看不顺眼了

因为从扩展部署、运维角度出发的 k8s，在最原始容器、应用程序部署及网络层管理的基础上，已逐步实现并贴近应用层的需要，一些微服务架构下的基础需求（如：Service Discovery、API Gateway 等）开始直接或间接被纳入 k8s 生态。 导致双方有很多组件功能重复，且只能择一而终， 一旦你选了 Spring Cloud 的解決方案，就得放弃 k8s 那边的机制。

![image-20230302140538500](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302140538500.png)



### 服务发现 DiscoveryClient

Spring Cloud 的经典解决方案：Netflix Eureka、Alibaba Nacos、Hashicorp。

现在它为K8s提供了 DiscoveryClient 组件，让基于 Spring Cloud 所开发的程序可方便查询其他服务。 使用了 Kubernetes 原生的服务发现，才能被 Istio 追踪，为来才能纳入 Service Mesh 的管控。



![image-20230302140208940](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302140208940.png)



### 配置管理

Spring Cloud 的解决方案：spring-cloud-config。但在 Kubernetes 上，有 ConfigMap 和 Secret 可使用，而且通常还会搭配 Vault 管理敏感配置。



![image-20230302140442931](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230302140442931.png)





# K8s-实践

流程图如下：

![image-20230831160702629](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230831160702629.png)



### 一、Nginx 服务器配置

在 Nginx 服务器上的配置，如下所示。监听地址 www.gxcospower.cn 的 443 端口的请求。然后代理到地址 http://local_upstream。它是 k8s 集群的两个 node 节点。



![image-20230831153937033](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230831153937033.png)



![image-20230831154205900](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230831154205900.png)



### 二、Ingress 配置

Ingress 的配置，如下所示。

~~~ bash
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: www-kct-ingress
  namespace: test-sit
  selfLink: /apis/extensions/v1beta1/namespaces/test-sit/ingresses/www-kct-ingress
  uid: 8946db35-bcb8-4829-b421-4bd96d827c47
  resourceVersion: '55390491'
  generation: 12
  creationTimestamp: '2023-02-09T07:30:25Z'
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: >
      {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{},"name":"www-kct-ingress","namespace":"test-sit"},"spec":{"rules":[{"host":"www.gxcospower.cn","http":{"paths":[{"backend":{"serviceName":"kct-cos-power-repair","servicePort":8081},"path":"/v1/repair/"},{"backend":{"serviceName":"kct-cos-power-repair-web-admin-svc","servicePort":80},"path":"/repair/"},{"backend":{"serviceName":"cos-power-customer-service-admin","servicePort":8081},"path":"/v1/customer/"},{"backend":{"serviceName":"cos-power-customer-service-admin","servicePort":8081},"path":"/v1_admin/customer/"},{"backend":{"serviceName":"cos-power-customer-service-admin","servicePort":8081},"path":"/app_v1/customer/"},{"backend":{"serviceName":"cos-power-customer-service-web-admin-svc","servicePort":80},"path":"/customer/"},{"backend":{"serviceName":"cos-power-mobile-h5-svc","servicePort":80},"path":"/mobile/"},{"backend":{"serviceName":"cos-power-customer-service-admin","servicePort":9001},"path":"/v1/customer/jmx"}]}}]}}
  managedFields:
    - manager: kubectl
      operation: Update
      apiVersion: extensions/v1beta1
      time: '2023-03-08T10:32:56Z'
      fieldsType: FieldsV1
      fieldsV1:
        'f:metadata':
          'f:annotations':
            .: {}
            'f:kubectl.kubernetes.io/last-applied-configuration': {}
        'f:spec':
          'f:rules': {}
spec:
  rules:
    - host: www.gxcospower.cn
      http:
        paths:
          - path: /v1/repair/
            pathType: ImplementationSpecific
            backend:
              serviceName: kct-cos-power-repair
              servicePort: 8081
          - path: /repair/
            pathType: ImplementationSpecific
            backend:
              serviceName: kct-cos-power-repair-web-admin-svc
              servicePort: 80
          - path: /v1/customer/
            pathType: ImplementationSpecific
            backend:
              serviceName: cos-power-customer-service-admin
              servicePort: 8081
          - path: /v1_admin/customer/
            pathType: ImplementationSpecific
            backend:
              serviceName: cos-power-customer-service-admin
              servicePort: 8081
          - path: /app_v1/customer/
            pathType: ImplementationSpecific
            backend:
              serviceName: cos-power-customer-service-admin
              servicePort: 8081
          - path: /customer/
            pathType: ImplementationSpecific
            backend:
              serviceName: cos-power-customer-service-web-admin-svc
              servicePort: 80
          - path: /mobile/
            pathType: ImplementationSpecific
            backend:
              serviceName: cos-power-mobile-h5-svc
              servicePort: 80
          - path: /v1/customer/jmx
            pathType: ImplementationSpecific
            backend:
              serviceName: cos-power-customer-service-admin
              servicePort: 9001
status:
  loadBalancer: {}

~~~



转换为 Nginx，如下所示。

意思就是，监听 www.gxcospower.cn 的 80 端口的请求，然后代理到 k8s 的服务。



【注意】Ingress 默认的监听端口是 80 端口。



~~~bash
server {
    listen 80;
    server_name www.gxcospower.cn;

    location /v1/repair/ {
        proxy_pass http://kct-cos-power-repair:8081;
    }

    location /repair/ {
        proxy_pass http://kct-cos-power-repair-web-admin-svc:80;
    }

    location /v1/customer/ {
        proxy_pass http://cos-power-customer-service-admin:8081;
    }

    location /v1_admin/customer/ {
        proxy_pass http://cos-power-customer-service-admin:8081;
    }

    location /app_v1/customer/ {
        proxy_pass http://cos-power-customer-service-admin:8081;
    }

    location /customer/ {
        proxy_pass http://cos-power-customer-service-web-admin-svc:80;
    }

    location /mobile/ {
        proxy_pass http://cos-power-mobile-h5-svc:80;
    }

    location /v1/customer/jmx {
        proxy_pass http://cos-power-customer-service-admin:9001;
    }
}

~~~



【问题】这里的监听地址不应该是本机的 IP 地址吗？

* 此时，请求已经经过了一次代理。如：www.gxcospower.cn:443/v1/repair 已经变成了 http://10.12.154.2/v1/repair 或 http://10.12.154.3/v1/repair 。
* 需要在本机的 host 文件上加上，将 www.gxcospower.cn 指向本机 IP。



如下所示。

![image-20230831155207513](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230831155207513.png)