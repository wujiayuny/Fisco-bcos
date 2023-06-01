# Webase一键部署

本次搭建webase一键部署采用的是 **非国密** 且是 **搭建新链** 进行安装！！

配置是4G 2核

```shell
# 配置环境时，如要切换到 root 用户下，第一次使用要解除root锁定，为root用户设置密码。
sudo passwd 
我设置的密码均为000000    六个0

#vim乱码问题
apt-get update
apt-get install -y vim

#配置ssh
apt install -y openssh-server 
vim /etc/sshd_config

#取消注释22端口、以及开启root登录
#Port 22   第13行
Port 22

#PermitRootLogin prohibit-password  第33行
PermitRootLogin yes

#重启服务
service ssh resart 

#Can't load /root/.rnd into RNG
cd /root
openssl rand -writerand .rnd
```



#### **环境是ubuntu 18.04  环境一定要用18.04的啊！！！**

```shell
root@wujiayun:/home/wujiayun# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.6 LTS
Release:        18.04
Codename:       bionic

```



#### **java采用tar包的形式安装**

```shell
mkdir /usr/lib/jvm
tar -zxvf /root/jdk-8u73-linux-x64.tar.gz -C /usr/lib/jvm
vim /etc/profile

export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_121 
export PATH=${JAVA_HOME}/bin:$PATH

source /etc/profile

#检查java版本
java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)

```



#### **mysql的安装 5.7版本**

```shell
apt-get install software-properties-common
sudo add-apt-repository 'deb http://archive.ubuntu.com/ubuntu trusty universe'
sudo apt-get update
sudo apt install -y mysql-server-5.7
sudo apt install -y mysql-client-5.7

#如果下载不了、可以用mariadb！干嘛非要在mysql上死磕、换个方向多条路啊(莫要在mysql这棵树上吊死!)    于2023.6.1更新、祝各位大朋友、小朋友六一儿童节快乐！
apt install -y mariadb-server-10.3

这里下载是5.7版本 不要下载5.6版本
#查看mysql版本
mysql --version

#使用root用户登录，密码为初始化设置的密码
mysql -uroot -p -h localhost -P 3306

#授权root用户远程访问
mysql > GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '000000' WITH GRANT OPTION;
mysql > flush PRIVILEGES;

#创建test用户并授权本地访问
mysql > GRANT ALL PRIVILEGES ON *.* TO 'test'@localhost IDENTIFIED BY '000000' WITH GRANT OPTION;
mysql > flush PRIVILEGES;

#测试连接和创建数据库
mysql -utest -p000000 -h localhost -P 3306

```



#### **Python3部署**

```shell
# 添加仓库，“回车”继续！！！
sudo add-apt-repository ppa:deadsnakes/ppa

#安装python 3.6
sudo apt-get install -y python3.6
sudo apt-get install -y python3-pip

#查看python3版本
python3 --version
```



#### PyMySQL部署（Python3.6+）

```shell
apt-get install -y python3-pip
pip3 install PyMySQL

#检查PyMySQl版本
pip3 show PyMySQl
```



#### 拉取脚本

```shell
wget https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeBASE/releases/download/v1.5.5/webase-deploy.zip
unzip webase-deploy.zip
cd webase-deploy
vim common.properties

# WeBASE子系统的最新版本(v1.1.0或以上版本)
webase.web.version=v1.5.5
webase.mgr.version=v1.5.5
webase.sign.version=v1.5.5
webase.front.version=v1.5.5

#####################################################################
## 使用Docker启用Mysql服务，则需要配置以下值

# 1: enable mysql in docker
# 0: mysql run in host, required fill in the configuration of webase-node-mgr and webase-sign
docker.mysql=1

# if [docker.mysql=1], mysql run in host (only works in [installDockerAll])
# run mysql 5.6 by docker
docker.mysql.port=23306
# default user [root]
docker.mysql.password=123456

#####################################################################
## 不使用Docker启动Mysql，则需要配置以下值

# 节点管理子系统mysql数据库配置
mysql.ip=127.0.0.1
mysql.port=3306
mysql.user=dbUsername   #将“dbUsername”修改为test用户
mysql.password=dbPassword   #修改“dbPassword”为000000
mysql.database=webasenodemanager

# 签名服务子系统mysql数据库配置
sign.mysql.ip=localhost
sign.mysql.port=3306
sign.mysql.user=dbUsername  #将“dbUsername”修改为test用户
sign.mysql.password=dbPassword  #修改“dbPassword”为000000
sign.mysql.database=webasesign



# 节点前置子系统h2数据库名和所属机构
front.h2.name=webasefront
front.org=fisco

# WeBASE管理平台服务端口
web.port=5000
# 启用移动端管理平台 (0: disable, 1: enable)
web.h5.enable=1

# 节点管理子系统服务端口
mgr.port=5001
# 节点前置子系统端口
front.port=5002
# 签名服务子系统端口
sign.port=5004


# 节点监听Ip
node.listenIp=127.0.0.1
# 节点p2p端口
node.p2pPort=30300
# 节点链上链下端口
node.channelPort=20200
# 节点rpc端口
node.rpcPort=8545

# 加密类型 (0: ECDSA算法, 1: 国密算法)
encrypt.type=0
# SSL连接加密类型 (0: ECDSA SSL, 1: 国密SSL)
# 只有国密链才能使用国密SSL
encrypt.sslType=0

# 是否使用已有的链（yes/no）
if.exist.fisco=no

因为这次搭建是新搭建链所以上面是no
# 使用已有链时需配置
# 已有链的路径，start_all.sh脚本所在路径
# 路径下要存在sdk目录（sdk目录中包含了SSL所需的证书，即ca.crt、sdk.crt、sdk.key和gm目录（包含国密SSL证书，gmca.crt、gmsdk.crt、gmsdk.key、gmensdk.crt和gmensdk.key）
fisco.dir=/data/app/nodes/127.0.0.1
# 前置所连接节点，在127.0.0.1目录中的节点中的一个
# 节点路径下要存在conf文件夹，conf里存放节点证书（ca.crt、node.crt和node.key）
node.dir=node0

# 搭建新链时需配置
# FISCO-BCOS版本
fisco.version=2.9.1
# 搭建节点个数（默认两个）
node.counts=nodeCounts  #如果想搭建四个链 改为4就好了 这个根据自己需求来改 默认是2个

```



