## 修改ssh端口


```shell
# 拷贝配置文件，防止改错：
cp /etc/ssh/sshd_config /opt
vim /etc/ssh/sshd_config
##找到注释的 #Port 22 为了安全起见修改一个5位数，修改成 18586
systemctl restart sshd
```

## 新增DeployUser用户

指定默认包路径为:/home/ljdw，赋予sudo权限

```shell
# 创建用户以及主目录
groupadd LjdwDeploy
# 创建用户以及主目录
useradd -d /home/ljdw -m DeployUser

# 修改密码
passwd DeployUser

# 加入到sudoer用户中，授权
chmod 740 /etc/sudoers
# vim /etc/sudoers 添加
root ALL=(ALL) ALL
DeployUser    ALL=(ALL)       NOPASSWD:ALL
# 回收权限
chmod 440 /etc/sudoers
```


## 挂在磁盘

参考：https://blog.csdn.net/ju_362204801/article/details/122873049

### 参看分区

```shell
fdisk -l
```

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101201403.png)

### 挂载分区

```shell
fdisk /dev/vdb
```

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101201525.png)

### 再次查看


```shell
fdisk -l
```

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101201631.png)

### ext4 格式化

```
mkfs.ext4 /dev/vdb1
```

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101201718.png)

### 挂载到/home/ljdw/


```
mount /dev/vdb1 /home/ljdw/
```

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221101201807.png)

### 查看磁盘的id

```shell
blkid

## vim /etc/fstab

##  添加以下内容，然后保存
UUID=[上述磁盘的id] /home/ljdw              ext4    defaults        0 2

```

## 安装jdk环境



```shell
## 添加环境变量
vim /etc/profile

## 文件添加以下内容
export JAVA_HOME=/home/ljdw/Java/jdk1.8.0_202
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

```

## 安装docker和docker-compose

1.	查看docker是否曾经安装过

```
whereis docker
```

2.	如果安装过，则删除之前的版本

```
sudo yum remove docker docker-common docker-selinux docker-engine
```

3.	  安装需要的软件包，yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

4.	 设置yum源

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

或者

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

接着更新缓存

```
 sudo yum makecache fast
```


5.	 查看docker版本，一般使用稳定版

```
sudo yum list docker-ce --showduplicates | sort -r
```

6.	 安装docker

```
#默认安装最新版本
sudo yum -y install docker-ce

#安装某特定版本需增加版本号（如18.06.3.ce-3.el7）
sudo yum -y install docker-ce-18.06.3.ce
```

7.	 添加加速镜像

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://fwsn138q.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```


8.	修改镜像存储路径

```
# 创建目标目录
mkdir -p /home/ljdw/docker

# 拷贝原来的文件
sudo cp -rf /var/lib/docker/* /home/ljdw/docker

# 修改配置文件
sudo vim /usr/lib/systemd/system/docker.service
# 在 ExecStart 行的最后加上 --graph=/home/ljdw/docker

# 重新加载配置并启动
sudo systemctl daemon-reload
sudo systemctl restart docker

# 验证 
sudo docker info
```

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221104141837.png)



![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221104141859.png)

9.  添加当前用户到docker用户组，防止permission denied 报错

```
# 添加当前用户到docker用户组  ${USER}指用户名
sudo gpasswd -a ${USER} docker
# 查看用户组下用户，检查添加是否成功
cat /etc/group | grep docker
# 重启docker服务
sudo systemctl restart docker
# 切换当前会话到新组【group】或重启会话
newgrp - docker
```
10.  设置开机启动

```
sudo systemctl enable docker
```

11. 安装docker-compose

```
# 使用daoCloud 下载docker-compose 文件
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 下载完成以后 添加可执行命令
sudo chmod +x /usr/local/bin/docker-compose
# 验证
docker-compose --version
```
