---
title: Ganglia部署实战
date: 2016-07-12 18:00:00
updated: 2016-07-12 18:00:00
tags:
categories: 运维测试
---
Ganglia是UC Berkeley发起的一个开源集群监视项目，设计用于测量数以千计的节点。Ganglia的核心包含gmond、gmetad以及一个Web前端。主要是用来监控系统性能，如：cpu、mem、硬盘利用率、I/O负载、网络流量情况等，通过曲线很容易见到每个节点的工作状态，对合理调整、分配系统资源，提高系统整体性能起到重要作用。

# 工作原理

Ganglia主要有两个角色：gmond（ganglia monitor daemons）和gmetad（ganglia metadata daemons）。

gmond：需要在被监控的每台机器上部署，负责采集所在机器的系统状态，信息都是存储在内存里面的。

gmetad：负责收集gmond监控信息写入rrdtool环形数据库，然后ganglia-web读取rrdtool并且绘图呈现给用户。

# 部署环境

> ganglia v3.7.2

> CentOS release 6.3 (Final)

> Linux version 2.6.32-279.el6.x86_64

# 1.安装Ganglia

## 1.1 安装Ganglia服务端

	yum install ganglia ganglia-devel ganglia-gmetad ganglia-gmond ganglia-gmond-python ganglia-web

## 1.2 安装Ganglia客户端

	yum install ganglia ganglia-gmond

# 2.配置Ganglia

## 2.1 服务端配置gmetad ( /etc/ganglia/gmetad.conf )

	gridname "网格名称"

	data_source "集群名称" IP1：PORT1 IP2：PORT2
	
	# 可选配置
	# The port gmetad will answer requests for XML.
	xml_port 8651
	# 可选配置
	# The port gmetad will answer queries for XML.
	interactive_port 8652
	# 可选配置（响应请求线程）
	server_threads 4
	# 可选配置（数据保存路径）
	rrd_rootdir "/var/lib/ganglia/rrds"

注：PORT默认端口8649

## 2.2 客户端配置gmond ( /etc/ganglia/gmond.conf )

    globals {
      # 是否后台方式运行
      # When yes, gmond will daemonize. 
      # When no, gmond will run in the foreground.
      daemonize = yes
      # 是否指定UID
      # When yes, gmond will set its effective UID to the uid of the user specified by the user attribute. 
      # When no, gmond will not change its effective user.
      setuid = yes
      # 运行gmond用户
      user = ganglia
      # 调试级别
      # When set to zero (0), gmond will run normally.
      # The higher the debug_level the more verbose the output.
      # A debug_level greater than zero will result in gmond running in the foreground and outputting debugging information. 
      debug_level = 0
      # 发送包最大长度
      max_udp_msg_len = 1472
      # 是否广播采集数据
      mute = no
      # 是否接收广播数据
      deaf = no
      # 是否发送EXTRA_ELEMENT和EXTRA_DATA部分
      allow_extra_data = yes
      # （秒）
      host_dmax = 86400
      # （秒）
      host_tmax = 20
      # 清除过期数据最小时间间隔（秒）
      cleanup_threshold = 300
      # 是否启用gexec任务
      # http://www.theether.org/gexec/
      gexec = no
      # 可选配置（重写主机名称）
      # override_hostname = "主机名称"
      # 发送采集数据最小时间间隔（秒）
      # If you are not using multicast this value should be set to something other than 0.
      send_metadata_interval = 0
    }
    
    cluster {
      # 同data_source集群名称配置
      name = "集群名称"
      # 示例：Wythe 或 unspecified
      owner = "节点所有者"
      # 示例：N37.37 W122.23 或 unspecified
      latlong = "经纬度坐标"
      # 示例：https://babysource.github.io/ 或 unspecified
      url = "集群信息网址"
    }
    
    udp_send_channel {
      # 可选配置（是否绑定主机名称）
      # bind_hostname = yes
      
      # 多播地址（多播时配置，多播与单播模式互斥，仅限选一种模式）
      mcast_join = 239.2.11.71
      # 可选配置（多播时配置，绑定网络设备）
      # mcast_if = eth0
      
      # 单播地址（单播时配置，单播与多播模式互斥，仅限选一种模式）
      # 单播模式下可配置多个udp_send_channel
      # host = 127.0.0.1
      
      # 监听端口
      port = 8649
      # Time to Live
      ttl = 1
    }
    
    udp_recv_channel {
      # 多播地址（多播时配置，多播与单播模式互斥，仅限选一种模式）
      mcast_join = 239.2.11.71
      # 可选配置（多播时配置，绑定网络设备）
      # mcast_if = eth0
      
      # 绑定地址
      bind = 239.2.11.71
      # 监听端口
      port = 8649
      # 网络协议（IPV4: inet4; IPV6: inet6;）
      # family = inet4
      # 是否重试绑定
      retry_bind = true
      # Size of the UDP buffer.
      # buffer = 10485760
    }
    
    tcp_accept_channel {
      # 网络协议（IPV4: inet4; IPV6: inet6;）
      # family = inet4
      # 绑定地址
      # bind = 239.2.11.71
      # 监听端口
      port = 8649
      # 绑定网络设备
      # interface = eth0
      # 是否启用gzip传输XML
      gzip_output = no
    }

# 2.3 WEB站点配置 ( /etc/httpd/conf.d/ganglia.conf )

	Alias /ganglia /usr/share/ganglia
    <Location /ganglia>
      Order Deny,Allow
      Deny from all
      Allow from all
    </Location>

# 2.4 关闭SELINUX ( /etc/selinux/config )

	SELINUX=disable
	
注： 需重启机器。（还可执行指令"setenforce 0"临时关闭SELINUX）

# 3.启动服务

	service httpd start
	service gmond start
	service gmetad start

# 4.访问站点

	http://HOST:PORT/ganglia