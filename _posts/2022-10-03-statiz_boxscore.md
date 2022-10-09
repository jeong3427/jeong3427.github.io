---
layout: post

title:  "스탯티즈 박스스코어 크롤링"

categories : Python

tag : [python, pandas, lambda, 야구, 크롤링]

---

.

알고 싶은 날짜를 입력하면

스탯티즈 박스스코어를 크롤링 해오기

.

.

```python
from html_table_parser  import parser_functions as parser
from selenium import webdriver
import collections
if not hasattr(collections, 'Callable'):
    collections.Callable = collections.abc.Callable
import pandas as pd
from bs4 import BeautifulSoup as bs
import requests



#1. 크롤링할 당일의 링크 생성하기
today = input('알고 싶은 날짜 입력 (YYYYMMDD) : ')
date_num = today[:4] + '-' + today[4:6] +'-' + today[6:]
url='http://www.statiz.co.kr/boxscore.php?date='+date_num

response = requests.get(url)
html_text = response.text

soup = bs(response.text, 'html.parser')

a_tag = soup.select('a')

page_link =[]

for i in a_tag:
    game_href=i.attrs['href']
    page_link.append(game_href)


box_score_link = []

for l in range(len(page_link)):
    if str(page_link[l]).endswith('opt=4'): #opt=4로 끝나는 링크가 스탯티즈 박스스코어 링크
        box_score_link.append(page_link[l])

# print(box_score_link)


#2. 타격, 피칭, 수비에 대한 데이터 긁기

df_batting = pd.DataFrame()
df_pitching = pd.DataFrame()
df_defense = pd.DataFrame()

for n in range(len(box_score_link)):
    driver = webdriver.Chrome()
    url= 'http://www.statiz.co.kr/' + str(box_score_link[n])
    driver.get(url)
    
    play_html = driver.page_source
    play_soup = bs(play_html,'html.parser')
    
    
    #2-1. 선수 생일 만들어서 동명이인 선수 구분
    player_info = play_soup.find_all('a')
    
    href_list =[]
    
    for b in player_info:
        href = b.attrs['href']
        href_list.append(href)
        
    birth_day= [d for d in href_list if 'birth' in d]
    df_birth = pd.DataFrame(birth_day, columns=['link'])
    df_birth['date'] = df_birth['link'].apply(lambda x : str(x)[-10:])
    
    
    #2-2. 원정, 홈팀 각각 타격, 피칭, 수비 3개씩이어서 6회 반복
    
    for p in range(6):
        play_info = play_soup.find_all('table', attrs={'class':'table table-striped'})[p]
        play_table = parser.make2d(play_info)
        
        df = pd.DataFrame(play_table[1:-1], columns=play_table[0])
        df['birth'] = 0
        
        #경고 offf
        #생일 붙이는 부분에서 원본 파이에 값을 할당하는데 경고 발생. 무시해도 되니까 꺼둠.
        pd.set_option('mode.chained_assignment', None)
        birth_count = len(df)
        
        if p<=1:
            df_batting = pd.concat([df, df_batting], axis=0, ignore_index=True)
            df_batting['birth'][:birth_count] = df_birth['date'][:birth_count]
            
            
        if 2<=p<4:
            df_pitching = pd.concat([df, df_pitching], axis=0, ignore_index=True)
            df_pitching['birth'][:birth_count] = df_birth['date'][:birth_count]
            
            
        if 4<=p<6:
            df_defense = pd.concat([df, df_defense], axis=0, ignore_index=True)
            df_defense['birth'][:birth_count] = df_birth['date'][:birth_count]
            
            
        df_birth = df_birth[birth_count:].reset_index(drop=True)
        

#3. 승, 패, 홀드, 블론 정보 추출
df_pitching['record'] = df_pitching.이름.str.split('(').str[1]
df_pitching['record'] = df_pitching.record.str.split(',').str[0]
df_pitching['이름'] = df_pitching.이름.str.split('(').str[0]
df_pitching['record'].fillna("",inplace=True)


#4. 기록 숫자로 변환
df_batting.iloc[:,3:-1] = df_batting.iloc[:, 3:-1].apply(pd.to_numeric, errors='ignore')
df_pitching.iloc[:,1:10] = df_pitching.iloc[:,1:10].apply(pd.to_numeric, errors='ignore')
df_pitching.iloc[:,14:-2] = df_pitching.iloc[:,14:-2].apply(pd.to_numeric, errors='ignore')
df_defense.iloc[:,2:-1] = df_defense.iloc[:,2:-1].apply(pd.to_numeric, errors='ignore')

#5. 블론 추가
blown_url= 'http://www.statiz.co.kr/stat.php?mid=stat&re=1&ys='+f'{year}'+'&ye='+f'{year}'+'&se=0&te=&tm=&ty=0&qu=auto&po=0&as=&ae=&hi=&un=&pl=&da=10&o1=BLSV_Plus&o2=WAR&de=1&lr=0&tr=&cv=&ml=2&pa=0&si=&cn=&sn=100'

response = requests.get(blown_url)
html_text = response.text

blown_soup = bs(html_text, 'lxml')

player_ino_b = blown_soup.find_all('a')

href_list_b = []

for b in player_ino_b:
    href = b.attrs['href']
    if (not str(href).endswith('=3')) and (str(href).startswith('player')):
        href_list_b.append(href)
        
birthday_b = [d for d in href_list_b if 'birth' in d]

df_birth_b = pd.DataFrame(birthday_b, columns=['link'])
df_birth_b['date'] = df_birth_b['link'].apply(lambda x : str(x)[-10:])

df_date = df_birth_b.drop_duplicates(['link','date'], keep='first')

blown_info = blown_soup.find('table', attrs={'id':'mytable'})

b_record_table= parser.make2d(blown_info)

df_blown = pd.DataFrame(b_record_table[2:], columns=b_record_table[0])

pd.set_option('mode.chained_assignment', None)
df_blown['birth'] = df_date['date']

df_blown = df_blown[['순', '이름','팀','정렬','birth']]
df_blown['정렬'] = df_blown['정렬'].apply(pd.to_numeric)

df_blown_final = df_blown[df_blown['정렬']>0]
df_blown_final.columns=['순','이름','팀','BL+','birth']


#5. 엑셀에 정리
with pd.ExcelWriter(f'{today}'+'.xlsx') as writer:
    df_batting.to_excel(writer, sheet_name='타격', index=False)
    df_pitching.to_excel(writer, sheet_name='피칭', index=False)
    df_defense.to_excel(writer, sheet_name='수비', index=False)
    df_blown_final.to_excel(writer, sheet_name='블론', index=False)
    
print("완료되었습니다")
```

.

.

![image-20221003182851434](../../../../images/2022-10-03-statiz_boxscore/image-20221003182851434.png)

결과

