# Git多用户配置

一般来说我们在公司的git账户都不会是自己的真实git账户，我们可以通过以下配置让我们的一台电脑上有两个git账户，两个账户各司其职。

首先设置一个全局git用户，建议设置为自己的

.gitconfig

```sh
[user]
        name = x43125
        email = wx43125@163.com
[includeIf "gitdir:~/workspace/qt/"]
        path = .gitconfig-qt
```

.gitconfig-qt

```sh
[user]
        name = wangxiang
        email = wangxiang@flashhold.com
```

.ssh/config

```sh
Host qt
HostName 172.31.234.12
User wangxiang
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_qt

Host github
HostName github.com
User x43125
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_43125
```

