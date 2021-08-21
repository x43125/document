####  设置ssh免密登录

因为Flink集群版需要获得其他节点的直接访问权限，所以要先设置免密登录

可以在所有节点之间都设置免密登录，也可以只设置一个作为master机器，以后都用此台机器来作为总机，负责开关等集群操作

具体操作方式如下：

修改sshd配置文件

```bash
sudo vim /etc/ssh/sshd_config
增修改如下参数为yes
PasswordAuthentication yes
然后重启sshd服务
sudo systemctl restart sshd
```

假设我们有三台机器:

```
zhongfu-test004
zhongfu-test005
zhongfu-test006
```

我们设置zhongfu-test006为master

* 在zhongfu-test006上，首先使用指令创建公钥

```bash
$ ssh-keygen
```

* 使用默认位置，每次选项均直接回车，此命令执行完成之后会在根目录下生成一个文件夹，~/.ssh

```bash
$ cd ~/.ssh
```

* 如果有待访问机器的密码，即可以使用指令直接发送公钥

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@目标主机IP
```

* 如果没有，但可以登录，则将id_rsa.pub中的内容复制下来，进到其他两台机器的相同位置，如果没有.ssh文件夹，则自己创建。进入到.ssh文件夹下，手动创建文件authorized_keys，然后将刚刚复制的id_rsa.pub中的内容黏贴进去，则实现了从test006到其他两台机器的免密登录可以使用指令进行测试：

```bash
$ ssh root@zhongfu-test004
```