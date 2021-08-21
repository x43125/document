# 虚拟机相关配置

## 1 配置静态IP

```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

```bash
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"	-- 静态
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="e308b6c0-254b-4036-9432-4ecacc1f9a87"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.11.128"		-- IP
GATEWAY="192.168.11.2"		-- 网关
NETMASK="255.255.255.0"		-- 子网掩码
DNS1="8.8.8.8"		-- DNS
DNS2="114.114.114.114"
```

## 2 关闭防火墙/关闭开机自启

```bash
systemctl stop firewalld
systemctl disable firewalld
```

## 3 常用工具

```bash
yum install -y vim
yum install -y rpm 
yum install -y net-tools 
yum install -y dos2unix 
yum install -y telnet
```

## 4 设置节点hostname

```bash
hostnamectl set-hostname `node1`
```

## 5 设置节点间hosts映射

```bash
vim /etc/hosts

192.168.11.143 node01
192.168.11.144 node02
192.168.11.145 node03
```

## 6 免密登录

```bash
在需要免密登录其他节点的节点上输入以下生成公钥密钥
如使用默认设置，一路回车即可（一般默认设置即可，想要改也可以自行更改）
ssh-keygen
然后在输入以下命令将公钥传到其他节点
ssh-copy-id node01
ssh-copy-id node02
ssh-copy-id node03
```

```bash
scp /etc/hosts node2:/etc -- 传输指令
```

## 7 Java

```bash
openjdk安装：

yum search java
yum install -y java..._devel.x86_64

-- 环境变量
vim /etc/profile.d/java.sh
export JAVA_HOME=/usr/local/jdk1.8.0_271  	 -- openjdk默认安装位置为/usr/lib/jvm/
export PATH=$JAVA_HOME/bin:$JAVA_HOME/sbin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```