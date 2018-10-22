# SQL 相关

- select子查询、多表关联、聚集

  ```sql
  -- 性别类型分布
  SELECT mc.fjm as c_fjm ,1 as 'c_dm',COUNT (sxr.c_bzxrmc) as v_zb FROM db_zxzh.V_ZHZX_SXBZXR sxr,db_zxzh.TS_FYMC mc
  WHERE sxr.c_SXZT <> '' and sxr.c_zxfy = mc.fydm and A .XB = '09_00003-1' GROUP BY mc.fjm
  union all
  SELECT mc.fjm as c_fjm ,2 as 'c_dm',COUNT (sxr.c_bzxrmc) as v_zbFROM db_zxzh.V_ZHZX_SXBZXR sxr,db_zxzh.TS_FYMC mc
  WHERE sxr.c_SXZT <> '' and sxr.c_zxfy = mc.fydm and A .XB = '09_00003-2' GROUP BY mc.fjm;
  ```

  ``Union``，对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；select * 默认按照主键排序，也可以自定义排序需要注意的是，排序只能写在最后一个sql.

  ``Union All``，对两个结果集进行并集操作，包括重复行，不进行排序；

- 程序SQL字段为大写，但是Java查询结果返回小写

  ````sql
  Select corpId,c_ah from db_aj;
  ````

  sql 中的corpId,其中的I为大写，但是返回结果是cropid,此时我们在查询字段时加上双引号即可；

  ````sql
  Select corpId as "corpId",c_ah from db_aj;
  ````

- count( * )问题、去除多余的外层嵌套count( * )

  ````sql
  select count(*) from( 	 SELECT aj.C_BH,aj.C_AH,aj.C_BQJG,aj.N_ZZM,aj.N_XFBGLX,aj.N_CBR,aj.C_ZMZS,aj.C_DSR,aj.D_LARQ,aj.DT_FASJ,aj.D_JARQ,aj.D_SXRQ,aj.N_SDRS,aj.C_PC,aj.N_JBFY,aj.C_BH_LCSL FROM T_XS_AJ aj WHERE aj.N_XFBGLX in (1,2) AND aj.N_SPCX = 12 AND aj.N_CBR = 157286789 AND aj.DT_FASJ is not null AND aj.N_AJJZJD = 10) t_19b4ae3444
  ````

  外层count(*)毫无用处

