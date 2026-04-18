# python学习：
## python编码规范：
- 类名采用驼峰命名，不用下划线
- 方法和变量使用小写，单词间加下划线
- 每个类都应紧跟文档字符串，描述其功能
- 在类中使用一个空行分隔方法，在模块中使用两个空行分隔类


## python指令：
- pip list输出当前环境下已安装的包
- python -m pip install --user pygame 下载安装库（本例为pygame模糊安装，非指定版本）
	- 若想安装指定版本：pip install django=1.11.11（本例为下载django1.11.11版本）
- python -m pip uninstall pygame 卸载库
- pip show 库名 查看安装的库名信息


## python国内镜像源：
	临时改下载源指令：
		python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple


	豆瓣：http://pypi.douban.com/simple/
	
	清华：https://pypi.tuna.tsinghua.edu.cn/simple
	
	阿里云：http://mirrors.aliyun.com/pypi/simple/
	
	中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
	
	华中理工大学：http://pypi.hustunique.com/
	
	山东理工大学：http://pypi.sdutlinux.org/ 
	
## 虚拟环境：虚拟环境下载库或是执行命令，仍然需要到Scripts文件夹内，并且前面需要加*python -m*
## open一个同py文件同一个目录的文件的时候，用以下：
	../ 表示当前文件所在的目录的上一级目录
	./ 表示当前文件所在的目录(可以省略)
	/ 表示当前站点的根目录(域名映射的硬盘目录)