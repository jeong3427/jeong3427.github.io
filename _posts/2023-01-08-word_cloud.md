---
layout: post

title:  "워드 클라우드 생성"

categories : Python

tag : [python, 워드클라우드]
---

```python
import pandas as pd
import matplotlib.pyplot as plt
from wordcloud import WordCloud
from collections import Counter
from konlp.tag import Okt

#엑셀에서 긁어오기
df = pd.read_excel('result.xlsx', sheet_name='Sheet1')

#문자 처리
df['contents'] = df['contents'].astype(str)

#한 줄로 이어붙이기
comment_list = "".join(df['contents'].tolist())

#명사만 가져오기
okt = Okt()
nouns = okt.nouns(comment_list)

#한글은 한 글자일 경우 의미 없는 문자들이 많아서 2글자 이상부터 수집
words = [n for n in nouns if len(n)>1]

word_count = Counter(words)


#폰트는 폰트 경로를 설정하면 다른 폰트도 지정 가능
#이미지 크기, 글자 크기 설정
wc = WordCloud(font_path='malgun', width=400, height=400, scale=2.0, max_font_size=250)
gen = wc.generate_from_frequencies(word_count)
plt.figure()
plt.imshow(gen)

#result.png라는 이름으로 저장하기
wc.to_file('result.png')


# 원하는 이미지 모양대로 만들기
from PIL import Image
import numpy as np

img = Image.open('image.jpg')
img_array = np.array(img)

#mask = 옵션에 img_array 추가
wc = WordCloud(font_path='malgun', width=400, height=400, scale=2.0, max_font_size=250,
               mask=img_array)
gen = wc.generate_from_frequencies(word_count)

plt.figure()
plt.imshow(gen)

#result.png라는 이름으로 저장하기
wc.to_file('result.png')


```