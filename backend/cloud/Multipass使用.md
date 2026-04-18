# Multipass使用

官网：https://canonical.com/multipass

使用文档：https://documentation.ubuntu.com/multipass/latest/

## Multipass新建服务器基本操作

```sh
> multipass launch -n ubt01 -c 1 -m 2G -d 20G    // 生成一个叫ubt01、1核2g20g磁盘的虚拟机
> multipass ls    // 查看当前所有的运行中虚拟机
Name                    State             IPv4             Image
ubt01                   Running           192.168.2.5      Ubuntu 24.04 LTS
> multipass info ubt01    // 查看某虚拟机的信息
Name:           ubt01
State:          Running
Snapshots:      0
IPv4:           192.168.2.5
Release:        Ubuntu 24.04.3 LTS
Image hash:     a40713938d74 (Ubuntu 24.04 LTS)
CPU(s):         1
Load:           0.13 0.05 0.02
Disk usage:     2.0GiB out of 19.3GiB
Memory usage:   224.8MiB out of 1.9GiB
Mounts:         --

> multipass shell ubt01
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-90-generic aarch64)
......
ubuntu@ubt01:~$ sudo passwd ubuntu    // 修改ubuntu账号密码
New password:
Retype new password:
passwd: password updated successfully
ubuntu@ubt01:~$ sudo passwd root    // 修改root账号密码
New password:
Retype new password:
passwd: password updated successfully
ubuntu@ubt01:~$ su root    // 切换到root下用来修改可密码远程访问
Password:
root@ubt01:/home/ubuntu#
root@ubt01:/home/ubuntu# vim /etc/ssh/sshd_config    // 在 # Authentication下新增下面两行
# Authentication:
PermitRootLogin yes
passwordAuthentication yes

root@ubt01:/home/ubuntu# vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf    // 将no改为yes
root@ubt01:/home/ubuntu# sudo service ssh restart    // 重启SSH服务，即可远程密码访问
```