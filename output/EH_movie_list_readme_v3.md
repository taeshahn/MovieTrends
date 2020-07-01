## 데이터 관련 참고사항

### 1. 일부 영화의 CIN_ID - 영화 아이디(META_ID) 누락

1) 사유
최초 기준 데이터 작업을 위해 추출 했을 당시에는 추출 조건에 충족했으나,
내부 데이터 생성을 위해 신규 추출 시점 (6월 23일) 기준으로 조건에 충족하지 않아 추출 데이터에 누락 된 것으로 사료 됨.

2) 누락 추정 조건
 가> BTV 서비스 중지
  - 아래 추출 스크립트 내 with 절 기준으로, BTV 서비스 중인 콘텐츠(C.DIST_STS_CD = '65')를 추출 
  - 이전 키워드 작업을 위해 추출 시에도 아래 with 절과 동일한 조건으로 추출
  - 시사교양은 성인물/교도소 등 특수 시설 대상 콘텐츠가 다수 포함되어 대상에서 제외 (이전과 동일)
  
 with src as (
select *
from
    (
        SELECT A.META_ID,
               B.SRIS_NM,
               B.SVC_TYP_CD, /*12:디맨드,14:영화추천관,30:일반VOD*/
               ROW_NUMBER() OVER (PARTITION BY A.META_ID ORDER BY B.SVC_TYP_CD DESC) AS IDX,
               B.SVC_FR_DT as BTV_OPEN_DT
        FROM BTVCMS.MT_META_MST A
                 JOIN BTVCMS.CT_SRIS_MST B ON (A.META_ID = B.META_ID AND B.SRIS_TYP_CD = '02')
                 JOIN BTVCMS.CT_SRIS_STS_MRX_DTS C
                      ON (B.SRIS_ID = C.SRIS_ID AND C.DIST_STS_CD = '65' AND C.POC_TYP_CD = '10')
        WHERE TO_NUMBER(OPEN_DY) > 20190000
          AND TO_NUMBER(OPEN_DY) < 20191232
          AND A.THATVER_YN = 'Y'
          AND A.META_TYP_CD NOT IN ('003') /*003:시사교양*/
    )
WHERE IDX=1
)

select
       aaa.META_ID,
       bbb.META_TITLE,
       bbb.SUB_TITLE,
       bbb.ENGL_TITLE,
       bbb.OPEN_DY,
       bbb.DMST_SPCT_CNT,
       aaa.SRIS_NM,
       aaa.BTV_OPEN_DT,
       aaa.meta_manuf_ctry_cd,
       aaa.meta_manuf_ctry_nm,
       aaa.gnr_id,
       aaa.META_GNR_NM,
       bbb.WAT_LVL_CD,
       (select cd_nm from btvcms.ad_comm_cd where comm_grp_cd = 'WAT_LVL_CD' and comm_cd = bbb.wat_lvl_cd) as wat_lvl_nm,
       bbb.manufco_nm,
       bbb.dstbtco_nm,
 --      bbb.sris_yn,
       bbb.sstry_cin_yn,
 --      bbb.thatver_yn,
       bbb.kids_yn
from
    (
        select aa.META_ID,
               aa.SRIS_NM,
               aa.BTV_OPEN_DT,
               aa.meta_manuf_ctry_cd,
               aa.meta_manuf_ctry_nm,
               listagg(cc.gnr_id, '|') within GROUP (ORDER BY cc.sort_seq)      as gnr_id,
               listagg(cc.META_GNR_NM, '|') within GROUP (ORDER BY cc.sort_seq) as META_GNR_NM
        from (
                 select a.META_ID,
                        a.SRIS_NM,
                        a.BTV_OPEN_DT,
                        listagg(B.meta_manuf_ctry_cd, '|')
                                within GROUP (ORDER BY B.meta_manuf_ctry_cd) as meta_manuf_ctry_cd,
                        listagg(
                                (select cd_nm
                                 from btvcms.ad_comm_cd
                                 where comm_grp_cd = 'META_MANUF_CTRY_CD'
                                   and comm_cd = B.meta_manuf_ctry_cd), '|')
                                within GROUP (ORDER BY B.meta_manuf_ctry_cd) as meta_manuf_ctry_nm
                 from src A
                          left outer join btvcms.MT_META_MANUF_CTRY_DTS B on a.meta_id = b.meta_id
                 group by a.META_ID, a.SRIS_NM, a.BTV_OPEN_DT
             ) AA
                 left outer join btvcms.MT_META_GNR_REL BB on aa.meta_id = bb.meta_id
                 left outer join btvcms.mt_gnr_mst CC on bb.gnr_id = cc.gnr_id
        group by aa.META_ID, aa.SRIS_NM, aa.BTV_OPEN_DT, aa.meta_manuf_ctry_cd, aa.meta_manuf_ctry_nm
    ) aaa
join btvcms.mt_meta_mst bbb on aaa.meta_id = bbb.meta_id
order by bbb.meta_title

