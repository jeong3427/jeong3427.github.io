---
layout: post

title:  "네이버 웹툰 종합 크롤링"

categories : Python

tag : [python, 크롤링, lambda]
---







장르 페이지를 처음 크롤링 하면 데이터 프레임 df_genre에 

제목/ 작가 / 장르 세 컬럼으로 중복해서 들어가게 된다

![image-20221023205726978](../../../../images/2022-10-23-naver_webtoon2/image-20221023205726978.png)

.

그룹바이와 람다 함수를 써서 제목, 작가가 같을 경우 장르는 콤마(,) 후 join을 하라고 시키면 다음과 같이 중복이 제거된다

```python
df_genre_final = df_genre.groupby(['제목', '작가'])['장르'].apply(lambda x : ', '.join(x)).reset_index(drop=False)
```

![image-20221023205934302](../../../../images/2022-10-23-naver_webtoon2/image-20221023205934302.png)



.

replace는 두 번 연속 사용해도 무관

```python
author = author.replace('\n','').replace('\t','')
```

.

.



```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import collections
if not hasattr(collections, 'Callable'):
    collections.Callable = collections.abc.Callable

#1. 웹에서 크롤링

#네이버 웹툰 웹사이트 마지막 인덱스
week_day = ['mon','tue','wed','thu','fri','sat','sun','dailyplus']

#웹 최종 데이터 플레임
df_webtoon_w = pd.DataFrame()


# 요일별 인덱스로 반복문
for day in week_day:
    site_url = "https://comic.naver.com/webtoon/weekdayList?week=" + str(day)

    response = requests.get(site_url)
    soup = BeautifulSoup(response.content, 'html.parser')

    #상단 피쳐드는 제외하면서 시작
    daily= soup.find('div',{'class':'list_area daily_img'})

    comics_list = daily.find_all('dl')
    webtoon_info = []

    for i in comics_list:
        title = i.find('dt').find('a')['title']
        auth = i.find('dd', {'class': 'desc'}).a.get_text()
        rate = i.find('div', {'class': 'rating_type'}).strong.get_text()
        link  = "https://comic.naver.com" + str(i.find('dt').find('a')['href'])
        webtoon_info.append((title, auth, rate, link))

    webtoon_daily = pd.DataFrame(webtoon_info)
    if str(f'{day}')== 'mon':
        webtoon_daily['weekday'] = '월요웹툰'
    elif str(f'{day}')== 'tue':
        webtoon_daily['weekday'] = '화요웹툰'
    elif str(f'{day}')== 'wed':
        webtoon_daily['weekday'] = '수요웹툰'
    elif str(f'{day}')== 'thu':
        webtoon_daily['weekday'] = '목요웹툰'
    elif str(f'{day}')== 'fri':
        webtoon_daily['weekday'] = '금요웹툰'
    elif str(f'{day}')== 'sat':
        webtoon_daily['weekday'] = '토요웹툰'
    elif str(f'{day}')== 'sun':
        webtoon_daily['weekday'] = '일요웹툰'
    elif str(f'{day}')== 'dailyplus':
        webtoon_daily['weekday'] = '매일+웹툰'
    else:
        webtoon_daily['weekday'] = '기타'


    df_webtoon_w = pd.concat([df_webtoon_w, webtoon_daily], axis=0, ignore_index=True)


#컬럼명 정비
df_webtoon_w.columns=['제목','작가','평점','링크','요일']


#2. 모바일에서 관심 수 가져오기

#네이버 웹툰 웹사이트 마지막 인덱스
week_day_m = ['mon','tue','wed','thu','fri','sat','sun','dailyPlus']

df_webtoon_m = pd.DataFrame()

for day in week_day_m:
    webtoon_list_m = []
    mobile_site_url = "https://m.comic.naver.com/webtoon/weekday?week=" + str(day)
    
    response = requests.get(mobile_site_url)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    webtoon = soup.find_all('div', class_='info')

    for info in webtoon:
        title = info.find('span', class_='title_text').get_text()
        
        author = info.find('span', class_='author').get_text()
        #\n 이나 \t제거
        author = author.replace('\n','').replace('\t','')
        
        #관심 수 (신작에는 관심이 비어 있을 수 있어서 try)
        try:
            favcount = info.find('span', class_='count_num').get_text()
        except:
            favcount = ""
        
        webtoon_list_m.append((title, author, favcount))


    df_webtoon_list_m = pd.DataFrame(webtoon_list_m, columns=None)
    df_webtoon_m = pd.concat([df_webtoon_m, df_webtoon_list_m], axis=0, ignore_index=True)
    
df_webtoon_m.columns=['제목', '작가', '관심']


#3. 장르 페이지에서 장르 가져오기

genre_site = 'https://comic.naver.com/webtoon/genre?genre'

response = requests.get(genre_site)
soup = BeautifulSoup(response.content, 'html.parser')

genre_page = soup.find('ul', class_='spot')
a_tag = genre_page.find_all('a')

href_list = []

for genre in a_tag:
    href = genre.attrs['href']
    genre_name = genre.get_text()
    href_list.append((href, genre_name))
    
df_href = pd.DataFrame(href_list, columns=['link', 'genre_name'])

df_genre = pd.DataFrame()

for w in range(len(df_href)):
    url = 'https://comic.naver.com' + str(df_href['link'][w])
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    webtoon = soup.find('ul', class_ = 'img_list')
    
    comics_list = webtoon.find_all('dl')
    
    webtoon_info = []
    
    for i in comics_list:
        title = i.find('dt').find('a')['title']
        author = i.find('dd', {'class':'desc'}).a.get_text()
        
        webtoon_info.append((title, author))
        
    df_webtoon_genre = pd.DataFrame(webtoon_info, columns=['제목', '작가'])
    df_webtoon_genre['장르'] = df_href['genre_name'].values[w]
    
    df_genre = pd.concat([df_genre, df_webtoon_genre], ignore_index=True)
    
    
#하나의 웹툰에 2개 이상의 장르가 있기 때문에 group by 후 람다 처리    
df_genre_final = df_genre.groupby(['제목', '작가'])['장르'].apply(lambda x : ', '.join(x)).reset_index(drop=False)


#4. 웹, 모바일, 장르 합치기

#관심이랑 장르 정보 합치기
df_webtoon_final = df_webtoon_w.join(df_webtoon_m.set_index('제목')['관심'], on='제목')
df_webtoon_final = df_webtoon_final.join(df_genre_final.set_index('제목')['장르'], on='제목')

#평점은 숫자로 변경
df_webtoon_final['평점'] = df_webtoon_final['평점'].apply(pd.to_numeric, errors = 'ignore')

#이틀 연재하는 작품 처리
df_multi = df_webtoon_final.drop_duplicates(subset=['제목','요일'], keep='first')
df_multi = df_multi.groupby(['제목', '작가','평점','관심','장르'])['요일'].apply(lambda x : ', '.join(x)).reset_index(drop=False)
df_multi.columns = ['제목', '작가','평점','관심','장르','연재일']


#평점 순, 관심 순으로 내림차순
df_webtoon_final = df_webtoon_final.sort_values(by=['평점','관심'], ascending=[False, True]).reset_index(drop=True)


#전체는 요일이 아니라 연재일로 정리
df_webtoon_final_total = df_webtoon_final.join(df_multi.set_index('제목')['연재일'], on='제목')
df_webtoon_final_total = df_webtoon_final_total.drop_duplicates(subset=['제목'], keep='first')
df_webtoon_final_total = df_webtoon_final_total.drop(columns=['요일'])

df_webtoon_final = df_webtoon_final.drop_duplicates(subset=['제목','요일'], keep='first')

#컬럼 순서 변경
df_webtoon_final = df_webtoon_final[['제목','작가','평점','관심','장르','요일','링크']]
df_webtoon_final_total = df_webtoon_final_total[['제목','작가','평점','관심','장르','연재일','링크']]


#요일별은 오름차순 내림차순이 없어서 미리 세팅
day_categorize = pd.CategoricalDtype(
    ['월요웹툰','화요웹툰','수요웹툰','목요웹툰','금요웹툰','토요웹툰','일요웹툰','매일+웹툰','기타'],
    ordered=True
)


#엑셀로 저장
with pd.ExcelWriter('naver_webtoon.xlsx') as writer:
    #전체 평점 순으로 저장하고
    df_webtoon_final_total.to_excel(writer, sheet_name='전체',index=False)

    #위에서 수동으로 만든 오름차순 세팅 후 요일별, 평점순으로 저장 
    df_webtoon_final['요일'] = df_webtoon_final['요일'].astype(day_categorize)
    df_webtoon_final = df_webtoon_final.sort_values(by=['요일','평점','관심'], ascending=[True,False,True]).reset_index(drop=True)

    for category in df_webtoon_final.요일:
        df_webtoon_final[df_webtoon_final.요일 == f'{category}'].to_excel(writer, sheet_name=f'{category}',index=False)


print("완료!")
```



전체

![image-20221023210308423](../../../../images/2022-10-23-naver_webtoon2/image-20221023210308423.png)

.

.

각 요일별

![image-20221023210328988](../../../../images/2022-10-23-naver_webtoon2/image-20221023210328988.png)