---
layout: post

title:  "네이버 웹툰 영어 요약을 통한 추천 시스템"

categories : Python

tag : [python, 크롤링, lambda, 추천시스템, 추천, 머신러닝]
---





추천시스템 선생님

https://www.youtube.com/watch?v=TNcfJHajqJY

.

.

![image-20221128002123747](../../../../images/2022-11-28-webtoon_recommend/image-20221128002123747.png)

네이버 웹툰 영어 소개글을 통해 비슷한 소개글을 가진 웹툰 추천해내는 코드

한글은 소개글이 짧고 영어만큼 쉽게 학습이 되지 않아서 결과물이 맘에 들지 않음

.

.

우선 네이버 웹툰 영어 사이트 크롤링

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import collections
if not hasattr(collections, 'Callable'):
    collections.Callable = collections.abc.Callable
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)


url ='https://www.webtoons.com/en/dailySchedule'

response = requests.get(url, verify=False)
soup = BeautifulSoup(response.content,'html.parser')

daily_area = soup.find_all('a', class_= lambda x : str(x).startswith('daily_card_item'))

link_list =[]

for contents in daily_area:
    href = contents.get('href')
    link_list.append(href)

info_area = soup.find_all('div', class_='info')

author_list =[]

for contents in info_area:
    author = contents.find('p', class_='author').get_text()
    author_list.append(author)
    

info_list =[]

for i in range(len(link_list)):
    
    comics_url=link_list[i]
    
    response = requests.get(comics_url, verify=False)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    info_all = soup.find_all('em', class_='cnt')
    genre = soup.find('h2', class_= lambda x : str(x).startswith('genre')).text
    title = soup.find('h1', class_='subj').text
    
    view_count = info_all[0].get_text()
    sub_count = info_all[1].get_text()
    start_score = info_all[2].get_text()
    
    summary = soup.find('p', class_='summary').text
    status = soup.find('p', class_='day_info').text
    
    info_list.append((genre, title, view_count, sub_count, start_score, summary, status))
    
    
df_link = pd.DataFrame(link_list)
df_author = pd.DataFrame(author_list)
df_info = pd.DataFrame(info_list)

df_comics = pd.concat([df_link, df_author, df_info], axis=1, ignore_index=True)
df_comics.columns=['링크','작가','장르','제목','뷰','구독자','평점','요약','연재']


with pd.ExcelWriter('naver_webtoon_en.xlsx') as writer:
    #openpyxl.utils.exceptions.IllegalCharacterError 에러가 난다면
    #!pip install xlsxwriter 한 번 해주고 실행
    df_comics.to_excel(writer, sheet_name='Sheet1', index=False, engine='xlsxwriter')
    
print('완료')
```



877개 웹툰 크롤링 결과

![image-20221128002401473](../../../../images/2022-11-28-webtoon_recommend/image-20221128002401473.png)

.

.

추천 함수 만들기

```python
import pandas as pd
df_comics = pd.read_excel('naver_webtoon_en.xlsx', sheet_name='Sheet1')

import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.feature_extraction.text import ENGLISH_STOP_WORDS

tfidf = TfidfVectorizer(stop_words='english')

tfidf_matrix = tfidf.fit_transform(df_comics['요약'])

from sklearn.metrics.pairwise import linear_kernel

cosine_sim = linear_kernel(tfidf_matrix, tfidf_matrix)
indices = pd.Series(df_comics.index, index=df_comics['제목']).drop_duplicates()


def get_recommendations(title, cosine_sim=cosine_sim):
    #제목을 통해서 전체 데이터 기준의 웹툰 index 값 얻기
    idx = indices[title]
    
    #코사인 유사도 매트릭스(cosine_sim)에서 idx에 해당하는 데이터를 (idx, 유사도) 형태로 얻기
    sim_scores = list(enumerate(cosine_sim[idx]))
    
    #코사인 유사도 기준으로 내림차순 정렬
    sim_scores = sorted(sim_scores, key=lambda x:x[1], reverse=True)
    
    #자기 자신을 제외한 10개의 추천 웹툰 제목 슬라이싱
    sim_scores = sim_scores[1:11]
    
    #추천 목록 10개의 인덱스 정보 추출
    webtoon_indices = [i[0] for i in sim_scores]
    
    #인덱스 정보를 통해 제목 추출
    return df_comics['제목'].iloc[webtoon_indices]
```

.

.

![image-20221128002501061](../../../../images/2022-11-28-webtoon_recommend/image-20221128002501061.png)

marry my husband(내 남편과 결혼해줘)로 추천을 받으면

남편을 만렙으로 키우려 합니다

완벽한 결혼의 정석

반드시 해피엔딩

순서로 추천

![image-20221128002941652](../../../../images/2022-11-28-webtoon_recommend/image-20221128002941652.png)

![image-20221128002627723](../../../../images/2022-11-28-webtoon_recommend/image-20221128002627723.png)

![image-20221128003003221](../../../../images/2022-11-28-webtoon_recommend/image-20221128003003221.png)

![image-20221128003025040](../../../../images/2022-11-28-webtoon_recommend/image-20221128003025040.png)