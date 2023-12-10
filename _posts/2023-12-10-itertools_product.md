---
layout: post

title:  "itertools product"

categories : Python

tag : [python, itertools]

---

데카르트 곱을 사용한 조합으로 iteration을 돌릴 때 사용

2개 이상의 리스트에서 모든 조합을 구할 때 사용

```python
##json으로 저장된 정보
##df_data['value'][0].get('id'), get('positive'), get('negative'), get('Images')
##각각에서 정보를 가져와서 dataframe으로 만들고 싶을 때


v_list = ['id','positive', 'negative','Images']
 
import itertools
 
for v in v_list:
    df_data[v] =0
 
for v,i in itertools.product(v_list, range(len(df_data['meta']))):
     
    v_value =  df_data['value'][i].get(v)
     
    try :
        df_data[v][i] = v_value
    except:
        df_data[v][i]=None
```

.

.

```python
import pandas as pd
import json
 
# chained_assignment warning 끄기
pd.set_option('chained_assignment',None)
 
 
df_user = pd.read_csv('email.csv')
 
df_user_list=df_user[['userID','user.mail','user.name']]
df_user_list.columns=['user_id','email','name']
df_user_list=df_user_list.drop_duplicates(keep='first').reset_index(drop=True)
 
 
 
df_data=pd.read_csv('data.csv')
df_data['meta'] = df_data['meta'].apply(lambda x : json.loads(x))
df_data['value'] = df_data['meta'].apply(lambda x : x['value'])
 
 
v_list = ['user_id','positive', 'negativet','Images']
 
import itertools
 
for v in v_list:
    df_data[v] =0
 
for v,i in itertools.product(v_list, range(len(df_data['meta']))):
     
    v_value =  df_data['value'][i].get(v)
     
    try :
        df_data[v][i] = v_value
    except:
        df_data[v][i]=None
 
 
v_list2 = ['a','b', 'c','d','e','f']
 
import numpy as np
 
for v2 in v_list2:
    df_data[v2] = np.nan
 
for v2,i in itertools.product(v_list2, range(len(df_data['meta']))):
     
    try:
        v_value2 =  df_data['value'][i]['pipeline'].get(v2)
        df_data[v2][i] = v_value2
         
    except:
         
        pass
 
  
v_list3 = ['a','b', 'c','d','e','f','g',
          'h','i','j','k']
 
for v3 in v_list3:
    df_data[v3] = np.nan
 
for v3,i in itertools.product(v_list3, range(len(df_data['meta']))): 
    try:
        v_value3 =  df_data['value'][i]['options'].get(v3)
        if type(v_value3)==list:
            v_value3 = str(v_value3)
            df_data[v3][i] = v_value3
        else:
            df_data[v3][i] = v_value3
         
    except:
         
        pass
 
df_data.rename(columns={'h':'result_h','w':'result_w'}, inplace=True)
 

v_list4 = ['m', 'w','h']
 
 
for v4 in v_list4:
    df_data[v4] = np.nan
 
for v4,i in itertools.product(v_list4, range(len(df_data['meta']))): 
    try:
        v_value4 =  df_data['value'][i]['opt']['pre'].get(v4)
        if type(v_value4)==list:
            v_value4 = str(v_value4)
            df_data[v4][i] = v_value4
        else:
            df_data[v4][i] = v_value4
         
    except:
         
        pass
 
df_data.rename(columns={'m':'pre_m','h':'pre_h','w':'pre_w'}, inplace=True)
 
 
for v4 in v_list4:
    df_data[v4] = np.nan
 
for v4,i in itertools.product(v_list4, range(len(df_data['meta']))): 
    try:
        v_value4 =  df_data['value'][i]['opt']['post'].get(v4)
        if type(v_value4)==list:
            v_value4 = str(v_value4)
            df_data[v4][i] = v_value4
        else:
            df_data[v4][i] = v_value4
         
    except:
         
        pass
 
df_data.rename(columns={'m':'post_m','h':'post_h','w':'post_w'}, inplace=True)
 
 
v_list5 = ['a', 'b','c','d','e']
 
for v5 in v_list5:
    df_data[v5] = np.nan
 
for v5,i in itertools.product(v_list5, range(len(df_data['meta']))): 
    try:
        v_value5 =  df_data['value'][i]['input'].get(v5)
        if type(v_value5)==list:
            v_value5 = str(v_value5)
            df_data[v5][i] = v_value5
        else:
            df_data[v5][i] = v_value5
         
    except:
         
        pass
 
df_data.rename(columns={'image':'in_image'}, inplace=True)
 
df_data['s'] = np.nan
df_data['et'] = np.nan
 
for i in range(len(df_data['meta'])):
    try:
        v_value =  df_data['value'][i]['output'].get('s')
        if type(v_value)==list:
            v_value = str(v_value)
            df_data['s'][i] = v_value
        else:
            df_data['s'][i] = v_value
         
    except:
         
        pass
     
df_data.rename(columns={'s':'out_s'}, inplace=True)
     
for i in range(len(df_data['meta'])):
    try:
        v_value =  df_data['value'][i]['infer'].get('et')
        if type(v_value)==list:
            v_value = str(v_value)
            df_data['et'][i] = v_value
        else:
            df_data['et'][i] = v_value
         
    except:
         
        pass
     
     
for i in range(len(df_data['meta'])):
    if df_data['user_id'][i]==None:
        df_data['user_id'][i] = df_data['dist.userID'][i]
    else:
        pass
     
df_final = df_data.merge(df_user_list,on='user_id',how='left')
 
 
with pd.ExcelWriter('result.xlsx') as writer:
    df_final.to_excel(writer, sheet_name='result', index=False)
     
print("완료")
```