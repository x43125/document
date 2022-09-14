# docker

## 安装

docker要求centos系统的内核高于3.10，查看内核版本

```bash
 $ uname -r
```

使用root升级yum，

```bash
$ sudo yum -y update
```

安装依赖

```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

设置yum源

```bash
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

安装docker

```bash
$ sudo yum install -y docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
```

启动

```bash
$ sudo systemctl start docker
```

加入开机启动

```bash
$ sudo systemctl enable docker
```

验证

```bash
$ docker version
```



## 配置镜像加速

配置阿里云镜像加速

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://h72uhsdu.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

