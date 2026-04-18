## 1、修改Linux命令行颜色

vim .bashrc

添加上一行

```bash
PS1="\[\e[37;40m\][\[\e[32;40m\]\u\[\e[37;40m\]@\h \[\e[36;40m\]\w\[\e[0m\]]\\$ "
```

添加完之后，保存退出，执行命令

```bash
source .bashrc
```

## 2、添加alias

```sh
alias dps='docker ps --format "table  {{.ID}}\t{{.Status}}\t{{.Names}}\t{{.Image}}"'
```

