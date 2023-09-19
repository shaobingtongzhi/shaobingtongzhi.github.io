---
title: Linux 防火墙
date: 2023-07-23
tags: 
    - Linux
    - 防火墙
    - iptables
    - firewalld
    - es登录授权
categories:
    - 学习笔记 
    - 06-Linux相关知识
---
# Linux里的防火墙

> 注意：iptables 和 firewalld 服务不能同时启动

## iptables 防火墙

### iptables 简介

iptables是集成在***Linux内核***中的***包过滤防火墙系统***。使用iptable可以添加、删除具体的过滤规则，iptables默认维护着4个表和5个链，所有的防火墙策略规则都被分别写入这些表与链中

“四表”是指iptables的功能，默认的iptables规则表有filter表（过滤规则表）、nat表（地址转换规则表）、mangle（修改数据标记位规则表）、raw（跟踪数据表规则表）：

1. filter表：控制数据包是否允许进出及转发，可以控制的链路有INPUT、FORWARD和OUTPUT。

2. nat表：控制数据包中地址转换，可以控制的链路有PREROUTING、INPUT、OUTPUT和POSTROUTING。

3. mangle：修改数据包中的原数据，可以控制的链路有PREROUTING、INPUT、OUTPUT、FORWARD和POSTROUTING。

4. raw：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT。

“五链”是指内核中控制网络的NetFilter 定义的 5 个规则链。

每个规则表中包含多个数据链：

1. INPUT（入站数据过滤）。

2. OUTPUT（出站数据过滤）。

3. FORWARD（转发数据过滤）。

4. PREROUTING（路由前过滤）

5. POSTROUTING（路由后过滤）。

### iptables 语法格式

```
iptables [-t table] COMMAND [chain] CRETIRIA -j ACTION

各参数的含义为：
①-t：指定需要维护的防火墙规则表filter、nat、mangle或raw。在不使用-t 时则默认使用filter表。
②COMMAND：子命令，定义对规则的管理。
③chain：指明链表。
④CRETIRIA：匹配参数。
⑤ACTION：触发动作。
```

### 查看 iptables 规则

```sh
iptables -L # 查看iptables规则
iptables -L -vn # 查看iptables规则(详细信息)
```

### 清空 iptables 规则

```sh
iptables -F # 清除所有规则，不会处理默认的规则
iptables -X # 删除用户自定义的链
iptables -Z # 链的计数器清零(数据包计数器与数据包字节计数器)
```

### 添加 iptables 规则

```sh
iptables -t # 指定表(default: `filter')
iptables -A # 把规则添加到指定的链上，默认添加到最后一行
iptables -I # 插入规则，默认插入到第一行(封IP)
iptables -D # 删除链上的规则
```

### 删除某条规则

```sh
iptables -nL --line-numbers # 查看规则号码
iptables -D INPUT 1 # 删除指定链上的指定序号
```



## Firewalld 防火墙

### 前言

防火墙是Linux系统的主要的安全工具 ，可以提供基本的安全防护，在Linux历史上已经使用过的防火墙工具包括：ipfwadm、ipchains、iptables（即Centos6就是使用的iptables），而在firewalld中新引入了 区域（Zone）这个概念。

以前的iptables防火墙是静态的，每次修改都要求防火墙完全重启，这个过程包括内核netfilter防火墙模块的卸载和新配置所需模块的装载等，而模块的卸载将会破坏状态防火墙和确立建立的连接，现在firewalld可以动态管理防火墙，firewalld把netfilter的过滤功能集于一身。

### 基本操作

#### 启动、自启动、注销

```sh
# 启动防火墙
systemctl start firewalld
# 暂时关闭防火墙
systemctl stop firewalld
# 开启防火墙自启动
systemctl enable firewalld
# 关闭防火墙自启动
systemctl disable firewalld
# 注销firewalld服务，意味着系统重启后firewalld服务也不会启动
# 不能使用systemctl disable/enable 操作
# 不能使用systemctl start/stop 操作
systemctl mask firewalld
# 解除注销
systemctl unmask firewalld

