



mart_waimai.aggr_expt_shunt_uuid_exp_expand_dd    实验表     exp_scene_key实验场景；exp_id实验组；dt日期；UUID    目的 -> 取到命中了某个实验组的UUID

mart_waimai.aggr_flow_pinhf_mv_d  曝光表      attribute[‘label_type_name’] like ‘’  曝光包含某个标签（attribute[‘label_type_name’]多个标签是逗号拼在一起的）     spu_id 商品     用户     时间

mart_waimai.topic_ord_pinhf_info_d_view  订单表      时间 用户 商品 订单       o.is_pinhf_ord     o.is_arrange



需求：0，1，2，3，4        **命中了实验组**的转化率 = 订单/曝光





```sql
# create table mart_waimai.aggr_expt_shunt_uuid_exp_expand_dd
# (
#     id            bigint primary key auto_increment,
#     exp_id        bigint comment '实验组id',
#     exp_scene_key varchar(255) comment '实验场景',
#     uuid          bigint comment '用户id',
#     dt            date comment '日期'
# ) comment '实验组uuid扩展表';
#
# create table mart_waimai.aggr_flow_pinhf_mv_d
# (
#     id        bigint primary key auto_increment,
#     dt        date comment '日期',
#     uuid      bigint comment '用户id',
#     spu_id    bigint comment '商品id',
#     attribute json comment '属性'
# ) comment '曝光数据表';
#
# create table mart_waimai.topic_ord_pinhf_info_d_view
# (
#     id             bigint primary key auto_increment,
#     dt             date comment '日期',
#     wm_order_id    bigint comment '订单id',
#     uuid           bigint comment '用户id',
#     wm_food_spu_id bigint comment '商品id',
#     is_pinhf_ord   int comment '是否拼好饭订单',
#     is_arrange     int comment '是否排号'
# ) comment '订单信息表';

-- 订单
SELECT subquery.dt,
       subquery.exp_id,
       subquery.info_count,
       COUNT(DISTINCT o.wm_order_id) AS ord
FROM (
         -- 命中了实验组的uuid曝光数据
         SELECT v.dt,
                e.exp_id,
                v.spu_id,
                v.uuid,
                -- 商品标签数
                (CASE WHEN v.attribute -> '$.activity_tag_type' IN (2) THEN 1 ELSE 0 END +
                 CASE WHEN (v.attribute -> '$.label_type_name' LIKE '%PHF_MERGE_MT_DP_COLLECTION_YEAR%' OR
                            v.attribute -> '$.label_type_name' LIKE '%phf_poi_brand_tags%') THEN 1 ELSE 0 END +
                 CASE WHEN (v.attribute -> '$.label_type_name' LIKE '%phf_time_insurance%') THEN 1 ELSE 0 END +
                 CASE WHEN (v.attribute -> '$.label_type_name' LIKE '%phf_ugc%' OR
                            v.attribute -> '$.label_type_name' LIKE '%SPU_PHF_GOOD_COMMENT_NUM_1Y%' OR
                            v.attribute -> '$.label_type_name' LIKE '%PHF_SPU_GOOD_AMOUNT%' OR
                            v.attribute -> '$.label_type_name' LIKE '%SPU_PHF_REPURCHASE_USER_RATE_1Y%') THEN 1 ELSE 0 END) AS info_count
         FROM mart_waimai.aggr_expt_shunt_uuid_exp_expand_dd e
                  JOIN mart_waimai.aggr_flow_pinhf_mv_d v
                       ON e.uuid = v.uuid AND e.dt = v.dt
         WHERE e.dt BETWEEN '2025-02-09' AND '2025-02-10'  -- 实验组的数据日期
           AND e.exp_scene_key IN ('phf_product_card_clean')
           AND e.exp_id IN (2517133, 2517134, 2517135, 2517653)
     )
         AS subquery
         LEFT JOIN mart_waimai.topic_ord_pinhf_info_d_view o
    -- 命中了实验组且下过单的数据
                   ON subquery.uuid = o.uuid AND subquery.spu_id = o.wm_food_spu_id
WHERE o.is_pinhf_ord = 1
  AND o.is_arrange = 1  AND o.dt BETWEEN '2025-02-09' AND '2025-02-10'
GROUP BY subquery.dt, subquery.exp_id, subquery.info_count
ORDER BY subquery.dt,
         subquery.exp_id,
         subquery.info_count;


-- 曝光
SELECT subquery.dt,
       subquery.exp_id,
       subquery.info_count,
       COUNT(subquery.uuid) AS pv
FROM (
         SELECT e.dt,
                e.exp_id,
                v.spu_id,
                v.uuid,
                (CASE WHEN v.attribute -> '$.activity_tag_type' IN (2) THEN 1 ELSE 0 END +
                 CASE WHEN (v.attribute -> '$.label_type_name' LIKE '%PHF_MERGE_MT_DP_COLLECTION_YEAR%'
                     OR v.attribute -> '$.label_type_name' LIKE '%phf_poi_brand_tags%') THEN 1 ELSE 0 END +
                 CASE WHEN (v.attribute -> '$.label_type_name' LIKE '%phf_time_insurance%') THEN 1 ELSE 0 END +
                 CASE WHEN (v.attribute -> '$.label_type_name' LIKE '%phf_ugc%'
                     OR v.attribute -> '$.label_type_name' LIKE '%SPU_PHF_GOOD_COMMENT_NUM_1Y%'
                     OR v.attribute -> '$.label_type_name' LIKE '%PHF_SPU_GOOD_AMOUNT%'
                     OR v.attribute -> '$.label_type_name' LIKE '%SPU_PHF_REPURCHASE_USER_RATE_1Y%') THEN 1 ELSE 0 END
                    ) AS info_count
         FROM mart_waimai.aggr_expt_shunt_uuid_exp_expand_dd e
                  JOIN mart_waimai.aggr_flow_pinhf_mv_d v
                       ON e.uuid = v.uuid AND e.dt = v.dt
         WHERE e.dt BETWEEN '2025-02-09' AND '2025-02-10'
           AND e.exp_scene_key IN ('phf_product_card_clean')
           AND e.exp_id IN (2517133, 2517134, 2517135, 2517653)
     ) AS subquery
GROUP BY subquery.dt,
         subquery.exp_id,
         subquery.info_count
ORDER BY subquery.dt,
         subquery.exp_id,
         subquery.info_count;

show tables;

-- 实验组表
select * from mart_waimai.aggr_expt_shunt_uuid_exp_expand_dd;
-- 曝光数据表
select * from mart_waimai.aggr_flow_pinhf_mv_d;
-- 订单信息表
select * from mart_waimai.topic_ord_pinhf_info_d_view;
```







