---
layout: post

title:  "데이터프레임 컬럼 내 텍스트 출현 빈도"

categories : Python

tag : [python, explode,value_counts]

---





## 1. 텍스트 출현 빈도



![image-20240219213259667](../../../../images/2024-02-19-text_counts/image-20240219213259667.png)

alphabet 이라는 컬럼에 여러 문자들이 들어가 있을 때

A는 몇 번 쓰였는지 B는 몇 번 쓰였는지 각각 카운트 하는 코드

```python
import pandas as pd

data = {'alphabet': ['A, B, C', 'A', 'A, C', 'D', 'A, E','B, D']}
df = pd.DataFrame(data)

# 각 알파벳별로 카운트
alphabet_counts = df['alphabet'].str.replace(' ', '').str.split(',').explode().value_counts()

# 결과 출력
print(alphabet_counts)
```

```
A    4
B    2
C    2
D    2
E    1
Name: alphabet, dtype: int64
```

![image-20240219214238553](../../../../images/2024-02-19-text_counts/image-20240219214238553.png)

split을 하기 전에는 Series형식으로 있다가

![image-20240219214429345](../../../../images/2024-02-19-text_counts/image-20240219214429345.png)

split을 하게 되면 리스트 형식으로 바뀌게 된다

.

.

![image-20240219214446263](../../../../images/2024-02-19-text_counts/image-20240219214446263.png)

explode함수는 리스트 형식의 값을 여러 행으로 전개하는 기능을 한다

모두 한 줄로 세워놓고 value_counts를 해주는 방법이다

.

.

```python
#results라는 데이터프레임의 text_wizard라는 컬럼에 내용이 담겨 있음
results['text_wizard']

#jupyter notebook에서 한 눈에 보기 위한 pandas 옵션
pd.set_option('display.max_rows', None)
pd.set_option('display.max_columns', None)
##원래대로
# pd.reset_option('display.max_rows')
# pd.reset_option('display.max_columns')

keywords_counts=results['text_wizard'].str.replace(' ', '').str.split(',').explode().value_counts()
keywords_counts

df_keywords_counts =keywords_counts.to_frame()
```

.

.

## 2. 텍스트 키워드를 유니크한 개인별로 카운트

```python
data = {'alphabet': ['A, B, C', 'A', 'A, C', 'D', 'A, C, E', 'B, D'],
        'name': ['John', 'Jane', 'Kim', 'David', 'Jean','Amy']}
df = pd.DataFrame(data)

# 알파벳을 리스트로 분할하고 explode하여 각 알파벳을 행으로 확장
df_exploded = df.assign(alphabet=df['alphabet'].str.replace(' ', '').str.split(',')).explode('alphabet')

# 알파벳과 이름의 조합에 대한 빈도 계산
alphabet_counts_by_name = df_exploded.groupby(['alphabet', 'name']).size().reset_index(name='count')

# 결과 출력
alphabet_counts_by_name

counts_unique=alphabet_counts_by_name.groupby(['alphabet']).agg({'name':'nunique'})
counts_unique.sort_values(by=['name'], ascending=False)
```

![image-20240219213321466](../../../../images/2024-02-19-text_counts/image-20240219213321466.png)



![image-20240219214702741](../../../../images/2024-02-19-text_counts/image-20240219214702741.png)

```python
df_results=results[['email', 'text_wizard']]
df_exploded = df_results.assign(text_wizard=df_results['text_wizard'].str.replace(' ', '').str.split(',')).explode('text_wizard')

counts_by_keywords = df_exploded.groupby(['text_wizard', 'email']).size().reset_index(name='count')
counts_by_keywords

counts_by_email = counts_by_keywords.groupby(['text_wizard']).agg({'email':'nunique'})
counts_by_email.sort_values(by=['email'], ascending=False)
```