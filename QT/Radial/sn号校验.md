# SN号校验

## 一、目的

根据是否开启sn校验来校验上架单是否符合要求

## 二、功能设计

### 2.1 入库时插入sn号相关信息

- 在一系列校验之后，最后入库的时候，把sn号一起入到sku_sn, level3_inventory_each两张表里

  > sn全局唯一

### 2.2 校验sn号

- 首先判断该上架商品的sn_enabled字段是否为true，如果为true则说明需要校验sn号；
- 然后查询该上架申请参数中是否有sn号，没有则报错；
- 其次校验该请求参数的上架quantity是否为1，如果不为1则报错；
- 再校验该sn号在数据库中是否已经存在了，如果存在了是否数量为0。
  - 现根据sku_code查询sku_id
  - 根据sku_id和sn号查找到evo_wes_inventory.level3_inventory_each中的记录
    - 判断是否有该记录，没有则创建，有则判断quantity是否为0，为0则创建
    - 否则返回

综上校验通过则可以进行创建入库单。

## 三、上架功能记录

需要通过interface来请求，因为有部分功能写在了interface里

请求URL：`POST`方式

>需要在header里添加认证：
>
>KEY: `Authorization`
>
>VALUE: `QmVhcmVyIGRldg==`

```http
https://172.31.239.76:10081/api/radial/quicktron/putReplenishOrder
```

请求参数：body-JSON

```json
{
    "header": {
        "appKey": "app-key-to-be-provided",
        "appSecret": "app secret to be provided",
        "requestId": "a4ccc2ad-e098-4e71-a06e-1624f3cf25e5",
        "timestamp": "2022-11-25 09:46:17",
        "version": "2.7"
    },
    "body": {
        "transactional": true,
        "warehouseCode": "EENL01",
        "data": [
            {
                "id": "39",
                "ownerCode": "LOAVIES",
                "billNumber": "POTSTN11250-01_100000011-A",
                "billDate": "2022-11-25 09:46:16",
                "billType": "supplier",
                "remark": "",
                "createdUser": "dega",
                "containerCode": "1059",
                "isReturn": false,
                "udf1": null,
                "udf2": null,
                "udf3": null,
                "udf4": null,
                "udf5": null,
                "extendedProperties": {},
                "details": [
                    {
                        "id": "39",
                        "location": "1059-4-5-04",
                        "isReturn": false,
                        "skuCode": "20000004088927",
                        "containerCode": "1059",
                        "quantity": "100",
                        "ownerCode": "LOAVIES",
                        "packCode": "",
                        "frozenFlag": false,
                        "lotAtt01": "",
                        "lotAtt02": "",
                        "lotAtt03": "",
                        "lotAtt04": "",
                        "lotAtt05": null,
                        "lotAtt06": "",
                        "lotAtt07": "",
                        "lotAtt08": "A",
                        "lotAtt09": "",
                        "lotAtt10": "",
                        "lotAtt11": "",
                        "lotAtt12": "",
                        "extendedProperties": {
                            "inventoryTypeId": "A"
                        }
                    }
                ]
            }
        ]
    }
}
```

> 注:
>
> 对于货架而言的话
> container_code - 》 bucketCode
> location ->  bucketSlotCode
>
> 对于容器而言的话 
> container_code - 》 level1containercode
> location ->  level2containercode

- id必填且不重复；
- containerCode要求入库单和明细单相同且与当前到站的料箱号一致；

## 四、工作站登录问题：

1、首先在工作站管理里添加 `Radial在线直接上架`

2、在工作中选择某一功能报错：

```verilog
### Error updating database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException: Duplicate entry '1-TbTbTR' for key 'uidx_station       _point_pointCode'
```

是因为在station_point表里该点位已经有一个工作站了，且该工作站并不是当前选择的这个工作站（脏数据），将该工作站id换成当前选择的工作站id后刷新即可。

3、选择一功能后报错：

![image-20221201175929343](resources/image-20221201175929343.png)

则是工作站的udf1未设置，可以将station表的udf1字段设置成 `G2P`或者`W2P`来指示当前工作站的工作类型

4、选择常规收货后报错：

```verilog
### Error querying database. Cause: java.sql.SQLException: sql injection violation, syntax error: ERROR. pos 142, line 1, column 141, token RPAREN : SELECT id,container_slot_type_code,container_slot_type_name,length,width,depth,weight FROM evo_basic.basic_container_slot_type WHERE id IN ( ) 
### The error may exist in com/kc/evo/wes/basic/a53027/mapper/A53027BasicContainerSlotTypeMapper.java (best guess) ### The error may involve com.kc.evo.wes.basic.a53027.mapper.A53027BasicContainerSlotTypeMapper.selectBatchIds 
### The error occurred while executing a query 
### SQL: SELECT id,container_slot_type_code,container_slot_type_name,length,width,depth,weight FROM evo_basic.basic_container_slot_type WHERE id IN ( ) 
### Cause: java.sql.SQLException: sql injection violation, syntax error: ERROR. pos 142, line 1, column 141, token RPAREN : SELECT id,container_slot_type_code,container_slot_type_name,length,width,depth,weight FROM evo_basic.basic_container_slot_type WHERE id IN ( ) ; uncategorized SQLException; SQL state [null]; error code [0]; sql injection violation, syntax error: ERROR. pos 142, line 1, column 141, token RPAREN : SELECT id,container_slot_type_code,container_slot_type_name,length,width,depth,weight FROM evo_basic.basic_container_slot_type WHERE id IN ( ); nested exception is java.sql.SQLException: sql injection violation, syntax error: ERROR. pos 142, line 1, column 141, token RPAREN : SELECT id,container_slot_type_code,container_slot_type_name,length,width,depth,weight FROM evo_basic.basic_container_slot_type WHERE id IN ( )


### Error querying database.  Cause: java.sql.SQLException: sql injection violation, syntax error: syntax error, error in :'each null limit 0 , 2147483647', expect NULL, actual NULL pos 66, line 2, column 54, token NULL : SELECT *
        from evo_wes_inventory.level3_inventory_each null limit 0 , 2147483647
### The error may exist in URL [jar:file:/opt/evo-wes/evo-wes.jar!/BOOT-INF/lib/evo-wes-inventory-biz-2.9.1.w2p-SNAPSHOT.jar!/mapper/inventory/core/Level3InventoryEachMapper.xml]
### The error may involve com.kc.evo.wes.inventory.core.dal.Level3InventoryEachMapper.queryList
### The error occurred while executing a query
### SQL: SELECT *         from evo_wes_inventory.level3_inventory_each null limit 0 , 2147483647
### Cause: java.sql.SQLException: sql injection violation, syntax error: syntax error, error in :'each null limit 0 , 2147483647', expect NULL, actual NULL pos 66, line 2, column 54, token NULL : SELECT *
        from evo_wes_inventory.level3_inventory_each null limit 0 , 2147483647
; uncategorized SQLException; SQL state [null]; error code [0]; sql injection violation, syntax error: syntax error, error in :'each null limit 0 , 2147483647', expect NULL, actual NULL pos 66, line 2, column 54, token NULL : SELECT *
        from evo_wes_inventory.level3_inventory_each null limit 0 , 2147483647; nested exception is java.sql.SQLException: sql injection violation, syntax error: syntax error, error in :'each null limit 0 , 2147483647', expect NULL, actual NULL pos 66, line 2, column 54, token NULL : SELECT *

```

可能原因：在组装mybatis-plus的查询语句的时候没有加`and`或是多加了`and`

```Java
WesQuery query = new WesQuery();
query.addCondition("sn", OperatorEnum.EQ, detailBO.getLotAtt04());
```