---
layout: post

title:  "연도-월별 합산 후 저장"

categories : Python

tag : [python, pandas, SQL]
---

```sql
#데이터 꺼내오기
#달러 구매일 경우 1150원 곱해서 꺼내기
select id, (case when 가격 < 120 then 가격*1150 else 가격 end) as price, date(생성일자) as date 
from 구매데이터;
```

<img src="../../../../img/2022-01-09-year_month/raw_data.png" alt="raw_data" style="zoom: 33%;" />



---

200만 줄이 넘는 데이터

id별 + 연도_월별 합산 후 각 연도별로 파일 저장하기

```python
import pandas as pd

#2013년 예시
df1 = pd.read_csv('purchase.txt', sep='\t')
df1['date'] = pd.to_datetime(df1['date'])
df1['year'] = df1['date'].dt.year
df1['month'] = df1['date'].dt.month
df_2013 = df1[df1.year ==2013]
df_2013_sum = df_2013.groupby(['id', 'month']).agg({'price':'sum'})


#2014년부터 루프 돌리고 각 연도별로 다른 파일로 저장하기
for i in range(7):
    df1 = pd.read_csv('purchase.txt', sep='\t')
    df1['date'] = pd.to_datetime(df1['date'])
    df1['year'] = df1['date'].dt.year
    df1['month'] = df1['date'].dt.month
    df_year = df1[df1.year == (2014+i)]
    df_year_sum = df_year.groupby(['id', 'month']).agg({'price':'sum'})
    
    dataframe = pd.DataFrame(df_year_sum)
    dataframe.to_csv(f'{2014+i}_result.txt', header = True, index = True)
```

![result](../../../../img/2022-01-09-year_month/result.png)