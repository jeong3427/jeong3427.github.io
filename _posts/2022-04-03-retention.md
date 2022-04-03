---
layout: post
title:  "가입일자별 리텐션 시각화"
categories : Python
tag : [python, seaborn, 시각화]
---





게임 특성상 리세마라를 하는 유저가 많아서

3rd party의 device id 기준으로 하나의 계정만 생성한 유저, 

여러 계정을 생성했을 경우 마지막에 가입한 계정만 대상으로 리텐션 계산

.

 .

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df_gamepot = pd.read_csv('Mar23_29.csv', low_memory=False)
df_user = pd.read_csv('team_uid.txt',sep='\t')
df_login = pd.read_csv('login.txt',sep='\t')

df_gamepot = df_gamepot.sort_values(by='createdAt', ascending=False).reset_index(drop=True)
df_gamepot_unique = df_gamepot.drop_duplicates('device_id',keep='first').reset_index(drop=True)

df_unique_user = df_gamepot_unique[['id']]
df_unique_user['check']='v'

df_user.insert(3, 'unique_uid',df_user['uid'].map(df_unique_user.set_index('id')['check']))

df_cal = pd.merge(df_login,df_user, on='team_id',how='inner')


def dailyRetention(df_cal, date):
 cohort=df_cal[(df_cal['unique_uid']== 'v')& (df_cal['team_crt_date']==date)]
 num_of_users= cohort['team_id'].nunique()
 list_cohort= cohort['team_id'].unique()
 df_cohort= df_cal[df_cal['team_id'].isin(list_cohort)]
 cohort_data=pd.DataFrame((df_cohort.groupby(['play_date']).nunique()['team_id']/num_of_users*100).round(2))
 return cohort_data


date_list = df_cal.team_crt_date.drop_duplicates().reset_index(drop=True)


date_len = len(date_list)

ret_list = []
df_pit=pd.DataFrame()

for i in range(date_len-1):
    ret = dailyRetention(df_cal,date_list[i])
    ret['CohortIndex']=ret.index.values.astype(str)[0]
    
    arr= np.arange(1,30,1)
    ret.insert(0, 'day', range(1, 1 + len(ret)))
    re_list = [ret]


    cohort_count= pd.concat(re_list)
    cohort_pivot= cohort_count.pivot(index='CohortIndex', 
    columns='day', values='team_id')
    
    df_pit = df_pit.append(cohort_pivot)
    

#plot
plt.figure(figsize = (35,10))
plt.title('Cohort Analysis - Retention Rate (%)')
sns.heatmap(data = df_pit, 
            annot = True,
            cbar = True,
            fmt='0.3g',
            vmin = 10,
            vmax = 100,
            cmap = "YlGnBu")
plt.xlabel('Number of days from cohort start date')
plt.show()
```

![image-20220403223913256](../../../../img/2022-04-03-retention/image-20220403223913256.png)