---
layout: post

title:  "야구 크롤링 한 이닝 두 타석 처리, 선발/중계 구분"

categories : Python

tag : [python, pandas, lambda, 야구, 크롤링]

---

![image-20230405221516466](../../../../images/2023-04-05-baseball_crawler/image-20230405221516466.png)

한 이닝에 두 타석을 서는 경우 같은 슬래시를 통해 표기한다.

엑셀에 한 칸당 하나의 타석 결과만 남기고 싶을 때

.

```python
#df_bat_final_naver 이라는 데이터 프레임에 위와 같은 표 크롤링을 끝낸 상태

df_bat_info = df_bat_final_naver.iloc[:,0:10] #선수 정보, 기록 요약은 따로 떼놓고

df_bat_check = df_bat_final_naver.iloc[:,10:] #이닝별 결과만

df_second_check = pd.DataFrame()

for i in range(len(df_bat_check.columns)):
    col = df_bat_check.iloc[:,i]
    new_col = col.str.split('/',expand=True)
    
    col_len = len(new_col.columns)
    
    #split한 결과가 한 줄이면 이닝 그대로 표기
    if col_len ==1:
        new_col.columns=[f'{i+1}'] 
        
    #split한 결과가 두 줄이면 2, 2-2 같이 표기    
    if col_len ==2:
        new_col.columns=[f'{i+1}',str(f'{i+1}')+'-2'] 
        
    #호옥시 한 이닝에 세 타석..
    if col_len ==3:
        new_col.columns=[f'{i+1}',str(f'{i+1}')+'-2',str(f'{i+1}')+'-3']
        
    #2-2와 같은 이닝을 그대로 표기, join 처리 안 하면 기존과 같이 1,2,3,4로 표기됨
    df_second_check = pd.concat([df_second_check,new_col], axis=1,join='outer')


#선수 정보와 붙이기
df_bat_final = pd.concat([df_bat_info, df_second_check],axis=1)
```



결과는 이렇게.

![image-20230405222029038](../../../../images/2023-04-05-baseball_crawler/image-20230405222029038.png)

.

.

선발 투수와 중계 투수 구분

![image-20230405222129051](../../../../images/2023-04-05-baseball_crawler/image-20230405222129051.png)

가장 처음 나온 투수에겐 선발, 그 뒤로는 중계라고 구분해주는 열을 추가해준다



```python
#기록 크롤링 후 df 라는 데이터 프레임에 넣어둔 상황
df = pd.DataFrame(play_table[1:-1], columns=None)
df = df.iloc[:,1:]

#첫 번째 열에 '구분'이라는 열을 추가하고, 그 값을 첫 번째는 '선발' 그 외는 '중계'라고 넣는다
df.insert(0,'구분', ['선발' if n==0 else '중계' for n in range(len(df))])
```

.

결과

![image-20230405222437271](../../../../images/2023-04-05-baseball_crawler/image-20230405222437271.png)

.

.

전체 코드

