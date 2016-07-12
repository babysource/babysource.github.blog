---
title: ELK部署实战
date: 2016-07-12 18:00:00
updated: 2016-07-12 18:00:00
tags:
categories: 运维测试
---
ELK是ElasticSearch、Logstash、Kibana三个开源软件的缩写。在实时数据检索和分析场合，通常是三者组合使用，故有此简称。

ElasticSearch简称ES，是个开源分布式搜索引擎，主要用来存储和检索数据。

Logstash主要用来往ES中写入数据，它可以对日志进行收集、分析，并将其存储供以后使用。

Kibana主要用来展示数据，提供日志分析友好的WEB界面，可以帮助汇总、分析和搜索重要数据日志。

# 部署环境

> Topbeat 1.2.3

> Filebeat 1.2.3

> Kibana 4.5.1

> Logstash 2.3.3

> Elasticsearch 2.3.3

> beats-dashboards-1.2.3

> OpenJDK version "1.8.0_91"

> CentOS Linux release 7.2.1511 (Core)

> Linux version 3.10.0-123.el7.x86_64

# 1.安装ELK

## 1.1 下载ELK

	wget https://download.elastic.co/kibana/kibana/kibana-4.5.1-1.x86_64.rpm
	
	wget https://download.elastic.co/logstash/logstash/packages/centos/logstash-2.3.3-1.noarch.rpm
	
	wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/rpm/elasticsearch/2.3.3/elasticsearch-2.3.3.rpm
	
	wget https://download.elastic.co/beats/topbeat/topbeat-1.2.3-x86_64.rpm
	
	wget https://download.elastic.co/beats/filebeat/filebeat-1.2.3-x86_64.rpm

## 1.2 服务端安装ELK

	yum -y localinstall elasticsearch-2.3.3.rpm
	
	yum -y localinstall kibana-4.5.1-1.x86_64.rpm
	
	yum -y localinstall logstash-2.3.3-1.noarch.rpm

## 1.3 客户端安装Beat

	yum -y localinstall topbeat-1.2.3-x86_64.rpm
	
	yum -y localinstall filebeat-1.2.3-x86_64.rpm

# 2.配置ELK

注： 未使用SSL的配置方案；

## 2.1 配置Kibana （ /opt/kibana/config/kibana.yml ）
	
	# 服务端口（默认：5601）
	server.port
	# 绑定主机
	server.host
	
	# 索引名称
	kibana.index
	# 默认应用
	kibana.defaultAppId
	
	# Elasticsearch 网站地址
	elasticsearch.url
	# Elasticsearch 登录用户
	elasticsearch.username
	# Elasticsearch 登录密码
	elasticsearch.password
	# Elasticsearch 主机保护
	elasticsearch.preserveHost
	
	......

## 2.2 配置Logstash

### 2.2.1 创建配置文件 ( /etc/logstash/conf.d/logstash-initial.conf )

	touch /etc/logstash/conf.d/logstash-initial.conf
	
### 2.2.2 配置日志方案

	# 输入配置
	input {
	  beats {
		port => 5000
		type => "logs"
	  }
	}
	
	# 过滤规则
	filter {
	  if [type] == "syslog" {
		grok {
		  match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
		  add_field => [ "received_at", "%{@timestamp}" ]
		  add_field => [ "received_from", "%{host}" ]
		}
		geoip {
		  source => "clientip"
		}
		syslog_pri {}
		date {
		  match => [ "syslog_timestamp", "MMM d HH:mm:ss", "MMM dd HH:mm:ss" ]
		}
	  }
	}
	
	# 输出配置
	output {
	  elasticsearch {
	  	hosts => ["localhost:9200"]
	  	index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
	  }
	  stdout { codec => rubydebug }
	}

## 2.3 配置Elasticsearch ( /etc/elasticsearch/elasticsearch.yml )

	# 服务端口（默认：9200）
	http.port
	
	# 数据目录
	path.data
	# 日志目录
	path.logs

	# 节点名称
	node.name
	# 集群名称
	cluster.name
	# 绑定主机
	network.host
	
	......

## 2.4 配置Beats

### 2.4.1 配置FileBeat

#### 2.4.1.1 基础配置 ( /etc/filebeat/filebeat.yml )

	# 采集方案
	filebeat:
		# 最大发送数量
		spool_size: 1024
		# 空闲发送时长
		idle_timeout: 5s
		# 配置拆分目录
		config_dir: /etc/filebeat/conf.d
		# 记录文件目录
		registry_file: /var/lib/filebeat/registry
	
	# 采集报送
	output:
		logstash:
			# 主机地址
			hosts: ["localhost:5000"]
			
	......
	
