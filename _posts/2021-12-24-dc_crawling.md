---
layout: post

title:  "디시인사이드 크롤링 후 시각화"

categories : Python

tag : [python, 크롤링, 시각화]
---



`konlpy`사용 시 java 설치 필요

- 선생님 : [이것이 데이터 분석이다 with 파이썬_08\] 나무위키 분석 예제](https://www.youtube.com/watch?v=cPhUKnLGoGw&list=PLVsNizTWUw7FmLj3IMECoauQ_-DUbNF0M&index=9) 





```python
import requests
import pandas as pd
from bs4 import BeautifulSoup

url = 'https://gall.dcinside.com/mgallery/board/lists/'

list_index = ['제목', '글쓴이', '날짜' ]
list = []

for num in range(1,5):

    try :
        params = {'id':'9up', 'page': f'{num}'}
        headers = {'User-Agent' : 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.162 Safari/537.36'}
        resp = requests.get(url, params=params, headers = headers)
        soup = BeautifulSoup(resp.content, 'html.parser')
        contents = soup.find('tbody').find_all('tr')
        page_size = len(contents)

        for i in contents:
            line =[]
            try :
                new_dict = {}

                title_tag = i.find('a')
                title = title_tag.text
                line.append(title)

                writer_tag = i.find('td', class_ = 'gall_writer ub-writer').find('span', class_ ='nickname')

                try:
                    if writer_tag is not None:
                        writer = writer_tag.text
                        line.append(writer) 

                    else:
                            line.append('없음')

                except :
                        line.append('없음')

                date_tag = i.find('td', class_ = 'gall_date')
                date_dict = date_tag.attrs

                if len(date_dict) == 2:
                    line.append(date_dict['title'])
                else:
                    line.append(date_tag.text)
                
                list.append(line)
            
            except:
                continue
    except:
        continue


    resp.close

df = pd.DataFrame(list, columns =list_index)


title_list = "".join(df['제목'].tolist())


from konlpy.tag import Okt
from collections import Counter

nouns_tagger = Okt()

nouns = nouns_tagger.nouns(title_list)
count = Counter(nouns)


import random
import pytagcloud
import webbrowser

ranked_tags = count.most_common(40)  #자주 사용한 단어 40개
taglist = pytagcloud.make_tags(ranked_tags, maxsize=80)
pytagcloud.create_tag_image(taglist, 'wordcloud.jpg', size = (900, 600), fontname ='NanumGothic', rectangular = False)

from IPython.display import Image
Image(filename='wordcloud.jpg')
```

![자주 사용한](../../../../img/2021-12-24-dc_crawling/자주 사용한.png)