# 重启服务
systemctl restart firewalld
```

#### 查看状态

```sh
systemctl status firewalld
```

### 区域

#### 概念

- 一个 zone 就是一套过滤规则，数据包必须要经过某个 zone 才能入站或出站。不同 zone 中规则粒度粗细、安全强度都不尽相同。可以把 zone 看作是一个个出站或入站必须经过的安检门，有的严格、有的宽松、有的检查细致、有的检查粗略
- 每个 zone 单独对应一个 xml 配置文件，在目录 /usr/lib/firewalld/services/ 下，文件名为 <zone名称>.xml。自定义 zone 只需要添加一个 <zone名称>.xml 文件，然后在其中添加过滤规则即可
- 每个 zone 都有一个默认的处理行为，包括：default(省缺)、ACCEPT、REJECT、DROP

#### 类型

firewalld 将网卡对应到不同的区域（zone），zone 默认共有 9 个区域：block，dmz，drop，external，home，internal，public，trusted，work。

- trusted（信任区域）	允许所有网络流量连接，即使没有开放任何服务，那么使用此 zone 的流量照样通过
- public（公共区域）	默认的 zone，部分公开，不信任网络中其他计算机，只放行特定服务
- external（外部区域）	允许与 ssh 预定义的服务传入流量，其余均拒绝。默认将通过此区域转发的 IPv4 传出流量进行地址伪装可用于为路由器启用了伪装功能的外部网络
- home（家庭区域）	允许与 ssh、ipp-client、mdns、samba-client 或 dhcpv6-client 预定义的服务传入流量，其余均拒绝
- internal（内部区域）	默认值时与 home 区域相同
- work（工作区域）	允许与 ssh、ipp-client、dhcpv6-client 预定义的服务传入流量，其余均拒绝
- dmz（隔离区域也称为非军事区域）	允许与 ssh 预定义的服务传入流量，其余均拒绝
- block（限制区域）	任何流入的包都被拒绝，返回 icmp-host-prohibited 报文（ipv4）或 icmp6-adm-prohibited 报文（ipv6）。只允许由该系统初始化的网络连接
- drop（丢弃区域）	任何流入的包都被丢弃，不做任何响应，只允许流出的数据包
  

### firewalld 服务

- 在 /usr/lib/firewalld/services/ 目录中，还保存了另外一类配置文件，每个文件对应一项具体的网络服务，如 ssh 服务等
- 与之对应的配置文件中记录了各项服务所使用的 tcp/udp 端口，在最新版本的 firewalld 中默认已经定义了 70 多种服务供我们使用
- 当默认提供的服务不够用或者需要自定义某项服务的端口时，我们需要将 service 配置文件放置在 /etc/firewalld/services/ 目录中

```sh
/etc/firewalld/		中存放修改过的配置（优先查找，找不到再找默认的配置）
/usr/lib/firewalld/	默认的配置信息
```

正常情况下，firewalld是默认开启ssh服务的，当我们开启firewalld的时候，不会影响ssh连接。这个我们可以通过查看firewall配置文件确认。

```sh
cat /etc/firewalld/zones/public.xml    
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>  #表示ssh服务开放
</zone>
```

但其实，防火墙开放服务时，会开放服务的默认端口。可以检查ssh端口是否是默认端口22.

```sh
cat /etc/ssh/sshd_config |grep Port

[root@localhost zones]# cat /etc/ssh/sshd_config |grep Port
#Port 22
#GatewayPorts no
```

发现端口是22，如果不是22,比如是2222那怎么办，修改public.xml即可

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>  #表示ssh服务开放
  <service name="dhcpv6-client"/>
  <port protocol="tcp" port="2222"/> 
</zone>
```



### firewall-cmd 命令行配置工具

#### 查看防火墙状态

```sh
firewall-cmd --state
```



### 服务管理

