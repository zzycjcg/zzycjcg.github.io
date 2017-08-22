---
title: mybatis技巧之foreach递归
date: 2017-08-21 17:45:06
tags:
        - mybatis
        - foreach
        - nested
        - 分表
        - union
---

mybatis的foreach已经用得很熟了，可如果foreach存在递归时，该怎么用？

<!-- more -->

##起因
在项目中，有个接口存在循环查询数据库的情况，需要改造成批量查询；这个批量查询比较特殊的地方是，
数据库是分表：根据主键id分表。一般的foreach都是针对单表的，典型的使用如下：
```
select
<include refid="allColumns" />
from table_demo
where id in 
<foreach collection="ids" item="it" open="(" separator="," close=")">
		#{it}
</foreach>
```

##union
针对分表的批量查询，可以想象成对多张表的批量，多张表的批量又可以使用<strong>union</strong>
将结果合并成一个结果集，批量的sql应该如下结构：
```
select <include refid="allColumns" /> from table_demo_001 where id in (1,2,3)
union
select <include refid="allColumns" /> from table_demo_002 where id in (101,201,301)
union
select <include refid="allColumns" /> from table_demo_003 where id in (1001,2001,3001);
```
这里有个问题：mysql是否对union的个数有限制？

##sqlmap nested foreach
既然明确了最终的sql结构，那么就可以根据这个目标构造sqlmap;
Java构造参数：
```
Map<String, List<Long>> tbIdsMap = new HashMap<>();
for (Long id:ids) {
    // 根据id取得分表后缀，即table_demo_001的'001'
    String tb = getTbSuffix(id);
    List<Long> ids = tbIdsMap.get(tb);
    if (ids==null) {
        ids = new ArrayList<>();
        tbIdsMap.put(tb, ids);
    }
    ids.add(cid);
}
Map<String, Object> params = new HashMap<>();
// 给对象添加一个变量名
params.put("map", tbIdsMap);
return selectList("findByIds", params);
```
生成sqlmap
```
<foreach collection="map.keys" item="key" open=" " separator="union" close=" " >
    select
    <include refid="fields" />
    from consumer_${key}
    where id in
    <foreach collection="map[key]" item="id" open="(" close=")" separator="," >
        #{id}
    </foreach>
</foreach>
```
这里有2个地方要留意：
1.key的引用方式：应该是${key}而不是#{key}，区别主要在于$输出变量内容，会去掉字符串的单引号，
但#则会保留；
2.map[key]相当于map.get("key")，是根据key拿到value

这里还有一个不太合理的地方，是遍历的方式，如果能使用map.entrySet()方式遍历，效率会更高，可惜试了很多种
情况，一直报错，只能委曲求全使用map.keys()+map.get("key")的组合方式。如果有人有更好的方式，请告知我。