#### 2.4.1.2 采集配置 ( /etc/filebeat/conf.d/*.yml )

	filebeat:
		prospectors:
			# 采集文件目录（支持通配符）
			- paths:
	      		- /var/log/messages
	      	# 可选附加字段
	      	fields:
	      		system: centos7
	      	# 采集文件编码
	      	encoding: plain
	      	# 是否末尾读取
	      	tail_files: false
	      	# 采集输入类型
	      	input_type: log
	      	# 采集数据类型
	      	document_type: syslog
	      	# 采集排除文件
	      	exclude_files: [".gz$"]
			# 采集排除内容
	      	exclude_lines: ["^DBG"]
			# 采集包含内容
	      	include_lines: ["^ERR", "^WARN"]
	      	# 文件更新检测时长
	      	backoff: 1s
	      	# 忽略监听过期时长
	      	ignore_older: 0
	      	# 目录扫描间隔时长
	      	scan_frequency: 10s
	      	# 文件读取缓冲大小
	      	harvester_buffer_size: 16384
	      	# 多行数据合并配置
	      	multiline:
	      		# 多行匹配规则
	      		pattern: ^\[
	      		# 多行匹配超时
	      		timeout: 5s
	      		# 多行合并模式
	      		match: after
	      		# 是否转置规则
	      		negate: false
	      		# 匹配最大行数
	      		max_lines: 500
	......

### 2.4.2 配置TopBeat ( /etc/topbeat/topbeat.yml )

	# 采集方案
	input:
		# 采集间隔频率
		period: 10
		# 采集进程规则
		procs: [".*"]
		stats:
			# 是否采集系统信息
			system: true
			# 是否采集进程信息
			process: true
			# 是否采集磁盘信息
			filesystem: true
			# 是否采集多核信息
			cpu_per_core: true
	
	# 采集报送
	output:
		logstash:
			# 主机地址
			hosts: ["localhost:5000"]
			# 索引名称
			index: topbeat
			
	......
	
## 2.5 配置防火墙 （ /etc/sysconfig/iptables ）

	-A INPUT -m state --state NEW -m tcp -p tcp --dport 5000 -j ACCEPT

	-A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
	
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 9300 -j ACCEPT
	
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 5601 -j ACCEPT
	
# 3.启动相关服务

	service kibana start
	service logstash start
	service elasticsearch start
	cd 
	service topbeat start
	service filebeat start

# 4.访问Kibana

	http://HOST:5601/
	
# 5.安装Dashboards

## 5.1 下载Beats-Dashboards

	https://download.elastic.co/beats/dashboards/beats-dashboards-1.2.3.zip
	
## 5.2 安装Beats-Dashboards
	
	./load.sh -url "http://localhost:9200"

# 6.安装ElastAlert

## 6.1 下载ElastAlert

	git clone https://github.com/Yelp/elastalert.git

## 6.2 安装依赖

	yum -y install libyaml
	yum -y install libyaml-devel 
	
	yum -y install python-devel
	yum -y install python-setuptools

## 6.3 执行安装

	python setup.py install

## 6.4 创建索引

	$ elastalert-create-index
	# 输入索引名
	New index name (Default elastalert_status)
	Name of existing index to copy (Default None)
	New index elastalert_status created
	Done!

## 6.5 创建配置

### 6.5.1 拷贝配置

	cp config.yaml.example config.yaml

### 6.5.2 配置选项

	# 告警规则目录
	rules_folder: rules
	# 告警轮询时长
	run_every:
	  minutes: 5
	# 缓存时间范围
	buffer_time:
	  minutes: 45
	# 时间范围字段
	timestamp_field: 
	# Elasticsearch 主机地址
	es_host: localhost
	# Elasticsearch 主机端口
	es_port: 9200
	# Elasticsearch 登录用户
	es_username:
	# Elasticsearch 登录密码
	es_password:
	# Elasticsearch 索引名称
	writeback_index: elastalert_status
	# 重试有效时长
	alert_time_limit:
	  days: 1

## 6.6 设定规则

	# Elasticsearch 主机地址
	es_host: localhost
	# Elasticsearch 主机端口
	es_port: 9200
	# 规则名称（唯一）
	name: 
	# 规则类型
	type: 
	# 依赖索引
	index: 
	# 累积触发报警时长（小时）
	timeframe:
  		hours: 1
  	# 过滤条件
  	filter:
  	# 告警方式
  	alert:

## 6.7 启动服务

	python -m elastalert.elastalert --config ./config.yaml

注：使用 `--verbose` 参数跟踪告警信息

# 7.相关指令与工具

## 7.1 ElastAlert规则测试

	elastalert-test-rule rule.yaml

## 7.2 Elasticsearch索引状态查询

	curl http://localhost:9200/_stats/indexes?pretty=1