- 应使用select count(*)而非select count(col)


  ````sql
  1、abase执行计划：count(*)、count(1)是选取了一个整型索引字段进行查询的。
  2、sybase执行计划：count(*)、count(1)、count(主键)是选取了一个整型索引字段进行查询的。
  3、abase执行效率：count(*)>=count(1)>=count(整型索引字段)>count(字符索引字段)>count(字符主键)>count(整型非索引字段)
  4、sybase执行效率：count(*)=count(1)=count(整型主键)>count(整型索引字段)>count(字符索引字段)>count(整型非索引字段)
  5、abase与sybase的count() 括号里面都是是判断是否为空的，空则不参与计算（李贵阳曰）
  6、非特殊场景查询，统一要求使用count(*)
  ````

  [内网帖子](http://home.thunisoft.com:8082/forum.php?mod=viewthread&tid=51889?_blank)

  使用count( * )>=count(1)>=count(c_bh)，count(1)会转为count( * ).

- 避免隐含的数据类型转换

  ````sql
  string类型传入了int类型或者numberic和int类型相加
  SELECT COUNT(*),n_scs,c_ss_wg FROM wgxt.t_sc_sx WHERE c_sfyx=1 GROUP BY c_ss_wg;
  c_sfyx为string类型
  ````

- 禁止对abase数据库同一字段建立多个相同类型的索引

  ````sql
  Indexes:
  "t_zhld_db_pkey" PRIMARY KEY, btree (c_bh)
   "i_zhld_db_type" btree (n_type)
   "i_zhld_db_type1" btree (n_type)
  ````

- 常见SQL优化

  - 1、abase分析表字段为单值代码，类型分为几类，查询是占用数据库全部数据的1/3，1/2不走索引。

  - 2、日期查询>=,=<如果范围过大，也不会走索引，所以要考虑到查询大量数据是否对客户有用，尽量缩小范围。 

  - 3、两张表联查的时候，喜欢把from t1,t2 where t2.xx=? and …,这种查询，可尝试写成：

    From t1 join (select xx from t2 where t2.xx=? and …) 这样如果根据条件查询出t2的结果集可能就很小了。另外查看执行计划第二种写法访问的平均宽度要小。

  - 4、不要再条件中使用函数运算，where sal > 2000/12,to_char(zbaj.d_jarq,'yyyyMMdd')<='20170322'这样不会走索引，如果要使用索引，得建立函数索引,但是不建议。

  - in 操作，建议in里面的的值在100以内。

- 可以使用创建临时表做关联查询

  ````sql
  1.EXPLAIN SELECT * FROM X WHERE x_num IN(SELECT y_num FROM y);  
  2.                              QUERY PLAN                                
  3.----------------------------------------------------------------------  
  4. Hash Join  (cost=23.25..49.88 rows=350 width=86)  
  5.   Hash Cond: (x.x_num = y.y_num)  
  6.   ->  Seq Scan on x  (cost=0.00..17.00 rows=700 width=86)  
  7.   ->  Hash  (cost=20.75..20.75 rows=200 width=4)  
  8.         ->  HashAggregate  (cost=18.75..20.75 rows=200 width=4)  
  9.               ->  Seq Scan on y  (cost=0.00..17.00 rows=700 width=4)  
  ````

- 合理使用like

  like再前匹配和后匹配的时候会走索引，但是全模糊匹配的时候不会走索引，在abase中可以创建gin和gist索引来走索引。

  ````sql
  SELECT * FROM T1 WHERE NAME LIKE '%L%';
  SELECT * FROM T1 WHERE NAME LIKE 'L%;
  SELECT * FROM T1 WHERE NAME LIKE '%L';
  ````

- 预防sql中存在嵌套循环

  更改前：更新757726条数据耗时3h。

  ````sql
  update db_zxzhld_bak.t_zhld_zbajxx set d_larq = 
  (select larq from db_zxzhld_bak.cacheTable where db_zxzhld_bak.t_zhld_zbajxx.c_ajbh = db_zxzhld_bak.cacheTable.c_ajbh) where c_zblx in ('2001','2002');
  ````

  更改后：耗时6.5s

  ````sql
  update db_zxzhld_bak.t_zhld_zbajxx t1 set d_larq = t.larq from db_zxzhld_bak.cacheTable t where t.c_ajbh = t1.c_ajbh AND t1.c_zblx in ('2001','2002');
  ````

  第一个sql查询其实是一个循环查询，特别耗时，类似于:

  ````sql
  select c_bh,(select d_larq from t2) as larq from t1
  ````

  嵌套循环。

- 如果表中有需要查询user,dept的关联去掉，从内存中去查询。

  ````java
  ArteryOrganUtil的相关方法
  ````

- or条件可以改写成union

  ````sql
  SELECT * FROM t1 WHERE (t1.xx=104  AND t1.yy>1001) OR 
  T1.zz=1008
  ````

  更改后

  ````sql
  SELECT * FROM t1 WHERE t1.xx=104 AND t1.yy>1001 
  UNION 
  SELECT * FROM t1 WHERE t1.zz=1008 
  ````

  Abase使用or也可以走索引、但是需要做bitmapOr操作、效率比union all低一点。

- 能用UNION ALL就不要用UNION

  UNION ALL不执行SELECT DISTINCT函数，这样就会减少很多不必要的资源

- 可以使用int的字段，就不要用VARCHAR

- 择优选择使用exists和in，join

  ````sql
  /* 语句一 */
  SELECT a, name FROM table1 WHERE EXISTS (SELECT 1 FROM table2 WHERE table1.a = table2.a);
  /* 语句二 */
  SELECT a, name FROM table1 WHERE table1.a IN (SELECT table2.a FROM table2);
  ````

  一般情况下，table1数量小二table2数据量非常大时，语句一的查询效率高、table1数量非常大而table2数据量小时，语句二的查询效率高。
    对于not exists查询，外表条件字段存在空值，存在空值的那条记录最终会输出；对于not in、外表条件字段存在空值、存在空值的那条记录最终将被过滤、其他数据不受影响。

- 禁止使用not in语句、建议使用not exists

  ````sql
  not exists查询，内表条件字段存在空值对查询结果没有影响、对于not in查询、内表条件字段存在空值将会导致最终的查询结果为空。
  ````

- 使用order by和gourp by排序操作，建议在列上建立索引

  - 避免count(distinct xx)	

  更改前:耗时，0.15s

  ````sql
  SELECT count(DISTINCT c_ah) FROM db_gzlpg.t_fb_ptaj_3000
  ````

  更改后：耗时，0.004s

  ````sql
  SELECT count(*) FROM (select DISTINCT c_ah from db_gzlpg.t_fb_ptaj_3000) tt
  ````

  效率提升37.5倍。

- 数字类型原则上只有integer和numeric(20,4)

- 删除全部表数据，应该使用truncate 语句

  ​

### 查看SQL执行计划，优化SQL

- 前言

  一个顺序磁盘页面操作的cost值由系统参数seq_page_cost (floating point)参数指定的，由于这个参数默认为1.0，所以我们可以认为一次顺序磁盘页面操作的cost值为1。

  示例SQL,优化前执行时间，27.088s

  ````sql
  SELECT
  	aj.n_ajbs,
  	aj.d_jarq,
  	aj.n_cbr,
  	dsr.c_xlabs
  FROM
  	(
  		SELECT
  			n_ajbs,
  			d_jarq,
  			n_cbr,
  			n_ssdw1,
  			n_ay,
  			string_agg (c_mc, ';') AS c_dsr
  		FROM
  			(
  				SELECT
  					dsr.n_ajbs,
  					aj.d_jarq,
  					cbrml.n_bs AS n_cbr,
  					n_ssdw1,
  					aj.n_ay,
  					ml.c_mc
  				FROM
  					db_xzys.t_xzys aj
  				LEFT JOIN db_xzys.t_xzysyastml cbrml ON aj.n_cbr = cbrml.n_xh
  				AND aj.n_ajbs = cbrml.n_ajbs
  				LEFT JOIN db_xzys.t_xzysdsr dsr ON aj.n_ajbs = dsr.n_ajbs
  				LEFT JOIN db_xzys.t_xzysyastml ml ON dsr.n_xh = ml.n_xh
  				AND dsr.n_ajbs = ml.n_ajbs
  				WHERE
  					AJ.N_XJAFS IN (2, 5, 13)
  				ORDER BY
  					dsr.n_ajbs,
  					aj.d_jarq,
  					ml.n_bs,
  					n_ssdw1,
  					ml.c_mc
  			) dsr
  		GROUP BY
  			dsr.n_ajbs,
  			d_jarq,
  			n_cbr,
  			n_ay,
  			dsr.n_ssdw1
  	) aj,
  	(
  		SELECT
  			n_ssdw1,
  			n_ay,
  			n_cbr,
  			c_dsr,
  			REPLACE (
  				CAST (
  					db_jxglpt.uuid_generate_v1 () AS VARCHAR
  				),
  				'-',
  				''
  			) AS c_xlabs
  		FROM
  			(
  				SELECT
  					n_ajbs,
  					n_ssdw1,
  					n_cbr,
  					n_ay,
  					string_agg (c_mc, ';') AS c_dsr
  				FROM
  					(
  						SELECT
  							dsr.n_ajbs,
  							n_ssdw1,
  							cbrml.n_bs AS n_cbr,
  							aj.n_ay,
  							ml.c_mc
  						FROM
  							db_xzys.t_xzys aj
  						LEFT JOIN db_xzys.t_xzysyastml cbrml ON aj.n_cbr = cbrml.n_xh
  						AND aj.n_ajbs = cbrml.n_ajbs
  						LEFT JOIN db_xzys.t_xzysdsr dsr ON aj.n_ajbs = dsr.n_ajbs
  						LEFT JOIN db_xzys.t_xzysyastml ml ON dsr.n_xh = ml.n_xh
  						AND dsr.n_ajbs = ml.n_ajbs
  						ORDER BY
  							dsr.n_ajbs,
  							n_ssdw1,
  							ml.c_mc
  					) dsr
  				GROUP BY
  					dsr.n_ajbs,
  					n_cbr,
  					n_ay,
  					dsr.n_ssdw1
  			) dsr
  		GROUP BY
  			n_ay,
  			n_ssdw1,
  			n_cbr,
  			c_dsr
  		HAVING
  			COUNT (1) > 1
  	) dsr
  WHERE
  	aj.n_ssdw1 = dsr.n_ssdw1
  AND aj.c_dsr = dsr.c_dsr
  AND aj.n_cbr = dsr.n_cbr
  AND aj.n_ay = dsr.n_ay
  GROUP BY
  	aj.n_ajbs,
  	aj.d_jarq,
  	aj.n_cbr,
  	dsr.c_xlabs;
  ````

  - 首先分析该SQL，是由两个大的子查询组成的

    先看第一个子查询的执行计划,sql执行时间0.226s

    ````sql
    EXPLAIN
    SELECT
    			n_ajbs,
    			d_jarq,
    			n_cbr,
    			n_ssdw1,
    			n_ay,
    			string_agg (c_mc, ';') AS c_dsr
    		FROM
    			(
    				SELECT
    					dsr.n_ajbs,
    					aj.d_jarq,
    					cbrml.n_bs AS n_cbr,
    					n_ssdw1,
    					aj.n_ay,
    					ml.c_mc
    				FROM
    					db_xzys.t_xzys aj
    				LEFT JOIN db_xzys.t_xzysyastml cbrml ON aj.n_cbr = cbrml.n_xh
    				AND aj.n_ajbs = cbrml.n_ajbs
    				LEFT JOIN db_xzys.t_xzysdsr dsr ON aj.n_ajbs = dsr.n_ajbs
    				LEFT JOIN db_xzys.t_xzysyastml ml ON dsr.n_xh = ml.n_xh
    				AND dsr.n_ajbs = ml.n_ajbs
    				WHERE
    					AJ.N_XJAFS IN (2, 5, 13)
    				ORDER BY
    					dsr.n_ajbs,
    					aj.d_jarq,
    					ml.n_bs,
    					n_ssdw1,
    					ml.c_mc
    			) dsr
    		GROUP BY
    			dsr.n_ajbs,
    			d_jarq,
    			n_cbr,
    			n_ay,
    			dsr.n_ssdw1
    ````

    执行计划：

    ````sql
    GroupAggregate  (cost=72061.66..72610.59 rows=6757 width=63)
      Group Key: dsr.n_ajbs, dsr.d_jarq, dsr.n_cbr, dsr.n_ay, dsr.n_ssdw1
      ->  Sort  (cost=72061.66..72128.01 rows=26541 width=63)
            Sort Key: dsr.n_ajbs, dsr.d_jarq, dsr.n_cbr, dsr.n_ay, dsr.n_ssdw1
            ->  Subquery Scan on dsr  (cost=69779.67..70111.44 rows=26541 width=63)
                  ->  Sort  (cost=69779.67..69846.03 rows=26541 width=67)
                        Sort Key: dsr_1.n_ajbs, aj.d_jarq, ml.n_bs, dsr_1.n_ssdw1, ml.c_mc
                        ->  Nested Loop Left Join  (cost=39252.37..67829.45 rows=26541 width=67)
                              ->  Hash Right Join  (cost=39251.95..41742.69 rows=9826 width=34)
                                    Hash Cond: (dsr_1.n_ajbs = aj.n_ajbs)
                                    ->  Seq Scan on t_xzysdsr dsr_1  (cost=0.00..2097.53 rows=78653 width=18)
                                    ->  Hash  (cost=39216.84..39216.84 rows=2809 width=26)
                                          ->  Hash Right Join  (cost=1448.32..39216.84 rows=2809 width=26)
                                                Hash Cond: ((cbrml.n_xh = aj.n_cbr) AND (cbrml.n_ajbs = aj.n_ajbs))
                                                ->  Seq Scan on t_xzysyastml cbrml  (cost=0.00..36007.18 rows=231618 width=18)
                                                ->  Hash  (cost=1406.18..1406.18 rows=2809 width=26)
                                                      ->  Seq Scan on t_xzys aj  (cost=0.00..1406.18 rows=2809 width=26)
                                                            Filter: (n_xjafs = ANY ('{2,5,13}'::integer[]))
                              ->  Index Scan using i_xzysyastml_ajbs on t_xzysyastml ml  (cost=0.42..2.64 rows=1 width=51)
                                    Index Cond: (dsr_1.n_ajbs = n_ajbs)
                                    Filter: (dsr_1.n_xh = n_xh)
    ````

    - 解读执行计划

      1.从下往上读
      2.explain报告查询的操作，开启的消耗，查询总的消耗，访问的行数 访问的平均宽度
      3.开启时间消耗是输出开始前的时间例如排序的时间
      4.消耗包括磁盘检索页，cpu时间 
      5.注意，每一步的cost包括上一步的
      6.重要的是，explain 不是真正的执行一次查询 只是得到查询执行的计划和估计的花费

      ps:索引有用条件 当满足特定条件的元组数小于总的数目

    cost=说明：

    - 第一个数字72061.66表示启动cost，这是执行到返回第一行时需要的cost值。
    - 第二个数字72610.59表示执行整个SQL的cost

    rows=6757 是实际的行数

    | 执行计划运算类型          |            操作说明            | 是否有启动时间 |
    | :---------------- | :------------------------: | :-----: |
    | Seq Scan          |            扫描表             |  无启动时间  |
    | Index Scan        |            索引扫描            |  无启动时间  |
    | Bitmap Index Scan |            索引扫描            |  有启动时间  |
    | Bitmap Heap Scan  |            索引扫描            |  有启动时间  |
    | Subquery Scan     |            子查询             |  无启动时间  |
    | Tid Scan          |         ctid = …条件         |  无启动时间  |
    | Function Scan     |            函数扫描            |  无启动时间  |
    | Nested Loop       |            循环结合            |  无启动时间  |
    | Merge Join        |            合并结合            |  有启动时间  |
    | Hash Join         |            哈希结合            |  有启动时间  |
    | Sort              |       排序，ORDER BY操作        |  有启动时间  |
    | Hash              |            哈希运算            |  有启动时间  |
    | Result            |        函数扫描，和具体的表无关        |  无启动时间  |
    | Unique            |      DISTINCT，UNION操作      |  有启动时间  |
    | Limit             |       LIMIT，OFFSET操作       |  有启动时间  |
    | Aggregate         | count, sum,avg, stddev集约函数 |  有启动时间  |
    | Group             |        GROUP BY分组操作        |  有启动时间  |
    | Append            |          UNION操作           |  无启动时间  |
    | Materialize       |            子查询             |  有启动时间  |
    | SetOp             |      INTERCECT，EXCEPT      |  有启动时   |

从上面的执行计划可看出db_xzys.t_xzysdsr 这个表走的全表扫描，rows:78653,查询该表的总数据量为78653，耗时0.028s，所以这里虽然是全表也不是很耗时；

最后的条件

```sql
WHERE AJ.N_XJAFS IN (2, 5, 13)
```

这里的查询走了索引

````sql
Index Scan using i_xzysyastml_ajbs on t_xzysyastml ml  (cost=0.42..2.64 rows=1 width=51)
````

所有查询整个语句总耗时：1.431秒

查看第二个子查询：耗时，

执行计划如下：

````sql
HashAggregate  (cost=178812.16..178933.87 rows=5409 width=44)
  Group Key: aj.n_ay, dsr.n_ssdw1, cbrml.n_bs, string_agg((ml.c_mc)::text, ';'::text)
  Filter: (count(1) > 1)
  ->  GroupAggregate  (cost=173732.33..177595.16 rows=54089 width=55)
        Group Key: dsr.n_ajbs, cbrml.n_bs, aj.n_ay, dsr.n_ssdw1
        ->  Sort  (cost=173732.33..174263.45 rows=212448 width=55)
              Sort Key: dsr.n_ajbs, cbrml.n_bs, aj.n_ay, dsr.n_ssdw1
              ->  Sort  (cost=145016.03..145547.15 rows=212448 width=55)
                    Sort Key: dsr.n_ajbs, dsr.n_ssdw1, ml.c_mc
                    ->  Hash Right Join  (cost=112143.60..118955.34 rows=212448 width=55)
                          Hash Cond: (dsr.n_ajbs = aj.n_ajbs)
                          ->  Merge Left Join  (cost=72265.23..76155.81 rows=212448 width=47)
                                Merge Cond: ((dsr.n_xh = ml.n_xh) AND (dsr.n_ajbs = ml.n_ajbs))
                                ->  Sort  (cost=8493.28..8689.92 rows=78653 width=18)
                                      Sort Key: dsr.n_xh, dsr.n_ajbs
                                      ->  Seq Scan on t_xzysdsr dsr  (cost=0.00..2097.53 rows=78653 width=18)
                                ->  Materialize  (cost=63771.95..64930.04 rows=231618 width=47)
                                      ->  Sort  (cost=63771.95..64351.00 rows=231618 width=47)
                                            Sort Key: ml.n_xh, ml.n_ajbs
                                            ->  Seq Scan on t_xzysyastml ml  (cost=0.00..36007.18 rows=231618 width=47)
                          ->  Hash  (cost=39597.29..39597.29 rows=22486 width=18)
                                ->  Hash Right Join  (cost=1659.15..39597.29 rows=22486 width=18)
                                      Hash Cond: ((cbrml.n_xh = aj.n_cbr) AND (cbrml.n_ajbs = aj.n_ajbs))
                                      ->  Seq Scan on t_xzysyastml cbrml  (cost=0.00..36007.18 rows=231618 width=18)
                                      ->  Hash  (cost=1321.86..1321.86 rows=22486 width=18)
                                            ->  Seq Scan on t_xzys aj  (cost=0.00..1321.86 rows=22486 width=18)
````

可以对比看出，第二个查询开销17万之多，大于第一个自查询的7万，两倍还多，总耗时2.8s

查看执行计划，该查询未走索引，因为里面嵌套的子查询中没有条件限制，倒是查询的数据是全量的

加了案件判断的条件之后，整体SQL查询时间耗时：3.753s

- 继续优化，因为"db_xzys"."t_xzys" 的 N_XJAFS 有过滤操作，考虑在这个字段加索引，查询表结构发现该表的这个字段不存在索引

  ````sql
  CREATE INDEX "i_mses_xjafs" ON "db_xzys"."t_xzys" USING btree ("n_xjafs");
  ````

  添加索引

- 查询结果耗时，2.686s

- 因为count(1)在查询是有会自动转成count(*)这会耗掉一定的时间我们改count(1)-->为count( * )

  查询结果耗时：2.669s 缩短了0.023s

- 从27.088s优化到2.669s提升10倍,优化后的SQL

  ````sql
  SELECT
  	aj.n_ajbs,
  	aj.d_jarq,
  	aj.n_cbr,
  	dsr.c_xlabs
  FROM
  	(
  		SELECT
  			n_ajbs,
  			d_jarq,
  			n_cbr,
  			n_ssdw1,
  			n_ay,
  			string_agg (c_mc, ';') AS c_dsr
  		FROM
  			(
  				SELECT
  					dsr.n_ajbs,
  					aj.d_jarq,
  					cbrml.n_bs AS n_cbr,
  					n_ssdw1,
  					aj.n_ay,
  					ml.c_mc
  				FROM
  					db_xzys.t_xzys aj
  				LEFT JOIN db_xzys.t_xzysyastml cbrml ON aj.n_cbr = cbrml.n_xh
  				AND aj.n_ajbs = cbrml.n_ajbs
  				LEFT JOIN db_xzys.t_xzysdsr dsr ON aj.n_ajbs = dsr.n_ajbs
  				LEFT JOIN db_xzys.t_xzysyastml ml ON dsr.n_xh = ml.n_xh
  				AND dsr.n_ajbs = ml.n_ajbs
  				WHERE
  					AJ.N_XJAFS IN (2, 5, 13)
  				ORDER BY
  					dsr.n_ajbs,
  					aj.d_jarq,
  					ml.n_bs,
  					n_ssdw1,
  					ml.c_mc
  			) dsr
  		GROUP BY
  			dsr.n_ajbs,
  			d_jarq,
  			n_cbr,
  			n_ay,
  			dsr.n_ssdw1
  	) aj,
  	(
  		SELECT
  			n_ssdw1,
  			n_ay,
  			n_cbr,
  			c_dsr,
  			REPLACE (
  				CAST (
  					db_jxglpt.uuid_generate_v1 () AS VARCHAR
  				),
  				'-',
  				''
  			) AS c_xlabs
  		FROM
  			(
  				SELECT
  					n_ajbs,
  					n_ssdw1,
  					n_cbr,
  					n_ay,
  					string_agg (c_mc, ';') AS c_dsr
  				FROM
  					(
  						SELECT
  							dsr.n_ajbs,
  							n_ssdw1,
  							cbrml.n_bs AS n_cbr,
  							aj.n_ay,
  							ml.c_mc
  						FROM
  							db_xzys.t_xzys aj
  						LEFT JOIN db_xzys.t_xzysyastml cbrml ON aj.n_cbr = cbrml.n_xh
  						AND aj.n_ajbs = cbrml.n_ajbs
  						LEFT JOIN db_xzys.t_xzysdsr dsr ON aj.n_ajbs = dsr.n_ajbs
  						LEFT JOIN db_xzys.t_xzysyastml ml ON dsr.n_xh = ml.n_xh
  						AND dsr.n_ajbs = ml.n_ajbs
  						WHERE
  							AJ.N_XJAFS IN (2, 5, 13)
  						ORDER BY
  							dsr.n_ajbs,
  							n_ssdw1,
  							ml.c_mc
  					) dsr
  				GROUP BY
  					dsr.n_ajbs,
  					n_cbr,
  					n_ay,
  					dsr.n_ssdw1
  			) dsr
  		GROUP BY
  			n_ay,
  			n_ssdw1,
  			n_cbr,
  			c_dsr
  		HAVING
  			COUNT (*) > 1
  	) dsr
  WHERE
  	aj.n_ssdw1 = dsr.n_ssdw1
  AND aj.c_dsr = dsr.c_dsr
  AND aj.n_cbr = dsr.n_cbr
  AND aj.n_ay = dsr.n_ay
  GROUP BY
  	aj.n_ajbs,
  	aj.d_jarq,
  	aj.n_cbr,
  	dsr.c_xlabs;
  ````

  执行计划：

  ````sql
  HashAggregate  (cost=33195.75..33195.76 rows=1 width=54)
    Group Key: aj.n_ajbs, aj.d_jarq, aj.n_cbr, dsr.c_xlabs
    ->  Merge Join  (cost=33123.48..33195.74 rows=1 width=54)
          Merge Cond: ((aj.n_ssdw1 = dsr.n_ssdw1) AND (aj.c_dsr = dsr.c_dsr) AND (aj.n_cbr = dsr.n_cbr) AND (aj.n_ay = dsr.n_ay))
          ->  Sort  (cost=16695.92..16709.05 rows=5254 width=62)
                Sort Key: aj.n_ssdw1, aj.c_dsr, aj.n_cbr, aj.n_ay
                ->  Subquery Scan on aj  (cost=15901.52..16371.24 rows=5254 width=62)
                      ->  GroupAggregate  (cost=15901.52..16318.70 rows=5254 width=42)
                            Group Key: dsr_1.n_ajbs, dsr_1.d_jarq, dsr_1.n_cbr, dsr_1.n_ay, dsr_1.n_ssdw1
                            ->  Sort  (cost=15901.52..15951.74 rows=20086 width=42)
                                  Sort Key: dsr_1.n_ajbs, dsr_1.d_jarq, dsr_1.n_cbr, dsr_1.n_ay, dsr_1.n_ssdw1
                                  ->  Subquery Scan on dsr_1  (cost=14214.91..14465.98 rows=20086 width=42)
                                        ->  Sort  (cost=14214.91..14265.12 rows=20086 width=46)
                                              Sort Key: dsr_2.n_ajbs, aj_1.d_jarq, ml.n_bs, dsr_2.n_ssdw1, ml.c_mc
                                              ->  Nested Loop Left Join  (cost=5846.04..12779.37 rows=20086 width=46)
                                                    ->  Hash Right Join  (cost=5845.62..7326.43 rows=5983 width=34)
                                                          Hash Cond: (dsr_2.n_ajbs = aj_1.n_ajbs)
                                                          ->  Seq Scan on t_xzysdsr dsr_2  (cost=0.00..1245.62 rows=46762 width=18)
                                                          ->  Hash  (cost=5825.39..5825.39 rows=1619 width=26)
                                                                ->  Hash Right Join  (cost=804.26..5825.39 rows=1619 width=26)
                                                                      Hash Cond: ((cbrml.n_xh = aj_1.n_cbr) AND (cbrml.n_ajbs = aj_1.n_ajbs))
                                                                      ->  Seq Scan on t_xzysyastml cbrml  (cost=0.00..4078.11 rows=123911 width=19)
                                                                      ->  Hash  (cost=779.98..779.98 rows=1619 width=26)
                                                                            ->  Seq Scan on t_xzys aj_1  (cost=0.00..779.98 rows=1619 width=26)
                                                                                  Filter: (n_xjafs = ANY ('{2,5,13}'::integer[]))
                                                    ->  Index Scan using i_xzysyastml_ajbs on t_xzysyastml ml  (cost=0.42..0.90 rows=1 width=31)
                                                          Index Cond: (dsr_2.n_ajbs = n_ajbs)
                                                          Filter: (dsr_2.n_xh = n_xh)
          ->  Sort  (cost=16427.57..16428.88 rows=526 width=76)
                Sort Key: dsr.n_ssdw1, dsr.c_dsr, dsr.n_cbr, dsr.n_ay
                ->  Subquery Scan on dsr  (cost=16386.70..16403.80 rows=526 width=76)
                      ->  HashAggregate  (cost=16386.70..16398.54 rows=526 width=44)
                            Group Key: aj_2.n_ay, dsr_3.n_ssdw1, cbrml_1.n_bs, string_agg((ml_1.c_mc)::text, ';'::text)
                            Filter: (count(*) > 1)
                            ->  GroupAggregate  (cost=15901.52..16268.49 rows=5254 width=34)
                                  Group Key: dsr_3.n_ajbs, cbrml_1.n_bs, aj_2.n_ay, dsr_3.n_ssdw1
                                  ->  Sort  (cost=15901.52..15951.74 rows=20086 width=34)
                                        Sort Key: dsr_3.n_ajbs, cbrml_1.n_bs, aj_2.n_ay, dsr_3.n_ssdw1
                                        ->  Sort  (cost=14214.91..14265.12 rows=20086 width=34)
                                              Sort Key: dsr_3.n_ajbs, dsr_3.n_ssdw1, ml_1.c_mc
                                              ->  Nested Loop Left Join  (cost=5846.04..12779.37 rows=20086 width=34)
                                                    ->  Hash Right Join  (cost=5845.62..7326.43 rows=5983 width=26)
                                                          Hash Cond: (dsr_3.n_ajbs = aj_2.n_ajbs)
                                                          ->  Seq Scan on t_xzysdsr dsr_3  (cost=0.00..1245.62 rows=46762 width=18)
                                                          ->  Hash  (cost=5825.39..5825.39 rows=1619 width=18)
                                                                ->  Hash Right Join  (cost=804.26..5825.39 rows=1619 width=18)
                                                                      Hash Cond: ((cbrml_1.n_xh = aj_2.n_cbr) AND (cbrml_1.n_ajbs = aj_2.n_ajbs))
                                                                      ->  Seq Scan on t_xzysyastml cbrml_1  (cost=0.00..4078.11 rows=123911 width=19)
                                                                      ->  Hash  (cost=779.98..779.98 rows=1619 width=18)
                                                                            ->  Seq Scan on t_xzys aj_2  (cost=0.00..779.98 rows=1619 width=18)
                                                                                  Filter: (n_xjafs = ANY ('{2,5,13}'::integer[]))
                                                    ->  Index Scan using i_xzysyastml_ajbs on t_xzysyastml ml_1  (cost=0.42..0.90 rows=1 width=27)
                                                          Index Cond: (dsr_3.n_ajbs = n_ajbs)
                                                          Filter: (dsr_3.n_xh = n_xh)
  ````

  优化后，cost从9万多到现在的3万多。