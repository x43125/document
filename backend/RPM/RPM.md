# RPM调研与使用

## RPM 软件包管理器

英文释义：Red-Hat Package Manager / RPM Package Manager递归缩写 

一种用于互联网下载包的打包及安装工具，将文件打包成带有 .RPM扩展名的文件，用于安装。原先是Red Hat Linux发行版专门用来管理Linux各项套件的程序。但由于其支持[GPL](https://baike.baidu.com/item/GPL/2357903?fr=aladdin)(通用公共许可证)，使用起来非常方便，因此广受欢迎，也就逐渐被其他发行版所采用，成为了一种通用的下载、安装工具。

## 命令

.RPM文件的安装指令如下

```bash
rpm -i *.rpm
```

## 打包

文件夹

| 默认位置             | 宏代码         | 名称              | 用途                                         |
| :------------------- | :------------- | :---------------- | :------------------------------------------- |
| ~/rpmbuild/SPECS     | %_specdir      | Spec 文件目录     | 保存 RPM 包配置（.spec）文件                 |
| ~/rpmbuild/SOURCES   | %_sourcedir    | 源代码目录        | 保存源码包（如 .tar 包）和所有 patch 补丁    |
| ~/rpmbuild/BUILD     | %_builddir     | 构建目录          | 源码包被解压至此，并在该目录的子目录完成编译 |
| ~/rpmbuild/BUILDROOT | %_buildrootdir | 最终安装目录      | 保存 %install 阶段安装的文件                 |
| ~/rpmbuild/RPMS      | %_rpmdir       | 标准 RPM 包目录   | 生成/保存二进制 RPM 包                       |
| ~/rpmbuild/SRPMS     | %_srcrpmdir    | 源代码 RPM 包目录 | 生成/保存源码 RPM 包(SRPM)                   |

## SPEC

| 参数          | 描述                                                         |
| :------------ | ------------------------------------------------------------ |
| Name          | 软件包名                                                     |
| Version       | 软件版本                                                     |
| Release       | 软件发行版本号                                               |
| Summary       | 简介                                                         |
| Group         | 软件包所属类别：Development/System(开发/系统)；System Environment/Daemon是（系统环境/守护） |
| License       | 软件授权协议，通常是GPL                                      |
| URL           | 软件的主页                                                   |
| Source0       | 源程序软件包的名字                                           |
| BuildRequires | 制作过程中用到的软件包，构建依赖                             |
| Requires      | 安装时所需软件包                                             |
| %description  | 描述                                                         |
| %prep         | 预处理，通常用来执行一些解开源程序包的命令，为下一步的编译安装作准备 |
| %build        | 构建」阶段，这个阶段会在 `%_builddir` 目录下执行源码包的编译。一般是执行执行常见的 `configure` 和 `make` 操作 |
| %install      | 「安装」阶段，就是执行 `make install` 命令操作。开始把软件安装到虚拟的根目录中。这个阶段会在 `%buildrootdir` 目录里建好目录结构，然后将需要打包到 rpm 软件包里的文件从 `%builddir` 里拷贝到 `%_buildrootdir` 里对应的目录里。 |
| %files        | 本段是文件段，主要用来说明会将 `%{buildroot}` 目录下的哪些文件和目录最终打包到rpm包里。定义软件包所包含的文件，分为三类：说明文档（doc） 配置文件（config） 执行程序 |
| %changelog    | 本段是修改日志段，记录 spec 的修改日志段。你可以将软件的每次修改记录到这里，保存到发布的软件包中，以便查询之用。 |

