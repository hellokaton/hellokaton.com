---
layout: post
title: mysql中的百分比统计实例，round函数
categories: mysql
tags: mysql
---

```sql
select  b.year as year,b.month  as month  ,a.`typeName` as typeName ,
sum(`bj_xl`) as bj_xl,
sum(`bj_cl`) as bj_cl,
sum(`bj_xl`)+sum(`bj_cl`) as bj_all,
ROUND(sum(`bj_xl`)/(sum(`bj_xl`)+sum(`bj_cl`))*100, 1) as bj_xl_pre,
ROUND(sum(`bj_cl`)/(sum(`bj_xl`)+sum(`bj_cl`))*100, 1) as bj_cl_pre
from `phpcms_form_qysj_zfyg_tree` a ,`phpcms_form_qysj_zfyg_data` b
where a.typeId=b.`product_code`   AND `year` like '%2008%'  AND `month` like '%%'    AND `totalType` like '%粉%'
group by `product_code`  order by  typeId asc 
ROUND(X) ROUND(X,D) 
```

<!-- more -->

返回参数X, 其值接近于最近似的整数。在有两个参数的情况下，返回 X ，其值保留到小数点后D位，而第D位的保留方式为四舍五入。若要接保留X值小数点左边的D 位，可将 D 设为负值。 

```bash
mysql> SELECT ROUND(-1.23);
        -> -1
mysql> SELECT ROUND(-1.58);
        -> -2
mysql> SELECT ROUND(1.58);
        -> 2
mysql> SELECT ROUND(1.298, 1);
        -> 1.3
mysql> SELECT ROUND(1.298, 0);
        -> 1
mysql> SELECT ROUND(23.298, -1);
        -> 20
```

返回值的类型同 第一个自变量相同(假设它是一个整数、双精度数或小数)。这意味着对于一个整数参数,结果也是一个整数(无小数部分)。
当第一个参数是十进制常数时，对于准确值参数，ROUND() 使用精密数学题库：
 
对于准确值数字, ROUND() 使用“四舍五入” 或“舍入成最接近的数” 的规则:对于一个分数部分为 .5或大于 .5的值，正数则上舍入到邻近的整数值， 负数则下舍入临近的整数值。(换言之, 其舍入的方向是数轴上远离零的方向）。对于一个分数部分小于.5 的值，正数则下舍入下一个整数值，负数则下舍入邻近的整数值，而正数则上舍入邻近的整数值。 
对于近似值数字，其结果根据C 库而定。在很多系统中，这意味着 ROUND()的使用遵循“舍入成最接近的偶数”的规则： 一个带有任何小数部分的值会被舍入成最接近的偶数整数。 
以下举例说明舍入法对于精确值和近似值的不同之处： 

```bash
mysql> SELECT ROUND(2.5), ROUND(25E-1);
+------------+--------------+
| ROUND(2.5) | ROUND(25E-1) |
+------------+--------------+
| 3          |            2 |
+------------+--------------+
```
