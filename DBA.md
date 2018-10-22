# SQL 高级成果物

- 2016年10月-2017年2月，参与最高执行大屏指挥系统的数据整合工作，排查全国上报数据问题，抽取通达海提供的相关失信人数据，标的金额，标的物相关的数据，终本案件失信人相关数据，失信人信用惩戒情况数据统计工作，排查慢SQL优化问题，从失信人年龄维度，职业多维度 统计工作。

- 2017年2月-至今，参与四川高院绩效考核系统的研发工作，主要从事Java后台代码的研发工作，主要负责模块是审判数据补充，考评成绩详情页面展示，考评成绩手动成绩修改，数据同步工作，

- select子查询、多表关联、聚集

  ```sql
  -- 结案日期大于立案日期:
  select fy.c_jc,fy.c_gydm,x from (select c_gydm,count(*) as x from (db_zxaj.t_zhzx_sla sla left join db_zxaj.t_zhzx_jaqk jaqk on sla.ahdm = jaqk.ahdm) left join db_dat_wb.dim_fy fy on sla.gfdm = fy.c_fy where larq >= '20150101' and (jaqk.jarq<sla.larq ) and btrim(jarq)!='' group by fy.c_gydm) as a LEFT JOIN db_dat_wb.dim_fy fy on fy.c_fbdm = a.c_gydm order by x desc;

  -- 性别类型分布
  SELECT mc.fjm as c_fjm ,1 as 'c_dm',COUNT (sxr.c_bzxrmc) as v_zb FROM db_zxzh.V_ZHZX_SXBZXR sxr,db_zxzh.TS_FYMC mc
  WHERE sxr.c_SXZT <> '' and sxr.c_zxfy = mc.fydm and A .XB = '09_00003-1' GROUP BY mc.fjm
  union all
  SELECT mc.fjm as c_fjm ,2 as 'c_dm',COUNT (sxr.c_bzxrmc) as v_zbFROM db_zxzh.V_ZHZX_SXBZXR sxr,db_zxzh.TS_FYMC mc
  WHERE sxr.c_SXZT <> '' and sxr.c_zxfy = mc.fydm and A .XB = '09_00003-2' GROUP BY mc.fjm;
  ```

  ``Union``，对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；select * 默认按照主键排序，也可以自定义排序需要注意的是，排序只能写在最后一个sql.

  ``Union All``，对两个结果集进行并集操作，包括重复行，不进行排序；

- update联表

  ```sql
  -- 连表查询使用中文案号关联，但是实际生产环境中文安好有英文括号也有中文括号所以无法直接关联，这里用Replace替换括号统一
  update db_gjyjda.t_data_aj_gpfhcs zs set n_bh = fb.n_ajbs from db_gjyjda.t_data_fb fb where REPLACE(REPLACE(REPLACE(REPLACE(zs.c_ah,'(',''),'（',''),')',''),'）','') = fb.c_ah";
  ```

  ​

- delete联表

  ```sql
  delete from db_gjyjda.t_data_aj_gpfhcs where n_bh in (select n_bh from db_gjyjda_temp.t_data_aj_gpfhcs_temp where n_bh is not null)";
  ```

  ​

