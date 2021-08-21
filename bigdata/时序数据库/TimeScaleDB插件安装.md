# TimeScaledb插件安装

> 要安装TimeScaledb插件首先需要安装PG数据库和cmake前置依赖

## PG数据库安装

详见PG数据库安装...

## CMake编译安装

1. 首先查看下当前系统是否安装cmake，是否高于或等于当前待安装的Timescaledb插件的要求：cmake --version

2. 未安装或版本过低则编译安装

	- 到 https://cmake.org/download/ 中下载相应的版本cmake，本文使用源码文件编译方式

	- ```sh
		wget https://github.com/Kitware/CMake/releases/download/v3.20.2/cmake-3.20.2.tar.gz
		```

	- 到环境中找到安装包解压

	- ```sh
		tar -zxvf cmake-*.tar.gz
		```

	- 编译cmake

	- ```sh
		cd cmake
		./bootstrap
		```

		若有`could not find openssl`相关报错，则为缺少openssl依赖导致，安装即可

		```sh
		yum install -y openssl  openssl-devel
		```

	- 安装好重新执行`./bootstrap`即可，执行完后会有语言提示输入`gmake`

	- 这一过程会持续很长时间

	- 结束后输入 `gmake install`

	- 执行完即安装号cmake，可以使用`cmake --version` 进行验证

## Timescaledb安装

```sh
git clone git@github.com:timescale/timescaledb.git
cd timescaledb
./bootstrap

报错
CMake Error at CMakeLists.txt:208 (message):
Unable to find 'pg_config'
```

