# 命令行激活Windows

使用管理员打开cmd，并依次键入以下命令

每次命令回车后都会弹窗出对应的回应，并且cmd会被强行关闭（视情况而定）重新打开继续即可

所有命令执行完后即可成功激活6个月180天

```sh
# “已成功卸载了产品密钥”
slmgr.vbs /upk
# “成功的安装了产品密钥”
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
# “密钥管理服务计算机名成功的设置为zh.us.to”
slmgr /skms zh.us.to
# “成功的激活了产品”
slmgr /ato
# ”批量激活的过期时间”
slmgr /xpr
```

