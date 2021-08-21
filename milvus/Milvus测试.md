# Milvus测试

## 1.Milvus安装

- Milvus0.10.0 CPU版安装参考：https://milvus.io/cn/docs/v0.10.0/cpu_milvus_docker.md

- 安装Python SDK

```shell
pip install pymilvus==0.2.13
```

- 参数配置

将 `server_config.yaml` 中的参数 CPU: cache_size 设置为 40 GB。该值需要根据库中的数据量大小修改，本次测试中设置为 40GB。其余参数为默认值。

## 2.生成数据

执行以下命令随机生成1024维bit型向量。

```shell
mkdir data
python generate_random_data.py
```

> 上述脚本会在 data 目录下生成 800 个 .npy 文件，每个文件里有 10 万条向量。

## 3.2000w数据测试

本测试通过汉明距离来计算向量间的相似度。

1.使用脚本`milvud_tollkits.py`创建一个名为`test_2000w`集合

```shell
python milvud_tollkits.py --collection_name test_2000w --create
```

2.使用脚本`load_data_to_milvus.py`将2000万条向量导入集合`test_2000w`中

```shell
python load_data_to_milvus.py -n 200 --collection_name test_2000w --load
```

> 导入2000w数据共耗时107秒。
>
> -n 200 指的是从data目录中加载 200 个文件，一共 2000 万向量。

3.性能测试

```shell
python milvud_tollkits.py --collection_name test_2000w --search
```

> 查询时间为 86 ms.

## 3.4000w数据测试

本测试通过汉明距离来计算向量间的相似度。

1.使用脚本`milvud_tollkits.py`创建一个名为`test_4000w`集合

```shell
python milvud_tollkits.py --collection_name test_4000w --create
```

2.使用脚本`load_data_to_milvus.py`将2000万条向量导入集合`test_4000w`中

```shell
python load_data_to_milvus.py -n 200 -c test_4000w --load
```

> 导入4000w数据共耗时217秒。

3.性能测试

```shell
python milvud_tollkits.py --collection_name test_4000w --search
```

> 查询时间为 167 ms.



## 3.8000w数据测试

本测试通过汉明距离来计算向量间的相似度。

1.使用脚本`milvud_tollkits.py`创建一个名为`test_8000w`集合

```shell
python milvud_tollkits.py --collection_name test_8000w --create
```

2.使用脚本`load_data_to_milvus.py`将8000万条向量导入集合`test_8000w`中

```shell
python load_data_to_milvus.py -n 800 -c test_8000w --load
```

> 导入8000w数据共耗时409秒。

3.性能测试

```shell
python milvud_tollkits.py --collection_name test_8000w --search
```

> 查询时间为 336 ms.

## 4.性能结果

| 数据量 | 导入时间（s） | 查询时间（ms） |
| ------ | ------------- | -------------- |
| 2000万 | 107           | 86             |
| 4000万 | 217           | 167            |
| 8000万 | 409           | 336            |

上述导入时间指的是导入总数居所消耗时间。

查询时间指的是在库中查询一条向量的 top 10 所用时间。

> 注：本测试过程中仅使用 CPU 资源。