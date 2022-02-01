---
layout: post

title:  "네이버 웹툰 요일별, 평점별 크롤링"

categories : Python

tag : [python, 크롤링]
---

<img src="../../../../img/2022-02-01-naver_webtoon/image-20220201174045468.png" alt="image-20220201174045468" style="zoom:67%;" />

하나의 요일에서 전체 크롤링하고,

그 다음 요일로 넘어가는 방식

.

.

---

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

#네이버 웹툰 사이트 마지막 인덱스
week_day = ['mon','tue','wed','thu','fri','sat','sun','dailyplus']

#최종 리스트
webtoon_list = pd.DataFrame()


# 요일별 인덱스로 반복문
for day in week_day:
    site_url = "https://comic.naver.com/webtoon/weekdayList?week=" + str(day)

    response = requests.get(site_url)
    soup = BeautifulSoup(response.content, 'html.parser')

    #상단 피쳐드는 제외하면서 시작
    daily= soup.find('div',{'class':'list_area daily_img'})

    comics_list = daily.findAll('dl')
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


    webtoon_list= webtoon_list.append(webtoon_daily)


#컬럼명 정비
webtoon_list.columns=['제목','작가','평점','링크','요일']

#평점 순으로 내림차순
webtoon_list = webtoon_list.sort_values(by=['평점'], ascending=False).reset_index(drop=True)

#요일별은 오름차순 내림차순이 없어서 미리 세팅
day_categorize = pd.CategoricalDtype(
    ['월요웹툰','화요웹툰','수요웹툰','목요웹툰','금요웹툰','토요웹툰','일요웹툰','매일+웹툰','기타'],
    ordered=True
)


#엑셀로 저장
with pd.ExcelWriter('naver_webtoon.xlsx') as writer:
    #전체 평점 순으로 저장하고
    webtoon_list.to_excel(writer, sheet_name='전체',index=False)

    #위에서 수동으로 만든 오름차순 세팅 후 요일별, 평점순으로 저장 
    webtoon_list['요일'] = webtoon_list['요일'].astype(day_categorize)
    webtoon_list = webtoon_list.sort_values(by=['평점','요일'], ascending=[False,True]).reset_index(drop=True)

    for category in webtoon_list.요일:
        webtoon_list[webtoon_list.요일 == f'{category}'].to_excel(writer, sheet_name=f'{category}',index=False)


print("완료!")
```

.

.

**최종 결과물**

![image-20220201174356878](../../../../img/2022-02-01-naver_webtoon/image-20220201174356878.png)

.

모바일에서는 관심 웹툰으로 등록한 숫자를 확인할 수 있는데

PC에서는 정확한 숫자로 안 나오고 99,999로 나오는 중



다음번에는 m.comic.naver.com으로 해볼까