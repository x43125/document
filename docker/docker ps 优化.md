# docker ps 优化

修改 `~/.basrc`

文件在末尾添加此行，保存后退出

```sh
alias dps='docker ps --format "table  {{.ID}}\t{{.Status}}\t{{.Names}}\t{{.Image}}"'
```

使用命令刷新环境变量 `source ~/.bashrc`

