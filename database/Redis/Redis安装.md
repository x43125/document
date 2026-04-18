# Redis安装

## 1. 源码方式安装

官网下载源码，上传到服务器上后，解压，进入目录内并进入src目录，输入命令：make & make install

其中 第一个make 为大包 第二个 make install 为下载，将打包好的redis安装到指定目录下，此目录由打包文件`Makefile`里的 `PREFIX`来指定，只需要修改其后的值，即可以修改完成。