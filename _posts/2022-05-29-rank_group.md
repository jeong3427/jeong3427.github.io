---
layout: post

title:  "pandas 순위 구하기, 시간별 그룹 구하기"

categories : Python

tag : [python, pandas, rank, group]
---





pandas rank 함수로 래더 순위를 구하고

순위별로 해당 래더가 진행된 기간 동안 얼마나 과금했는지 알아보기

```python
import pandas as pd

#랭크 정보
df= pd.read_csv('rw.txt',sep='\t')

# rw_id, team_id, point 만 사용
df_filter = df[['rw_id','team_id','point']]

# rw_id 별로 순위 구하기
df_filter['rank'] = df_filter.groupby('rw_id')['point'].rank(method='min',ascending=False)
df_filter=df_filter.sort_values(by=(['rw_id','rank']), ascending=True).reset_index(drop=True)

# 과금 정보
df_pur = pd.read_excel('purchase.xlsx',sheet_name='Sheet1')
df_pur['crt_date']=pd.to_datetime(df_pur['crt_date'])


# 과금을 어떤 주에 했는지
import datetime

def classify(x):
    if x<datetime.datetime(2022,3,26,22):
        return 20220319
    elif (x>=datetime.datetime(2022,3,26,22) ) & (x<datetime.datetime(2022,4,2,22)):
        return 20220326
    elif (x>=datetime.datetime(2022,4,2,22)) & (x<datetime.datetime(2022,4,9,22)):
        return 20220402
    elif (x>=datetime.datetime(2022,4,9,22)) & (x<datetime.datetime(2022,4,16,22)):
        return 20220409
    elif (x>=datetime.datetime(2022,4,16,22)) & (x<datetime.datetime(2022,4,23,22)):
        return 20220416
    elif (x>=datetime.datetime(2022,4,23,22)) & (x<datetime.datetime(2022,4,30,22)):
        return 20220423
    else :
        return 20220430
        
df_pur['rw_id'] = df_pur['crt_date'].apply(classify)


# 주차별로 과금액 정리
df_pur_group = df_pur.groupby(['rw_id','team_id']).agg({'price':'sum'}).reset_index(drop=False)
df_final = pd.merge(df_filter, df_pur_group,on=['rw_id','team_id'],how='left')

# 무과금은 0원 처리
df_final=df_final.fillna(0)

# 순위별로 평균 과금액
df_final_in50 = df_final[df_final['rank']<=50]
df_final_in50.groupby(['rw_id']).agg({'price':'mean'})

df_final_in100 = df_final[(df_final['rank']>50) & (df_final['rank']<=100)]
df_final_in100.groupby(['rw_id']).agg({'price':'mean'})

df_final_in300 = df_final[(df_final['rank']>100) & (df_final['rank']<=300)]
df_final_in300.groupby(['rw_id']).agg({'price':'mean'})

df_final_in500 = df_final[(df_final['rank']>300) & (df_final['rank']<=500)]
df_final_in500.groupby(['rw_id']).agg({'price':'mean'})

df_final_in1000 = df_final[(df_final['rank']>500) & (df_final['rank']<=1000)]
df_final_in1000.groupby(['rw_id']).agg({'price':'mean'})

df_final_over1000 = df_final[df_final['rank']>1000]
df_final_over1000.groupby(['rw_id']).agg({'price':'mean'})

# 인원수
df_final.groupby(['rw_id']).agg({'team_id':'count'})
```