#### 部署

- 执行installAll命令，部署服务将自动部署FISCO BCOS节点，并部署 WeBASE 中间件服务，包括签名服务（sign）、节点前置（front）、节点管理服务（node-mgr）、节点管理前端（web）

```shell
# 部署并启动所有服务
python3 deploy.py installAll
```



部署完成后可以看到`deploy has completed`的日志：

```shell
python3 deploy.py installAll
...
============================================================
              _    _     ______  ___  _____ _____ 
             | |  | |    | ___ \/ _ \/  ___|  ___|
             | |  | | ___| |_/ / /_\ \ `--.| |__  
             | |/\| |/ _ | ___ |  _  |`--. |  __| 
             \  /\  |  __| |_/ | | | /\__/ | |___ 
              \/  \/ \___\____/\_| |_\____/\____/  
...
...
============================================================
==============      deploy  has completed     ==============
============================================================
==============    webase-web version  v1.5.5        ========
==============    webase-node-mgr version  v1.5.5   ========
==============    webase-sign version  v1.5.3       ========
==============    webase-front version  v1.5.5      ========
============================================================
```



#### 检查各子系统进程

检查节点进程，此处部署了两个节点node0, node1

```shell
 ps -ef | grep node
 root     29977     1  1 17:24 pts/2    00:02:20 /root/fisco/webase/webase-deploy/nodes/127.0.0.1/node1/../fisco-bcos -c config.ini
root     29979     1  1 17:24 pts/2    00:02:23 /root/fisco/webase/webase-deploy/nodes/127.0.0.1/node0/../fisco-bcos -c config.ini
```



检查节点前置webase-front的进程

```shell
ps -ef | grep webase.front 
root     31805     1  0 17:24 pts/2    00:01:30 /usr/local/jdk/bin/java -Djdk.tls.namedGroups=secp256k1 ... conf/:apps/*:lib/* com.webank.webase.front.Application

```



检查节点管理服务webase-node-manager的进程

```shell
ps -ef  | grep webase.node.mgr
root      4696     1  0 17:26 pts/2    00:00:40 /usr/local/jdk/bin/java -Djdk.tls.namedGroups=secp256k1 ... conf/:apps/*:lib/* com.webank.webase.node.mgr.Application

```



检查webase-web对应的nginx进程

```shell
ps -ef | grep nginx
root      5141     1  0 Dec08 ?        00:00:00 nginx: master process /usr/sbin/nginx -c /root/fisco/webase/webase-deploy/comm/nginx.conf

```



检查签名服务webase-sign的进程

```shell
ps -ef  | grep webase.sign
root     30718     1  0 17:24 pts/2    00:00:19 /usr/local/jdk/bin/java ... conf/:apps/*:lib/* com.webank.webase.sign.Application

```



检查节点channel端口(默认为20200)是否已监听

```shell
$ netstat -anlp | grep 20200
tcp        0      0 0.0.0.0:20200           0.0.0.0:*               LISTEN      29069/fisco-bcos
```



检查webase-front端口(默认为5002)是否已监听

```shell
$ netstat -anlp | grep 5002
tcp6       0      0 :::5002                 :::*                    LISTEN      2909/java 
```



检查webase-node-mgr端口(默认为5001)是否已监听

```shell
$ netstat -anlp | grep 5001  
tcp6       0      0 :::5001                 :::*                    LISTEN      14049/java 
```



检查webase-web端口(默认为5000)在nginx是否已监听

```shell
$ netstat -anlp | grep 5000
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      3498/nginx: master  
```



检查webase-sign端口(默认为5004)是否已监听

```shell
$ netstat -anlp | grep 5004
tcp6       0      0 :::5004                 :::*                    LISTEN      25271/java 
```

