# 题目:分组查询每组价格最高的两个元素有效元素



## 1. 和自己比较 循环查出每组大于当前价格的数量 然后取前两个。最后排序

```sql
SELECT a.* FROM mygoods a WHERE
	( SELECT count( 1 ) FROM mygoods WHERE cat_id = a.cat_id AND price > a.price ) < 2 
ORDER BY a.cat_id,a.price DESC;
```



## 2. 设置变量 先排序 再用函数给数据标行号 最后取小于2的行

```sql
set @row_num:=0;
set @pre_cat_id ='';
select * FROM (
select goods_id, cat_id, price,if(@pre_cat_id=cat_id,@row_num:=@row_num+1,@row_num:=0) as rownum,@pre_cat_id :=cat_id
from mygoods where status =1 order by cat_id,price desc) as t where rownum<2
```

##  3. mysql 8 支持 ROW_NUMBER() over  PARTITION by
```sql
select goods_id, cat_id, price,ROW_NUMBER() over(partition by cat_id order by price desc) mm from mygoods

```

-- 以上三种方式有个问题， 如果出现并排的则取值会遗漏其他相同项

## 4. mysql8还提供了rank()和dense_rank()函数。 

``` 
-- rank-跳跃排序，出现并列后会占用后续节点；如 两个并列第一 那会直接跳到第三名排在后面 第二名会丢失。
select goods_id, cat_id, price,rank() over(partition by cat_id order by price desc) tp from mygoods where tp<2
-- dense_rank-连续排序 并列情况下 不会占用后续节点 。如 两个并列第一 那第二名会紧跟其后。
select goods_id, cat_id, price,dense_rank() over(partition by cat_id order by price desc) tp from mygoods where tp<2
```



## 5. 针对第2种的优化方案

```sql
set @row_num:=0;
set @pre_cat_id ='';
set @pre_price=0;
select * FROM (
select goods_id, cat_id, price,if(@pre_cat_id=cat_id, if(@pre_price=price,@row_num,@row_num:=@row_num+1),@row_num:=0) as rownum,@pre_cat_id :=cat_id,@pre_price :=price
from mygoods  order by cat_id,price desc) as t where rownum<2
```







## 建表语句

```sql
DROP TABLE IF EXISTS `mygoods`;
CREATE TABLE `mygoods` (
  `goods_id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `cat_id` int(11) NOT NULL DEFAULT '0',
  `price` tinyint(3) NOT NULL DEFAULT '0',
  `status` tinyint(3) DEFAULT '1',
  PRIMARY KEY (`goods_id`),
  KEY `icatid` (`cat_id`)
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of mygoods
-- ----------------------------
BEGIN;
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (1, 101, 90, 0);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (2, 101, 99, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (3, 102, 98, 0);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (4, 103, 96, 0);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (5, 102, 95, 0);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (6, 102, 94, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (7, 102, 93, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (8, 103, 99, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (9, 103, 98, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (10, 103, 97, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (11, 104, 96, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (12, 104, 95, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (13, 104, 94, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (15, 101, 92, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (16, 101, 93, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (17, 101, 99, 0);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (18, 102, 99, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (19, 105, 85, 1);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (20, 105, 89, 0);
INSERT INTO `mygoods` (`goods_id`, `cat_id`, `price`, `status`) VALUES (21, 105, 99, 1);
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