```python
from html_table_parser import parser_functions as parser
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import Select
import collections
if not hasattr(collections, 'Callable'):
    collections.Callable = collections.abc.Callable
import pandas as pd
from bs4 import BeautifulSoup as bs
import requests
import time
import re

# 경고 off
# 생일 붙이는 부분에서 원본 파일에 값을 할당하는데 경고 발생. 무시해도 되니까 꺼둠.
pd.set_option('mode.chained_assignment',  None) 

#0. 날짜 지정
game_day = input('알고 싶은 날짜 입력(YYYYMMDD) : ')
year = game_day[:4]
month = game_day[4:6]
day =  game_day[6:]
date_num = year+'-'+month+'-'+day


#1. 날짜 선택
driver = webdriver.Chrome()
kbo_url = 'https://www.koreabaseball.com/Schedule/GameCenter/Main.aspx'
driver.get(kbo_url)

#1-1. kbo 홈페이지에서 달력 버튼 누르기
calender_button = WebDriverWait(driver,30).until(
    EC.element_to_be_clickable((By.XPATH,'//*[@id="contents"]/div[2]/div/img'))
)

calender_button.click()


#1-2. 달력에서 설정한 연도, 월, 일 누르기
year_select=Select(driver.find_element(By.XPATH,'//*[@id="ui-datepicker-div"]/div/div/select[2]'))
year_select.select_by_value(year)

month_select=Select(driver.find_element(By.XPATH, '//*[@id="ui-datepicker-div"]/div/div/select[1]'))
month_select.select_by_value(str(int(month)-1))

if int(day)>9:
    day = day
if int(day) <10:
    day = str(day)
    day = day[1:]

day_pick = WebDriverWait(driver,10).until(
    EC.element_to_be_clickable((By.LINK_TEXT,f'{day}'))
)

day_pick.click()

#1-3. 그 날 진행된 게임 결과 정리
kbo_html = driver.page_source
kbo_soup = bs(kbo_html, 'lxml')

today_game = kbo_soup.find_all('li', class_ =re.compile('game-cont'))

total_game_list=[]

for info in today_game:
    g_id = info.attrs['g_id']
    clss_info= info.attrs['class']
    total_game_list.append((g_id,clss_info))

df_total_game = pd.DataFrame(total_game_list)
df_total_game.columns=['g_id','status']


game_count = len(today_game)

driver.quit()

#2. 네이버 기록 크롤링
df_etc_final_naver = pd.DataFrame()
df_bat_final_naver = pd.DataFrame()
df_pit_final_naver = pd.DataFrame()

for g in range(game_count):
    
    if 'cancel' in str(df_total_game['status'][g]):
        continue  
    
    elif 'end' in str(df_total_game['status'][g]) :
        driver = webdriver.Chrome()
        url = 'https://m.sports.naver.com/game/'+df_total_game['g_id'][g]+year+'/record'
        driver.get(url)
        driver.maximize_window()
        
        try:
            close_button = WebDriverWait(driver,30).until(
                EC.element_to_be_clickable((By.CLASS_NAME,'MyTicketTooltip_button_close__1PV-1'))
                        )

            close_button.click()
        except:
            pass
        
        time.sleep(1)
        
        naver_page_loading = WebDriverWait(driver,30).until(
            EC.presence_of_all_elements_located((By.CLASS_NAME,'result_list_group'))
            )

        naver_html = driver.page_source
        naver_soup = bs(naver_html, 'lxml')

        #기타 항목
        naver_etc = naver_soup.find_all('dd', class_= lambda x : x.startswith('MatchResult_info'))

        etc_name_list =[]
        for n in naver_etc:
            name_list = n.get_text()
            etc_name_list.append(name_list)

        etc_category_list=[]
        etc_category = naver_soup.find_all('dt', class_= lambda x : x.startswith('MatchResult_kind'))

        for n in etc_category:
            category_list = n.get_text()
            etc_category_list.append(category_list)
            
        df_naver_etc = pd.DataFrame(list(zip(etc_category_list, etc_name_list)), columns=['항목','이름'])

        df_etc_final_naver = pd.concat([df_etc_final_naver,df_naver_etc], axis=0, ignore_index=True)

        #선수들 player_id 가져오기
        naver_a_tag = naver_soup.find_all('a', class_= lambda x : str(x).startswith('PlayerRecord_player_info'))

        player_id_list=[]
        for p in naver_a_tag:
            href = p.get('href')
            if 'playerId' in str(href):
                player_id = re.sub(r'[^0-9]', '', href)
                player_id_list.append(int(player_id))


        #선수들 이름 가져오기
        naver_span_tag = naver_soup.find_all('span', class_= lambda x : str(x).startswith('PlayerRecord_name'))

        name_list=[]
        for p in naver_span_tag:
            p_name = p.get_text()
            name_list.append(p_name)

        df_info_naver = pd.DataFrame(list(zip(player_id_list,name_list)),columns=['playerid','name'])

        
        #타자, 투수 기록
        df_naver_bat = pd.DataFrame()
        df_naver_pit = pd.DataFrame()

        #기록 가져오기
        for r in range(4):
            if r<2:
                if naver_soup.find('button',lambda x: str(x).startswith('PlayerRecord_button_next')):
                    play_info = naver_soup.find_all('table', class_=lambda x : str(x).startswith('PlayerRecord_record_table'))[r]

                    play_table = parser.make2d(play_info)

                    df = pd.DataFrame(play_table[1:-1], columns=None)
                    df1 = df.iloc[:,1:]
                    
                    next_inning_button = WebDriverWait(driver,30).until(
                    EC.element_to_be_clickable((By.CLASS_NAME,'PlayerRecord_button_next__31NYw'))
                                )

                    next_inning_button.click()
                    
                    naver_html = driver.page_source
                    naver_soup = bs(naver_html, 'lxml')

                    play_info = naver_soup.find_all('table', class_=lambda x : str(x).startswith('PlayerRecord_record_table'))[r]
                    play_table = parser.make2d(play_info)
                    df = pd.DataFrame(play_table[1:-1], columns=None)
                    df = df.iloc[:,1:]
                    last_inning = int(play_table[0][-1])
                    
                    if last_inning==12:
                        df2=df.iloc[:,-3:]
                    if last_inning == 11:
                        df2=df.iloc[:,-2:]
                    if last_inning == 10:
                        df2=df.iloc[:,-1:]
                    
                    df = pd.concat([df1,df2],axis=1, ignore_index=True)


                else:
                    play_info = naver_soup.find_all('table', class_=lambda x : str(x).startswith('PlayerRecord_record_table'))[r]

                    play_table = parser.make2d(play_info)

                    df = pd.DataFrame(play_table[1:-1], columns=None)
                    df = df.iloc[:,1:]
                    
                count = len(df)
                    
                df_info = df_info_naver[:count]
                df=pd.concat([df_info,df], axis=1,ignore_index=True)
                df_naver_bat=pd.concat([df_naver_bat, df], axis=0,ignore_index=True)  
                    
            if r>=2:
                play_info = naver_soup.find_all('table', class_=lambda x : str(x).startswith('PlayerRecord_record_table'))[r]

                play_table = parser.make2d(play_info)

                df = pd.DataFrame(play_table[1:-1], columns=None)
                df = df.iloc[:,1:]
                df.insert(0,'구분', ['선발' if n==0 else '중계' for n in range(len(df))])
                
                count = len(df)
                
                df_info = df_info_naver[:count]
                df=pd.concat([df_info,df], axis=1,ignore_index=True)
                df_naver_pit=pd.concat([df_naver_pit, df], axis=0,ignore_index=True)        

            df_info_naver = df_info_naver[count:].reset_index(drop=True)
    else:
        continue


    df_bat_final_naver = pd.concat([df_bat_final_naver, df_naver_bat], axis=0,ignore_index=True)
    df_pit_final_naver = pd.concat([df_pit_final_naver, df_naver_pit], axis=0,ignore_index=True)
    
    driver.quit()


df_bat_final_naver.fillna("")
col_len = len(df_bat_final_naver.columns)


if col_len==22:
    df_bat_final_naver.columns=['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율','1','2','3','4','5','6','7','8','9','10','11','12']
if col_len==21:
    df_bat_final_naver.columns=['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율','1','2','3','4','5','6','7','8','9','10','11']
if col_len==20:
    df_bat_final_naver.columns=['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율','1','2','3','4','5','6','7','8','9','10']
if col_len==19:
    df_bat_final_naver.columns=['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율','1','2','3','4','5','6','7','8','9']

df_pit_final_naver.columns=['player_id','이름','구분','이닝','피안타','실점','자책','4사구','삼진','피홈런','타자','타수','투구수','경기','승리','패전','세이브','평균자책']


#2-2. 기록 숫자로 변환
df_bat_final_naver.iloc[:,2:10] = df_bat_final_naver.iloc[:,2:10].apply(pd.to_numeric, errors='ignore')
df_pit_final_naver['이닝']=df_pit_final_naver['이닝'].str.replace('⅓','1/3')
df_pit_final_naver['이닝']=df_pit_final_naver['이닝'].str.replace('⅔','2/3')

df_pit_final_naver.iloc[:,3:] = df_pit_final_naver.iloc[:,3:].apply(pd.to_numeric, errors='ignore')


#2-3. 한 이닝 두 타석 처리
df_bat_info = df_bat_final_naver.iloc[:,0:10]
df_bat_check = df_bat_final_naver.iloc[:,10:]

df_second_check = pd.DataFrame()

for i in range(len(df_bat_check.columns)):
    col = df_bat_check.iloc[:,i]
    new_col = col.str.split('/',expand=True)
    col_len = len(new_col.columns)
    if col_len ==1:
        new_col.columns=[f'{i+1}']
    if col_len ==2:
        new_col.columns=[f'{i+1}',str(f'{i+1}')+'-2']
    if col_len ==3:
        new_col.columns=[f'{i+1}',str(f'{i+1}')+'-2',str(f'{i+1}')+'-3']
    
    df_second_check = pd.concat([df_second_check,new_col], axis=1,join='outer')

df_bat_final = pd.concat([df_bat_info, df_second_check],axis=1)


#3. WPA 페이지 추가
wpa_url_list= ['http://www.statiz.co.kr/stat.php?mid=stat&re=0&ys='+f'{year}'+'&ye='+f'{year}'+'&se=0&te=&tm=&ty=0&qu=auto&po=0&as=&ae=&hi=&un=&pl=&da=4&o1=WPA&o2=WAR_ALL&de=1&lr=0&tr=&cv=&ml=2&sn=1000&si=&cn='
, 'http://www.statiz.co.kr/stat.php?mid=stat&re=1&ys='+f'{year}'+'&ye='+f'{year}'+'&se=0&te=&tm=&ty=0&qu=auto&po=0&as=&ae=&hi=&un=&pl=&da=5&o1=WPA&o2=WAR&de=1&lr=0&tr=&cv=&ml=2&pa=0&si=&cn=&sn=1000']

df_wpa_final = pd.DataFrame()

for url in wpa_url_list:
    response = requests.get(url)
    html_text = response.text

    blown_soup = bs(response.text, 'lxml')

    player_info_b = blown_soup.find_all('a')

    href_list_b=[]

    for b in player_info_b:
        href = b.attrs['href']
        if ('opt' not in str(href)) and (str(href).startswith('player')):
            href_list_b.append(href)


    birth_day_b = [d for d in href_list_b if 'birth' in d]
    df_birth_b = pd.DataFrame(birth_day_b, columns=['link'])
    df_birth_b['date'] = df_birth_b['link'].apply(lambda x : str(x)[-10:])

    df_date = df_birth_b.drop_duplicates(['link','date'],keep='first')

    wpa_info = blown_soup.find('table',attrs={'id':'mytable'})
    record_table = parser.make2d(wpa_info)

    df_wpa = pd.DataFrame(record_table[2:], columns=record_table[0])

    pd.set_option('mode.chained_assignment',  None) 
    df_wpa['birth'] = df_date['date']

    df_wpa = df_wpa[['순','이름','팀','정렬', 'birth']]
    df_wpa['정렬'] = df_wpa['정렬'] .apply(pd.to_numeric)
    
    df_wpa_final = pd.concat([df_wpa_final, df_wpa])
    df_wpa_final=df_wpa_final.drop(columns=['순'])
    df_wpa_final = df_wpa_final.sort_values(by=['정렬'],ascending=False)


# 4. 엑셀에 정리
with pd.ExcelWriter(f'{game_day}'+'.xlsx') as writer:
    df_bat_final.to_excel(writer, sheet_name='네이버_타격', index=False)
    df_pit_final_naver.to_excel(writer, sheet_name='네이버_피칭', index=False)
    df_etc_final_naver.to_excel(writer, sheet_name='네이버_기타', index=False)
    df_wpa_final.to_excel(writer, sheet_name='WPA',index=False)
    
print("크롤링 완료")
```