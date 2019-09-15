---
title: Linux bridge设备源码分析
date: 2019-9-14
categories: 
tags: 
	- Network
	- Linux kernel
---

目录：net/bridge

主要包含两个内核模块，bridge和br_netfilter，两个模块的代码入口（init和exit函数）分别在net/bridge/br_netfilter_hooks.c以及net/bridge/br.c里

代码结构：


函数调用关系
bridge内核模块初始化：
- br_init
  - stp_proto_register            注册了br_stp_rcv这个函数（在net/bridge/br_stp_bpdu.c这个文件里），大概是用来
                                  处理BPDU数据包

  - br_fdb_init                   调用kmem_cache_create函数给bridge的fdb分配了一块大小为sizeof(struct       
                                  net_bridge_fdb_entry)的slab缓存，并把返回的kmem_cache指针保存在br_fdb_cache
                                  这个全局变量里

  - register_pernet_subsys        看注释大概是为每个network namespace注册一个br_net_exit函数，这个函数在每个
                                  network namespace删除时会被调用，清除这个namespace中所有的bridge设备

  - br_nf_core_init               调用dst_entries_init，没看懂啥意思

  - register_netdevice_notifier   这个函数的作用是注册一个当网络设备事件发生时会被调用的notifier，
                                  参考https://www.cnblogs.com/pengdonglin137/p/4075148.html，
                                  该函数会将br_device_notifier注册进netdevice notifier chain，而br_device_notifier中包含了一个叫做br_device_event的回调函数，这个回调函数会处
                                  理所有bridge设备以及其从设备的状态变化（Handle changes in state of network devices enslaved to a bridge.）

  - register_switchdev_notifier   bridge这个模块自己搞了个notifier chain，叫做switchdev_notif_chain，
                                  register_switchdev_notifier用来向这个notifier chain注册notifier，这里注册了
                                  br_switchdev_notifier，其中回调函数为br_switchdev_event，它会处理bridge fdb
                                  的add、del和offloaded事件

  - br_netlink_init               初始化bridge模块的netlink接口，以通过netlink套接字进行内核态和用户态的交互（比如
                                  ip link add br0 type bridge），这里调用了rtnl_link_register函数注册了一个
                                  br_link_ops函数表（想通过rtnetlink配置就需要在这个框架中注册一项，然后就可以使用ip link命令配置设备），函数表里面包含了一系列bridge网络设备的一般配置操作（rtnl_link_ops结构体定义了所有设备的配置操作，这里只实现了一部分）

  - brioctl_set                   bridge的ioctl设置



主要研究下br_netlink_init这一部分，因为bridged设备创建、添加&删除slave等等操作都是在这里面被定义的，如之前所述，这里调用了rtnl_link_register函数注册了一个br_link_ops函数表（想通过rtnetlink配置就需要在这个框架中注册一项，然后就可以使用ip link命令配置设备）。函数表里面包含了一系列bridge网络设备的一般配置操作，其中包括了br_dev_setup（用来初始化设置bridge设备）、br_dev_newlink（配置和注册一个新的设备）、br_changelink（修改已存在设备的参数），表中其他函数的功能可以参考include/net/rtnetlink.h中对于rtnl_link_ops结构体的注释，基本上是一些配置读取操作。

br_dev_setup变量的类型为struct net_device_ops，其中定义了网络设备管理的钩子函数（the management hooks for network devices.），具体的每个字段定义见include/linux/netdevice.h中对于net_device_ops结构体的注释；结合注释我们可以找到bridge各方面的具体实现代码入口，这里进一步主要看负责bridge报文传输的 br_dev_xmit 函数以及添加slave的 br_add_slave 函数。


数据传输：
- br_dev_xmit
  - nf_ops->br_dev_xmit_hook             
  - (to be continued)

在br_dev_xmit中nf_ops = rcu_dereference(nf_br_ops)，nf_br_ops是一个被RCU_INIT_POINTER(nf_br_ops, &br_ops)初始化的RCU指针，br_ops其定义在net/bridge/br_netfilter_hooks.c中：

static const struct nf_br_ops br_ops = {
	.br_dev_xmit_hook =	br_nf_dev_xmit,
};

所以直接调用的是br_nf_dev_xmit这个函数，这个函数会检查skb（报文）是否进行了DNAT操作，然后调用br_nf_pre_routing_finish_bridge_slow函数（This is called when br_netfilter has called into iptables/netfilter, and DNAT has taken place on a bridge-forwarded packet.）对进行了DNAT的报文进行处理。。。。




添加slave:
- br_add_slave
  - (to be continued)








br_netfilter内核模块的初始化：
- br_netfilter_init
  - register_pernet_subsys                     为每个network namespace注册一个brnf_exit_net函数，这个函数在每
                                               个network namespace删除时会被调用，清除这个namespace中向netfilter注册的br_nf_ops表中的所有函数

  - register_netdevice_notifier                将brnf_notifier注册进netdevice notifier chain，
                                               brnf_notifier中包含了brnf_device_event回调函数，这个函数只处理
                                               bridge设备注册的事件，然后调用nf_register_net_hooks函数向
                                               bridge的namespace里的netfilter注册br_nf_ops表中的所有hook
                                               function
  
  - register_net_sysctl                        注册sysctl（sys文件系统）相关的内核参数，参数在brnf_table表中定义
                                               包括耳熟能详的bridge-nf-call-arptables、
                                               bridge-nf-call-iptables、bridge-nf-call-ip6tables等等。。

  - RCU_INIT_POINTER(nf_br_ops, &br_ops)       用&br_ops初始化了一个叫做nf_br_ops的RCU protected指针（全局变  
                                               量）

主要研究下br_nf_ops这个数组里的每个hook















内核版本5.1.16

每个bridge的从设备在代码中被称为bridge的一个“port”

net_device标识
IFF_EBRIDGE          Interface is Ethernet bridging device.


/* net device transmit always called with BH disabled */ 中的BH指的是bottom half
