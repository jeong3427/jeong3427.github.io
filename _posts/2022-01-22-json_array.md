---
layout: post

title:  "json array 처리"

categories : Python

tag : [python, pandas, json]
---



json 구조 중 array로만 구성되어 있으면서 데이터 값의 길이가 각각 다를 경우

<img src="../../../../img/2022-01-22-json_array/json-16428508876561.png" alt="json" style="zoom:80%;" />

.

.


```python
import json
import pandas as pd

df=pd.read_excel('team.xlsx',sheet_name='Sheet1')

df_final=pd.DataFrame()


for i in df.index:
    df_data=pd.DataFrame()
    json_data = json.loads(df['team_pows'].values[i])
    df_data= df_data.append(pd.json_normalize(json_data))
    df_data['team_id']=df['team_id'][i]
    df_final = df_final.append(df_data)
    
df_final
```

![result](../../../../img/2022-01-22-json_array/result.PNG)



.

.

---

![df](../../../..//img/2022-01-22-json_array/df.PNG)

![json_loads](../../../../img/2022-01-22-json_array/json_loads.PNG)

![json_data](../../../../img/2022-01-22-json_array/json_data.PNG)