### 备份

```shell
cd /etc/yum.repos.d/
mkdir repo_bak
mv *.repo repo_bak/
```

### 下载新的CentOS-Base.repo 到/etc/yum.repos.d/

```shell
wget http://mirrors.aliyun.com/repo/Centos-7.repo
```

### yum clean all 清除缓存，运行 yum makecache 生成新的缓存

```shell
yum clean all
yum makecache
```

### 安装EPEL（Extra Packages for Enterprise Linux ）源

```shell
yum install -y epel-release
```

### 再次运行yum clean all 清除缓存，运行 yum makecache 生成新的缓存

### 查看启用的yum源和所有的yum源

```shell
yum repolist enabled
yum repolist all
```

### 更新yum

```shell
yum -y update
```