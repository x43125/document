# Expect工具的使用

自动ssh到远程机器上执行命令

```shell
#!/usr/bin/expect
set IP     [lindex $argv 0]
set USER   [lindex $argv 1]
set PASSWD [lindex $argv 2]
set CMD    [lindex $argv 3]
 
spawn ssh $USER@$IP $CMD
expect {
    "(yes/no)?" {
        send "yes\r"
        expect "password:"
        send "$PASSWD\r"
        }
    "password:" {send "$PASSWD\r"}
    "* to host" {exit 1}
    }
expect eof
```



```shell
#!/usr/bin/expect

spawn ssh root@192.168.11.128 'sh /root/test.sh'
expect {
    "(yes/no)?" {
        send "yes\r"
        expect "password:"
        send "root\r"
        }
    "password:" {send "root\r"}
    "* to host" {exit 1}
    }
expect eof
```

## 返回值
使用 `wait` 命令
```shell
#!/usr/bin/expect

spawn ssh root@192.168.11.128 'sh /root/test.sh'
expect {
    "(yes/no)?" {
        send "yes\r"
        expect "password:"
        send "root\r"
        }
    "password:" {send "root\r"}
    "* to host" {exit 1}
    }
expect eof
############################
catch wait result
exit [lindex \$result 3]
############################
```
