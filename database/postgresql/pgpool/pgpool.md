# overview

pgpool可以只用一个，但如果pgpool坏了就无法继续调度了

pgpool可以和pg安装在一台机器上

pgpool通过投票选举出leader，因此需要奇数台



源码安装：

依赖包：postgresql-libs 和 postgresql-devel

