---
layout: post
title:  "netfilter hook"
author: chenlian
categories: network
tags: network
excerpt: Netfilter is a frameworks provided by the Linux kernel that allows various networking-related operations to be implemented in the form of customized handles. Netfilter is merely a series of hooks in various points in a protocol stack.
---


* content
{:toc}
Netfilter is a framework provided by the Linux kernel that allows various networking-related operations to be implemented in the form of customized handles. Netfilter offers various functions and operations for packet filtering, network address translation, and port translations, which provide the functionality required for directing packets through a network, as well as for providing ability to prohibit packets from reaching sensitive locations within a computer network.


Netfilter represents a set of hooks inside the Linux kernel, allowing specific kernel modules to register callback functions with the kernel's networking stack. Those functions, usually applied to the traffic in the form of filtering and modification rules, are called for every packet that traverses the respective hook within the networking stack.


## Netfilter hook point


```
   <--- LOCAL_IN                 LOCAL_OUT--->
           ^                         |
           |                         |
           |--------> FORWARD------->|
           |                         |
           |                         v
   ---> PRE_ROUTING                POST_ROUTING--->
```


include/uapi/linux/netfilter.h


```c
enum nf_inet_hooks {
	NF_INET_PRE_ROUTING,
	NF_INET_LOCAL_IN,
	NF_INET_FORWARD,
	NF_INET_LOCAL_OUT,
	NF_INET_POST_ROUTING,
	NF_INET_NUMHOOKS
};
```


* NF_INET_PRE_ROUTING ： 数据包进入网络栈，需要进行路由判断。因此，任何数据包进入，都需要经过NF_INET_PRE_ROUTING
* NF_INET_LOCAL_IN : 经过路由判断后，是本机接受，则需要经过NF_INTE_LOCAL_IN
* NF_INET_FORWARD ： 被路由需要本地主机继续转发的数据包，经过NF_INET_FORWARD
* NF_INET_LOCAL_OUT : 本地主机产生的数据包路由后需要发往外部，则需要经过NF_INET_LOCAL_OUT
* NF_INET_POST_ROUTING ：　被路由的转发数据包和本机产生的已路由发往外部，则需要经过NF_INET_POST_ROUTING
* 	NF_INET_NUMHOOKS ： hook 点的数目


支持协议类型 include/uapi/linux/netfilter.h


```c
enum {
	NFPROTO_UNSPEC =  0,
	NFPROTO_INET   =  1,
	NFPROTO_IPV4   =  2,
	NFPROTO_ARP    =  3,
	NFPROTO_NETDEV =  5,
	NFPROTO_BRIDGE =  7,
	NFPROTO_IPV6   = 10,
	NFPROTO_DECNET = 12,
	NFPROTO_NUMPROTO,
};
```


## hook 函数


```c
static inline int nf_hook(u_int8_t pf, unsigned int hook, struct net *net,
			  struct sock *sk, struct sk_buff *skb,
			  struct net_device *indev, struct net_device *outdev,
			  int (*okfn)(struct net *, struct sock *, struct sk_buff *));
```


hook 函数的返回值


```c
/* Responses from hook functions. */
#define NF_DROP 0			/* 丢弃数据包，停止处理 */
#define NF_ACCEPT 1			/* 正常处理 */
#define NF_STOLEN 2			/* 已经处理，停止再处理 */
#define NF_QUEUE 3			/* 排队数据包 */
#define NF_REPEAT 4			/* 重新调用hook函数 */
#define NF_STOP 5			/* 停止执行hook，直接返回 */
#define NF_MAX_VERDICT NF_STOP
```


## netfilter 结构体数组


netfilter 定义了一个nf_hook_ops 结构体来描述一个 hook。


```c
struct nf_hook_ops {
	struct list_head 	list;			/* 链表头 */

	/* User fills in from here down. */
	nf_hookfn		*hook;			/* hook 函数 */
	struct net_device	*dev;			/* 网络设备描述结构体 */
	void			*priv;			/* 私有数据指针 */
	u_int8_t		pf;			/* 协议族类型 */
	unsigned int		hooknum;		/* hook点 */
	/* Hooks are ordered in ascending priority. */
	int			priority;		/* 优先级，数值越小，优先级越高 */
};
```

所有的netfilter 都挂载在一个二维数组上。


```c
/* NFPROTO_NUMPROTO 协议的总数  默认13
   NF_MAX_HOOKS     hook 点的个数   #define NF_MAX_HOOKS 8
*/
struct list_head hooks[NFPROTO_NUMPROTO][NF_MAX_HOOKS];
```


## 注册和注销 hook


```c
/* Function to register/unregister hook points. */
int nf_register_net_hook(struct net *net, const struct nf_hook_ops *ops);
void nf_unregister_net_hook(struct net *net, const struct nf_hook_ops *ops);
int nf_register_net_hooks(struct net *net, const struct nf_hook_ops *reg,
			  unsigned int n);
void nf_unregister_net_hooks(struct net *net, const struct nf_hook_ops *reg,
			     unsigned int n);

int nf_register_hook(struct nf_hook_ops *reg);
void nf_unregister_hook(struct nf_hook_ops *reg);
int nf_register_hooks(struct nf_hook_ops *reg, unsigned int n);
void nf_unregister_hooks(struct nf_hook_ops *reg, unsigned int n);
```

nf_register_hook() 和 nf_unregister_hook() 源码(`base Linux 4.6`)


```c
int nf_register_hook(struct nf_hook_ops *reg)
{
	struct net *net, *last;
	int ret;

	rtnl_lock();
	for_each_net(net) {
		ret = nf_register_net_hook(net, reg);
		if (ret && ret != -ENOENT)
			goto rollback;
	}
	list_add_tail(&reg->list, &nf_hook_list);		//添加 hook 到链表
	rtnl_unlock();

	return 0;
rollback:
	last = net;
	for_each_net(net) {
		if (net == last)
			break;
		nf_unregister_net_hook(net, reg);
	}
	rtnl_unlock();
	return ret;
}
```


```c
void nf_unregister_hook(struct nf_hook_ops *reg)
{
	struct net *net;

	rtnl_lock();
	list_del(&reg->list);				//从链表中删除 hook
	for_each_net(net)
		nf_unregister_net_hook(net, reg);
	rtnl_unlock();
}
```


## netfilter hook 例子