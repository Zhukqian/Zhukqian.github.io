---
layout:     post
title:      "EHcache+Jgroups实现EHcache集群,缓存同步"
subtitle:   work
date:       2018-5-14
author:     朱坤乾
header-img: ![](https://imgchr.com/i/kQTFtH)
catalog: true
tags:
    - JAVA工作
---

##  使用JGroups TCP实现EHCache的集群
因为负载需要，使用两台服务器做负载均衡,所以需要对缓存进行集群处理

###  EhCache 从 1.7 版本开始，支持五种集群方案，分别是：

>Terracotta

>RMI

>JMS

>JGroups

>EhCache Server

本文主要是为了讲解Jgroups

JGroups 提供了一个非常灵活的协议栈、可靠的单播和多播消息传输，主要的缺点是配置复杂以及一些协议栈对第三方包的依赖。

JGroups 也提供了基于 TCP 的单播 ( Unicast ) 和基于 UDP 的多播 ( Multicast ) ，对应 RMI 的手工配置和自动发现。使用单播方式需要指定其它节点的主机地址和端口，下面是两个节点.

###  1:echache配置文件ehcache.xml
```
<cacheManagerPeerProviderFactory
	class="net.sf.ehcache.distribution.jgroups.JGroupsCacheManagerPeerProviderFactory"
	properties="connect=TCP(bind_addr=127.0.0.1;bind_port=1000):
						TCPPING(initial_hosts=127.0.0.1[1000],127.0.0.1[2000];port_range=1;timeout=5000;num_initial_members=2):
						MERGE2(min_interval=3000;max_interval=5000):
						FD_ALL(interval=5000;timeout=20000):
						FD(timeout=5000;max_tries=48;):
						VERIFY_SUSPECT(timeout=1500):
						pbcast.NAKACK(retransmit_timeout=100,200,300,600,1200,2400,4800;discard_delivered_msgs=true):
						pbcast.STABLE(stability_delay=1000;desired_avg_gossip=20000;max_bytes=0):
						pbcast.GMS(print_local_addr=true;join_timeout=5000)"
propertySeparator="::" />

```

就不一一详解,自行百度,或者官网查询

使用多播方式配置如下： 

```
<cacheManagerPeerProviderFactory
        class="net.sf.ehcache.distribution.jgroups.JGroupsCacheManagerPeerProviderFactory"
        properties="connect=UDP(mcast_addr=224.1.1.1;mcast_port=45678;ip_ttl=32;mcast_send_buf_size=120000;mcast_recv_buf_size=80000): 
        PING(timeout=2000;num_initial_members=2): 
        MERGE2(min_interval=5000;max_interval=10000): 
        FD_SOCK:VERIFY_SUSPECT(timeout=1500): 
        pbcast.NAKACK(retransmit_timeout=3000): 
        UNICAST(timeout=5000): 
        pbcast.STABLE(desired_avg_gossip=20000): 
        FRAG: 
        pbcast.GMS(join_timeout=5000;print_local_addr=true)"
        propertySeparator="::" />

```
从上面的配置来看，JGroups 的配置要比 RMI 复杂得多，但也提供更多的微调参数，有助于提升缓存数据复制的性能。
详细的 JGroups 配置参数的具体意义可参考 JGroups 的配置手册。
####  使用组播方式的注意事项：
>使用 JGroups 需要引入 JGroups 的 Jar 包以及 EhCache 对 JGroups 的封装包 ehcache-jgroupsreplication-xxx.jar 。

>在一些启用了 IPv6 的电脑中，经常启动的时候报如下错误信息：
java.lang.RuntimeException: the type of the stack (IPv6) and the user supplied addresses (IPv4) don't match: /231.12.21.132.
如果是 Tomcat 服务器，可在 catalina.bat 或者 catalina.sh 中增加如下环境变量即可：

SET CATALINA_OPTS=-Djava.net.preferIPv4Stack=true

###  2:为每一个需要同步的cache配置listener(监听),bootstrapCacheLoaderFactory(启动,当然也可以不要)

```
<cache name="mybatis_common" overflowToDisk="true" eternal="true"  
        timeToIdleSeconds="300" timeToLiveSeconds="600" maxElementsInMemory="10000"  
        maxElementsOnDisk="100" diskPersistent="true" diskExpiryThreadIntervalSeconds="300"  
        diskSpoolBufferSizeMB="50" memoryStoreEvictionPolicy="LRU">
          <cacheEventListenerFactory class="net.sf.ehcache.distribution.jgroups.JGroupsCacheReplicatorFactory"
              properties="replicateAsynchronously=true, replicatePuts=true,
              replicateUpdates=true, replicateUpdatesViaCopy=false, replicateRemovals=true",
              asynchronousReplicationIntervalMillis=500/>
          <bootstrapCacheLoaderFactory class="net.sf.ehcache.distribution.jgroups.JGroupsBootstrapCacheLoaderFactory" 
              properties="bootstrapAsynchronously=false"/>
</cache>

```
###   简单讲解
>replicatePuts=true | false – 当一个新元素增加到缓存中的时候是否要复制到其他的peers。默认是true。

>replicateUpdates=true | false – 当一个已经在缓存中存在的元素被覆盖时是否要进行复制。默认是true。

> replicateRemovals=true | false – 当元素移除的时候是否进行复制。默认是true。

>replicateAsynchronously=true | false – 复制方式是异步的指定为true时，还是同步的，指定为false时。默认是true。

>replicateUpdatesViaCopy=true | false – 当一个元素被拷贝到其他的cache中时是否进行复制指定为true时为复制，默认是true。



##  总结
一般情况下，这样就足够了，但是事有例外,自己在寻找一些解决问题的方法.

Good Luck。