- insert批量、联表

  ```sql
  CREATE OR REPLACE VIEW db_dat_entity.v_zhzx_entity_sxbzxr AS 
  select  sla.ahdm,                                                    -- 案号代码
  		sxr.c_id as id,                                              -- 失信人ID   
  		sla.larq,                                                    -- 立案日期
  		jaqk.jafsn,                                                  -- 结案方式 
  		jaqk.jarq,                                                   -- 结案日期
  		fy.c_fbdm AS jbfy,                                           -- 经办法院
          CASE
              WHEN sla.ajlxdm >= 193 AND sla.ajlxdm <= 196 THEN 1
              WHEN sla.ajlxdm >= 189 AND sla.ajlxdm <= 191 THEN 2
              ELSE 0
          END AS ajlxdm,                                               -- 案件类型代码
  		fy.c_gydm AS gydm,                                           -- 高院代码 
  		fy.c_zydm AS zydm,                                           -- 中院代码 
  		fy.c_qy AS qydm,                                             -- 区县院代码
  		sxr.c_bzxrmc as bzxrmc,                                      -- 被执行人名称
  		case 
  		    when sxr.c_XB = '09_00003-1' then 1 
  		    when sxr.c_XB = '09_00003-2' then 2 
  		    else 3 
  		end as xb,                                                   -- 性别：1.男2.女3其他
  		case 
  		    when sxr.n_nl  < 20 then 1 
  		    when sxr.n_nl >= 20 AND 30 < sxr.n_nl then 2
  		    when sxr.n_nl >= 30 AND 40 < sxr.n_nl then 3
   		    when sxr.n_nl >= 40 AND 50 < sxr.n_nl then 4
  		    when sxr.n_nl >= 50 then 5
  		    else 0
  		end as sxrnlqj,                                               -- 失信人年龄区间
  		coalesce(dsr.zy, '09_00022-255') as zy ,                      -- 失信人职业，如果为空，转为 '09_00022-255'
  		sxr.c_bzxrlx as lx,                                           -- 失信人类型：自然人、法人、非法人
  		sxr.C_SXZT as sxzt,                                           -- 失信状态
  		case 
  		    when sxr.dt_FBSJ >= date_trunc('year', now()) then 1 
  		    else 0 
  		end as jnsxz,                                                 -- 今年失信中
  		coalesce(cj_tl.num_r, 0) as xzly_tl_r,                        -- 限制_铁路_人
  		coalesce(cj_hk.num_r, 0) as xzly_hk_r,                        -- 限制_航空 人
  		coalesce(cj_tl.num_r, 0) | coalesce(cj_hk.num_r, 0) as xzgxf_r,   -- 限制_高消费_人
  		0 as sxbzxrxzcrj_r,                                           -- 13011 限制_出入境_人
  		0 as xycjxzztb_r,                                             -- 13013 限制_信用惩戒限制招投标_人
  		0 as xzcrjhxzgxfhxzztb_r,                                     -- 13014 限制_出入境和限制高消费和限制招投标_人
  		coalesce(cj_tl.num_c, 0) as xzly_tl_c,                        -- 限制_铁路_次
  		coalesce(cj_hk.num_c, 0) as xzly_hk_c,                        -- 限制_航空 次
  		coalesce(cj_tl.num_c, 0) + coalesce(cj_hk.num_c, 0) as xzgxf_c, -- 限制_高消费_次
  		0 as sxbzxrxzcrj_c,                                           -- 13011 限制_出入境_次
  		0 as xycjxzztb_c,                                             -- 13013 限制_信用惩戒限制招投标_次
  		0 as xzcrjhxzgxfhxzztb_c,                                     -- 13014 限制_出入境和限制高消费和限制招投标_次
  		case 
  		    when sxr.c_sxxwqx in('09_ZX0024-1','09_ZX0024-2','09_ZX0024-3','09_ZX0024-4')  then 1
  		    when sxr.c_sxxwqx in('09_ZX0024-5','09_ZX0024-6','09_ZX0024-7','09_ZX0024-8','09_ZX0024-9') then 2
  		    when sxr.c_sxxwqx = '09_ZX0024-10'  then 3
  		    when sxr.c_sxxwqx = '09_ZX0024-11'  then 4
  		    when sxr.c_sxxwqx = '09_ZX0024-12'  then 5
  		    when sxr.c_sxxwqx = '09_ZX0024-255' then 6
  		    else 0 
  		end as sxrfl                                                  -- 失信人情形分类 13102
  from db_zxaj.t_zhzx_sla sla                                           -- from sla表
  inner join db_dat_wb.dim_fy fy ON sla.gfdm = fy.n_fy                  -- from 法院维度表
  inner join db_zxaj.t_zhzx_sxbzxr sxr on sla.ahdm = sxr.c_ahdm         -- from 失信人表
  left  join (select c_ahdm, c_xh, 1 as num_r, count(*) as num_c from 
                   (select distinct c_ahdm, c_xh, d_restrictdate, c_restrictnum, c_type from  db_zxaj.t_zhzx_cjljfkxxb )as cj 
  			where cj.c_type = '0' OR cj.c_type = '1'
  			group by c_ahdm, c_xh) as cj_tl on sxr.c_ahdm = cj_tl.c_ahdm  and sxr.c_xh = cj_tl.c_xh                                                            -- from 惩戒_铁路
  left  join (select c_ahdm, c_xh, 1 as num_r, count(*) as num_c from 
                    (select distinct c_ahdm, c_xh, d_restrictdate, c_restrictnum, c_type from  db_zxaj.t_zhzx_cjljfkxxb )as cj 
  			where cj.c_type = '2'  
  			group by c_ahdm, c_xh) as cj_hk  on sxr.c_ahdm = cj_hk.c_ahdm  and sxr.c_xh = cj_hk.c_xh                                                            -- from 惩戒_航空			
  left  join db_zxaj.t_zxzh_edsr dsr on sxr.c_ahdm = dsr.ahdm and sxr.c_xh = dsr.xh                                                                                 -- from 当事人表
  left  join db_zxaj.t_zhzx_jaqk jaqk on sla.ahdm = jaqk.ahdm                                                                                                       -- from 结案情况表
  -- left join db_zxzh.v_zxzt zt on sxr.ahdm = zt.ahdm and sxr.c_xh = zt.xh
  where 
  sxr.c_sxzt is not null 
  and sxr.c_SXZT <> '' 
  and coalesce(trim(jaqk.jarq, ' '), now()::date::text) >= coalesce(trim(sla.larq, ' '), now()::date::text);  

  -- create table db_dat_entity.t_zhzx_entity_sxbzxr
  -- as select * from db_dat_entity.v_zhzx_entity_sxbzxr where 1 =2;

  -- drop table db_dat_entity.t_zhzx_entity_sxbzxr;

  CREATE TABLE db_dat_entity.t_zhzx_entity_sxbzxr
  (
    ahdm character varying(600),
    id character varying(300),
    larq character varying(600),
    jafsn character varying(600),
    jarq character varying(600),
    jbfy character varying(3),
    ajlxdm integer,
    gydm character varying(3),
    zydm character varying(3),
    qydm character varying(20),
    bzxrmc character varying(300),
    xb integer,
    sxrnlqj integer,
    zy character varying,
    lx character varying(300),
    sxzt character varying(300),
    jnsxz integer,
    xzly_tl_r integer,
    xzly_hk_r integer,
    xzgxf_r integer,
    sxbzxrxzcrj_r integer,
    xycjxzztb_r integer,
    xzcrjhxzgxfhxzztb_r integer,
    xzly_tl_c bigint,
    xzly_hk_c bigint,
    xzgxf_c bigint,
    sxbzxrxzcrj_c integer,
    xycjxzztb_c integer,
    xzcrjhxzgxfhxzztb_c integer,
    sxrfl integer
  )
  DISTRIBUTED BY (ahdm);

  insert into db_dat_entity.t_zhzx_entity_sxbzxr select * from db_dat_entity.v_zhzx_entity_sxbzxr;
  ```



