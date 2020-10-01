### 1 k8s概念和架构

> 特点：
>
> + 轻量级，消耗资源少
> + 开源
> + 弹性伸缩
> + 负载均衡：IPVS

#### 1.1 发展经历

+ Ifrastructure as a Service（基础设施即服务），阿里云
+ Platform as a Service（平台即服务），新浪云
+ Sofaware as a Service（软件即服务），Office软件通过BS访问服务器

#### 1.2 架构

##### 1.2.1 

名词解释：

+ api server：所有服务访问统一入口
+ CotrollerMaager：维持副本期望数量
+ Scheduler：调度器负责接收任务，选择合适的节点分配任务
+ etcd：键值对数据库，存储K8S集群所有重要的持久化信息（恢复K8S集群只需还原etcd即可）
+  Kubelet：与容器引擎交互实现容器的生命周期管理
+ Kube-proxy：负责写入规则至 IPTABLES / IPVS 实现服务映射访问
+ CoreDNS：为集群中的SVC创建一个域名IP的对应关系解析
+ Dashboard：給K8S集群提供一个B/S结构的访问体系
+ Igress Cotroller：官方只能实现4层代理，Igress可以实现7层代理
+ Federatio：提供一个可以跨集群中心多K8S统一管理功能
+ Prometheus：提供一个K8S集群的监控
+ ELK：提供一个K8S集群日志统一分以介入平台Cotroller创建Pod，Service定义访问一组Pod的规则

#### 1.3 网络通讯模式

> K8s网络有三层，从上到下依次是：
>
> + Service网络
> + Pod网络
> + 节点网络

Flanel团队已经实现扁平的网络空间

同一个Pod内多容器之间通讯（共用·pause`的网络栈）：localhost`

各Pod之间的通讯：Overlay Network（覆盖网络） 

同一主机：Docker网桥

不同主机：Flael+ Pod与Service之间的通讯：各节点的Iptables规则 / LVS

#### 1.4 Flanel

> 简介：是CoreOS团队针对Kuberetes设计的一个网络规划服务。**让集群中不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP，并且在这些IP之间建立一个覆盖网络（Overlay Network）**，通过这个覆盖网络，数据包将原封不动的传递到目标容器



### 2 搭建K8s集群

+ 3台CetOS服务器，网络选择NAT

+ k8s-master01

+ apiserver
+ scheduler
+ cotroller-manager
+ etcd
+ k8s-node01
+ k8s-node02
+ kubelet
+ kube-proxy
+  docker



#### 2.1 基于客户端工具kubeadm



##### 2.1.1 CetOS初始化

```shell
# 新建三台虚拟机，系统为CetOS7，master 2核4G，ode 4核8G
# 静态ip，192.168.44.100，101，102
# 禁止swap分区
yum -y istall wget
# 关闭防火墙且关闭开机自启
systemctl stop firewalld && systemctl disable firewalld
# 关闭seliux
sed -i 's/eforcig/disabled' /etc/seliux/cofigt
# 永久
seteforce 0t # 临时
# 关闭swap
swapoff -at
# 临时
sed -ri 's/.*swap.*/#&/' /etc/fstabt # 永久
# 设置主机名
hostamectl set-hostame k8smaster
# 只在master节点添加
hostscat > /etc/hosts << EOF
192.168.44.100 k8s-master
192.168.44.101 k8s-node1
192.168.44.102 k8s-ode2EOF
# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.cof << EOF
et.bridge.bridge-f-call-ip6tables = 1
et.bridge.bridge-f-call-iptables = 1
EOF
sysctl --systemt
# 使配置生效
# 同步时间
yum istall tpdate -ytpdate time.widows.com
```