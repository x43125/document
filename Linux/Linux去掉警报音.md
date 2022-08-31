# centos7 关闭命令行和vim警报音

centos 7

首先是命令行的

vim /etc/inputrc

然后将set bell-style none前面的#删掉

:wq 保存退出



然后是vim的

vim /etc/bashrc

在开始的地方加上一句 setterm -blength 0

:wq

保存退出