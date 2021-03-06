---
title: "openstack官方文档中的一些多余配置"
author: opengers
layout: post
permalink: /openstack/read-the-openstack-docs-with-a-critical-view/
categories: openstack
tags:
  - openstack
  - documentation
format: quote
---

><small>[OpenStack文档](https://docs.openstack.org/)肯定是我们学习OpenStack最好的资源之一，然而其中也难免有一些错误或多余的配置项</small>    

OpenStack官方文档内容很详细，新学习的话一般是从安装文档开始，安装文档包含每个组件的介绍和详细配置，刚开始不理解配置作用，可以直接copy文档中的配置，但是其中一些配置也不总是正确和有用的，况且openstack文档基本每天都在更新中。     

比如[Newton版centos7安装文档](https://docs.openstack.org/newton/install-guide-rdo/)，在Compute service中关于计算节点(compute node)的配置步骤[Install and configure a compute node](https://docs.openstack.org/newton/install-guide-rdo/nova-compute-install.html#install-and-configure-components)中有这几步配置       

![openstack-docs-1](/images/openstack/openstack-docs/openstack-docs-1.png)    

**Nova的计算节点上需要配置`keystone_authtoken`吗?，先来看下Nova的简单介绍**         

Compute service项目代号为Nova，是OpenStack中的计算服务，负责虚拟机整个生命周期管理。Nova中包含多个组件，这些组件会部署在两类节点上: 计算节点(compute node)和控制节点(controller node)。看一下Nova架构图                

![openstack-docs-2](/images/openstack/openstack-docs/openstack-docs-2.png)     

Nova架构比较复杂，包含很多组件。这些组件以子服务的形式运行。其中`nova-compute`是管理虚机的核心服务，通过调用 Hypervisor API实现虚机生命周期管理。这就意味着每个计算节点上必须有`nova-compute`运行。事实上，计算节点上只有`nova-compute`，其它组件是运行在控制节点上的(官方文档上是这个部署架构，当然由于openstack的灵活性，你可以随意改变某个组件的部署节点)       

另外，你可以找一台计算节点执行下`ps -ef |grep nova-`，看看是不是只有nova-compute存在          

从上面Nova架构图可以看到，`nova-compute`只与Hypervisor和Queue(MQ)交互    

- 与Hypervisor是通过默认配置项`compute_driver=libvirt.LibvirtDriver`指定当前计算节点上使用的Hypervisor类型      
- 与MQ通信是配置项`transport_url = rabbit://user:pass@mq_host_ip`，`nova-compute`从MQ中接受虚拟机创建请求    

上面说了这么多，在Nova部署中，若计算节点只运行`nova-compute`(大多数部署方案)，`/etc/nova/nova.conf`中根本就不需要配置keystone_authtoken，与keystone交互的是`nova-api`组件      

同样的，在网络服务Neutron中也有类似情况，比如       

- 只有在`neutron-server`运行的节点上才需要配置`/etc/neutron/plugins/ml2/ml2_conf.ini`，一般是控制节点。  
- 在Neutron计算节点上，不管使用哪种bridge agent，`neutron-linuxbridge-agent`或`neutron-openvswitch-agent`，他们是与MQ交互，`/etc/neutron/neutron.conf`中不要再配置keystone认证部分    

可以这样简单理解，需要配置keystone认证的是各服务的api组件，因此`nova-api`,`glance-api`,`neutron-server`所在的节点上是需要配置keystone认证的     
(本文完)