```sql
SELECT subquery.dt,
       subquery.exp_id,
       subquery.info_count,
       COUNT(DISTINCT o.wm_order_id) AS ord
FROM (
    SELECT e.dt,
           e.exp_id,
           v.spu_id,
           v.uuid,
           (CASE WHEN v.attribute['activity_tag_type'] IN (2) THEN 1 ELSE 0 END +
            CASE WHEN (v.attribute['label_type_name'] LIKE '%PHF_MERGE_MT_DP_COLLECTION_YEAR%' OR 
                       v.attribute['label_type_name'] LIKE '%phf_poi_brand_tags%') THEN 1 ELSE 0 END +
            CASE WHEN (v.attribute['label_type_name'] LIKE '%phf_time_insurance%') THEN 1 ELSE 0 END +
            CASE WHEN (v.attribute['label_type_name'] LIKE '%phf_ugc%' OR 
                       v.attribute['label_type_name'] LIKE '%SPU_PHF_GOOD_COMMENT_NUM_1Y%' OR 
                       v.attribute['label_type_name'] LIKE '%PHF_SPU_GOOD_AMOUNT%' OR 
                       v.attribute['label_type_name'] LIKE '%SPU_PHF_REPURCHASE_USER_RATE_1Y%') THEN 1 ELSE 0 END) AS info_count
    FROM mart_waimai.aggr_expt_shunt_uuid_exp_expand_dd e
    JOIN mart_waimai.aggr_flow_pinhf_mv_d v 
  ON e.uuid = v.uuid AND e.dt = v.dt
    WHERE e.dt BETWEEN $$begindatekey AND $$enddatekey 
      AND e.exp_scene_key IN ('phf_product_card_clean')
      AND e.exp_id IN (2517133, 2517134, 2517135, 2517653)
) 
AS subquery
LEFT JOIN mart_waimai.topic_ord_pinhf_info_d_view o 
ON subquery.uuid = o.uuid AND subquery.spu_id = o.wm_food_spu_id
WHERE o.is_pinhf_ord = 1
  AND o.is_arrange = 1  and o.dt BETWEEN $$begindatekey AND $$enddatekey 
GROUP BY subquery.dt, subquery.exp_id, subquery.info_count
ORDER BY subquery.dt,
          subquery.exp_id,
          subquery.info_count;
```