为了方便管理，firewalld预先定义了很多服务 ， 存 放 在**/usr/lib/firewalld/services/**目录中，服务通过单个的XML 配置文件来指定。这些配置文件则按以下格式命名：service-name.xml，每个文件对应一项具体的网络服务，如 ssh 服 务等。与之对应的配置文件中记录了各项服务所使用的 tcp/udp 端口。在最新版本的 firewalld 中默认已经定义了 70 多种服务供我们使用，对于每个网络区域，均可以配置允许访问的服务。当默认提供的服务不适用或者需要自定义某项服务的端口时，我们需要将 service 配置文件放置在/etc/firewalld/services/ 目录中。service 配置具有以下优点。

- 通过服务名字来管理规则更加人性化
- 通过服务来组织端口分组的模式更加高效，如果一个服务使用了若干个网络端口，则服务的配置文件就相当于提供了到这些端口的规则管理的批量操作快捷方式

```
[--zone=<zone>] --list-services 显示指定区域内允许访问的所有服务 
[--zone=<zone>] --add-service=<service> 为指定区域设置允许访问的某项服务 
[--zone=<zone>] --remove-service=<service> 删除指定区域已设置的允许访问的某项服务 
[--zone=<zone>] --list-ports 显示指定区域内允许访问的所有端口号
[--zone=<zone>] --add-port=<portid>[-<portid>]/<protocol> 为指定区域设置允许访问的某个/某段端口号 （包括协议名） 
[--zone=<zone>] --remove-port=<portid>[-<portid>]/<protocol> 删除指定区域已设置的允许访问的端口号（包括 协议名） 
[--zone=<zone>] --list-icmp-blocks 显示指定区域内拒绝访问的所有 ICMP 类型 
[--zone=<zone>] --add-icmp-block=<icmptype> 为指定区域设置拒绝访问的某项 ICMP 类型 
[--zone=<zone>] --remove-icmp-block=<icmptype> 删除指定区域已设置的拒绝访问的某项 ICMP 类 型
省略--zone=<zone>时表示对默认区域操作 
```

### 端口管理

在进行服务配置时，预定义的网络服务可以使用服务名配置，服务所涉及的端口就会自动打开。但是，***对于非预定义的服务只能手动为指定的区域添加端口***。

```sh
[root@localhost ~]# firewall-cmd --zone=internal --add-port=443/tcp  #在 internal 区域增加 443/TCP 端口
success
[root@localhost ~]#firewall-cmd --zone=internal --remove-port=443/tcp #在 internal 区域移除 443/TCP 端口
success
```



### 两种配置模式

- 运行时模式（Runtime mode）表示 当前内存中运行的防火墙配置，在系统或 firewalld 服务重启、停止时配置将失效
- 永久模式（Permanent mode）表示重启防火墙或重新加载防火墙时的规则配置，是永久存储在配置 文件中的

firewall-cmd 命令工具与配置模式相关的选项有三个：

- –reload：重新加载防火墙规则并保持状态信息，即将永久配置应用为运行时配置
- –permanent：带有此选项的命令用于设置永久性规则，这些规则只有在重新启动 firewalld 或重新加载防火墙规则时才会生效；若不带有此选项，表示用于设置运行时 规则
- –runtime-to-permanent：将当前的运行时配置写入规则配置文件中，使之成为永久性
  

## 实战

客户护网行动扫出elasticsearch的未授权访问漏洞，针对这个漏洞进行修复

- 限制9200端口的访问
- 对elasticsearch进行登录授权

### firewalld 解决办法

1. 查看firewalld服务状态

```sh
systemctl status firewalld 
```



> 注意：开启防火墙前要确认 ssh 的 22端口以及mysql的3306、redis的6379、80等服务端口
>
> cat /etc/firewalld/zones/public.xml  

2. 开启firewalld

```sh
systemctl start firewalld
```

3. 查看已设置的富规则

```sh
firewall-cmd --zone=public --list-rich-rules
firewall-cmd --list-all
```

> 因为客户环境使用的是docker 容器，所以官网描述 docker和firewall是有冲突的，具体冲突原因：https://docs.docker.com/network/packet-filtering-firewalls/
>
> 大概意思就是：firewall的底层是使用iptables进行数据过滤，建立在iptables之上，而docker使用iptables来进行网络隔离和管理，这可能会与 Docker 产生冲突。当 firewalld 启动或者重启的时候，将会从 iptables 中移除 DOCKER 的规则，从而影响了 Docker 的正常工作

4. 解决docker 和 firewall 的冲突

让docker绕过firewall

```sh
systemctl stop firewalld

####### 2、修改docker配置 #######
#修改docker配置
vim /etc/docker/daemon.json
#添加规则
{
...
"experimental" : true,
"iptables": false
}
#重启docker
systemctl daemon-reload
systemctl restart docker

####### 3、开启或重启防火墙 #######
systemctl restart firewalld
 
####### 4、再次重启docker #######
systemctl restart docker

```

***【重要一】***
绕过iptables之后，有个弊端：容器内部无法访问外界IP。如果需要访问，那么打开firewalld防火墙，且开启NAT转发功能，详细步骤如下：

此外，建议firewall防火墙开启NAT转发功能，解决阻止docker容器访问外界IP
见 https://blog.csdn.net/liyanggyang/article/details/128838350

***【重要二】***
在打开firewalld防火墙，且开启NAT转发功能后，允许容器访问外界IP：
1、容器访问本宿主机(物理/虚拟机)IP，本宿主机的防火墙检测到的IP是容器IP， 所以本机防火墙需要放行docker网段（可见“firewall-cmd命令可参考”）
2、访问其他外界物理/虚拟机，外界物理/虚拟机检测到的IP是容器所在宿主机IP。

***参考：https://blog.csdn.net/liyanggyang/article/details/130386522***



***开启防火墙 · 解决阻止docker容器访问外界IP ｜NAT转发***

- 检查是否允许 NAT 转发

```sh
firewall-cmd --query-masquerade
```

- 开启 NAT 转发

```sh
firewall-cmd --permanent --zone=public --add-masquerade && firewall-cmd --reload
```

- 禁止防火墙 NAT 转发

```sh
firewall-cmd --permanent --remove-masquerade && firewall-cmd --reload
```

5. 查看php容器ip

```sh
docker inspect php72
```



6. 针对php容器可以访问9200端口

```sh
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="172.17.0.3" port protocol="tcp" port="9200" accept" && firewall-cmd --reload
```

> 注意：执行后，一定要 重新载入设置 firewall-cmd --reload
>
> 补充删除规则的命令
>
> firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="172.17.0.3" port protocol="tcp" port="9200" accept" && firewall-cmd --reload

7. 开放8272、8306、6379端口

```sh
firewall-cmd --zone=public --add-port=8272/tcp --permanent && firewall-cmd --reload
firewall-cmd --zone=public --add-port=8306/tcp --permanent && firewall-cmd --reload
firewall-cmd --zone=public --add-port=6379/tcp --permanent && firewall-cmd --reload
```

> 补充删除规则命令
>
> firewall-cmd --zone=public --remove-port=9200/tcp --permanent && firewall-cmd --reload

8. 查看已设置的规则

```sh
firewall-cmd --zone=public --list-all
```

9. 查看端口有没有生效

```sh
firewall-cmd --zone=public --query-port=8272/tcp
```

### elasticsearch 登录授权

1. 复制http-basic.zip文件到es容器指定目录
   docker cp http-basic.zip es654:/usr/share/elasticsearch/plugins

2. 进入es容器
   docker exec -it es654 bash

3. 解压文件
   cd plugins	
   unzip http-basic.zip
   rm -rf http-basic.zip

4. 配置http-baisc，设置es的版本信息
   cd http-basic
   vi plugin-descriptor.properties

```properites
elasticsearch.version=6.5.4
```
5. 配置es
   cd /usr/share/elasticsearch/config
   vi elasticsearch.yml 

```yml
cluster.name: "docker-cluster"
network.host: 0.0.0.0
xpack.security.enabled: false
# minimum_master_nodes need to be explicitly set when bound on a public IP
# set to 1 to allow single node clusters
# Details: https://github.com/elastic/elasticsearch/pull/17288
discovery.zen.minimum_master_nodes: 1
http.basic.enabled: true
http.basic.log: true
http.basic.username: "ccbscf"
http.basic.password: "$.ccbscf#"
http.basic.ipwhitelist: ""                    
```
6. 修改代码
apps/cosola_V10/vendor/elasticsearch/elasticsearch/src/Elasticsearch/ClientBuilder 624行 修改代码

```php
foreach ($hosts as $host) {
    if (is_string($host)) {
        $host = $this->prependMissingScheme($host);
        $host = $this->extractURIParts($host);
    } elseif (is_array($host)) {
        $host = $this->normalizeExtendedHost($host);
    } else {
        $this->logger->error("Could not parse host: ".print_r($host, true));
        throw new RuntimeException("Could not parse host: ".print_r($host, true));
    }
    $host['user'] = "user";   //增加用户名
    $host['pass'] = "pass";  //增加密码
    $connections[] = $this->connectionFactory->create($host);
}
```
7. 重启es容器

docker restart es654

8. 验证 http-basic

 - curl 127.0.0.1:9200
 - curl --user user:pass 127.0.0.1:9200
 - http://xxxx.xxx.com/public/index.php?app=core&mod=Test&act=testEs
 - 浏览器 ip:9200 访问
