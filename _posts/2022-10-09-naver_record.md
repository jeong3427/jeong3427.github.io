---
layout: post
title:  "네이버 경기 결과 크롤링"
categories : Python
tag : [python, 크롤링, 야구, lambda]
---

.

.

네이버가 페이지 구조를 자주 바꾸기 때문에

자주 수정 필요

.

연장 갈 경우 처리가 복잡하고

find_all 한 다음 attrs로 가져올지 get으로 가져와야 할지에서 헤맴

.

앞부분은 kbo 홈페이지와 동일

.

``` python
from html_table_parser  import parser_functions as parser
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait, Select
from selenium.webdriver.support import expected_conditions as EC
import collections
if not hasattr(collections, 'Callable'):
    collections.Callable = collections.abc.Callable
import pandas as pd
from bs4 import BeautifulSoup as bs
import requests
import re
import time

#0. 날짜 지정
game_day = input('알고 싶은 날짜 입력 (YYYYMMDD) : ')
year = game_day[:4]
month = game_day[4:6]
day = game_day[6:]

#1-1. kbo 홈페이지에서 달력 버튼 누르기
driver = webdriver.Chrome()
kbo_url = 'https://www.koreabaseball.com/Schedule/GameCenter/Main.aspx'
driver.get(kbo_url)

calendar_button = WebDriverWait(driver, 30).until(
    EC.element_to_be_clickable((By.XPATH, '//*[@id="contents"]/div[2]/div/img'))
)

calendar_button.click()

#1-2. 달력에서 설정한 연도, 월, 일 누르기
year_select = Select(driver.find_element(By.XPATH,'//*[@id="ui-datepicker-div"]/div/div/select[2]'))
year_select.select_by_value(year)

month_select = Select(driver.find_element(By.XPATH,'//*[@id="ui-datepicker-div"]/div/div/select[1]'))
month_select.select_by_value(str(int(month)-1))

if int(day)>=10:
    day=day
if int(day)<10:
    day = str(day)
    day = day[1:]
    
day_pick = WebDriverWait(driver, 30).until(
    EC.element_to_be_clickable((By.LINK_TEXT, f'{day}'))
)

day_pick.click()


#1-3. 그 날 진행된 경기 결과 정리
kbo_html = driver.page_source
kbo_soup = bs(kbo_html, 'lxml')

today_game = kbo_soup.find_all('li', class_ = re.compile('game-cont'))

total_game_list = []

for info in today_game:
    g_id = info.attrs['g_id']
    class_info = info.attrs['class']
    total_game_list.append((g_id,class_info))
    
df_total_game = pd.DataFrame(total_game_list)
df_total_game.columns=['g_id','status']

game_count = len(today_game)
driver.quit()

#2. 기록 크롤링
df_etc_final_naver = pd.DataFrame()
df_bat_final_naver = pd.DataFrame()
df_pit_final_naver = pd.DataFrame()

for g in range(game_count):
    
    #취소일 경우 패스
    if 'cancel' in str(df_total_game['status'][g]):
        continue
    
    #경기가 정상 종료일 경우 진행
    elif 'end' in str(df_total_game['status'][g]):
        driver=webdriver.Chrome()
        url= 'https://m.sports.naver.com/game/'+ df_total_game['g_id'][g] + year +'/record'
        driver.get(url)
        driver.maximize_window()
        
        #네이버 마이티켓 끄기
        try:
            close_button = WebDriverWait(driver, 30).until(
                EC.element_to_be_clickable((By.CLASS_NAME, 'MyTicketTooltip_button_close__1PV-1'))
            )
            
            close_button.click()
        except:
            pass
        
        time.sleep(1)
        
        naver_page_loading = WebDriverWait(driver, 30).until(
            EC.presence_of_all_elements_located((By.CLASS_NAME, 'result_list_group'))
        )
        
        naver_html = driver.page_source
        naver_soup = bs(naver_html, 'lxml')
        
        
        #기타 항목
        naver_etc = naver_soup.find_all('dd', class_ = lambda x : x.startswith('MatchResult_info'))
        
        etc_name_list = []
        
        for n in naver_etc:
            name_list = n.get_text()
            etc_name_list.append(name_list)
            
        
        etc_category = naver_soup.find_all('dt', class_ = lambda x : x.startswith('MatchResult_kind'))
        
        etc_category_list = []
        
        for n in etc_category:
            category_list = n.get_text()
            etc_category_list.append(category_list)
            
        df_naver_etc = pd.DataFrame(list(zip(etc_category_list, etc_name_list)), columns=['항목', '이름'])
        
        df_etc_final_naver = pd.concat([df_etc_final_naver, df_naver_etc], axis=0, ignore_index=True)
        
        
        #선수들 player_id 가져오기
        naver_a_tag = naver_soup.find_all('a', class_ = lambda x : str(x).startswith('PlayerRecord_player_info'))
        
        player_id_list = []
        
        for p in naver_a_tag :
            href = p.get('href')
            if 'playerId' in str(href):
                player_id = re.sub(r'[^0-9]',"",href)
                player_id_list.append(int(player_id))
                
        #선수들 이름 가져오기
        naver_span_tag = naver_soup.find_all('span', class_ = lambda x : str(x).startswith('PlayerRecord_name'))
        
        name_list = []
        
        for p in naver_span_tag:
            p_name = p.get_text()
            name_list.append(p_name)
            
        df_info_naver = pd.DataFrame(list(zip(player_id_list, name_list)), columns=['playerid','name'])
        

        #타자, 투수 기록
        df_naver_bat = pd.DataFrame()
        df_naver_pit = pd.DataFrame()
        
        #기록 가져오기 - 타자/타자/투수/투수
        for r in range(4):
            #타자
            if r<2:
                #연장을 갔을 경우
                if naver_soup.find('button', lambda x : str(x).startswith('PlayerRecord_button_next')):
                    #1회부터 9회까지 기록
                    play_info = naver_soup.find_all('table', class_ = lambda x : str(x).startswith('PlayerRecord_record_table'))[r]
                    
                    play_table = parser.make2d(play_info)
                    
                    df = pd.DataFrame(play_table[1:-1], columns=None)

                    #몇 번 타자 이름 정보 날리기
                    df1 = df.iloc[:, 1:]
                    
                    #후반부 기록
                    next_inning_button = WebDriverWait(driver,30).until(
                        EC.element_to_be_clickable((By.CLASS_NAME, 'PlayerRecord_button_next__31NYw'))
                    )
                    
                    next_inning_button.click()
                    
                    #버튼 클릭하고 페이지 다시 로딩
                    naver_html = driver.page_source
                    naver_soup = bs(naver_html, 'lxml')
                    
                    play_info = naver_soup.find_all('table', class_ = lambda x : str(x).startswith('PlayerRecord_record_table'))[r]
        
                    play_table = parser.make2d(play_info)
                    df = pd.DataFrame(play_table[1:-1], columns=None)
                    df = df.iloc[:, 1:]
                    
                    last_inning = int(play_table[0][-1])
                    
                    # df1에는 1회~9회 기록, df2에는 연장 기록
                    if last_inning ==12:
                        df2 = df.iloc[:, -3:]
                    if last_inning ==11:
                        df2 = df.iloc[:, -2:]
                    if last_inning ==10:
                        df2 = df.iloc[:, -1:]
                        
                    #두 개 붙이기
                    df = pd.concat([df1,df2], axis=1, ignore_index=True)
                
                #연장 안 가면  
                else:
                    play_info = naver_soup.find_all('table', class_ = lambda x : str(x).startswith('PlayerRecord_record_table'))[r]
                    play_table = parser.make2d(play_info)
                    df = pd.DataFrame(play_table[1:-1], columns=None)
                    df = df.iloc[:, 1:]
                
                count = len(df)
                
                #선수 정보와 타자 기록 붙이기
                df_info = df_info_naver[:count]
                df = pd.concat([df_info, df], axis=1, ignore_index=True)
                df_naver_bat = pd.concat([df_naver_bat, df], axis=0, ignore_index=True)
                
            
            #투수
            if r>=2:
                play_info = naver_soup.find_all('table', class_ = lambda x : str(x).startswith('PlayerRecord_record_table'))[r]
                play_table = parser.make2d(play_info)
                df = pd.DataFrame(play_table[1:-1], columns=None)
                df = df.iloc[:, 1:]                                                   
                count = len(df)
                
                #선수 정보와 타자 기록 붙이기
                df_info = df_info_naver[:count]
                df = pd.concat([df_info, df], axis=1, ignore_index=True)
                df_naver_pit = pd.concat([df_naver_pit, df], axis=0, ignore_index=True)                    

            #player_id 정리
            df_info_naver = df_info_naver[count:].reset_index(drop=True)
    
    #기타 경기 패스
    else:
        continue
    
    df_bat_final_naver = pd.concat([df_bat_final_naver, df_naver_bat], axis=0, ignore_index=True)
    df_pit_final_naver = pd.concat([df_pit_final_naver, df_naver_pit], axis=0, ignore_index=True)
    
    driver.quit()
    

#연장 진행하지 않은 경기에는 타자 기록이 Nan
df_bat_final_naver.fillna("")

#위에서 컬럼명 모두 None으로 해두고 concat. 마지막에 설정
col_len = len(df_bat_final_naver.columns)

if col_len==22:
    df_bat_final_naver.columns = ['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율',
                                  '1','2','3','4','5','6','7','8','9','10','11','12']
if col_len==21:
    df_bat_final_naver.columns = ['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율',
                                  '1','2','3','4','5','6','7','8','9','10','11']
if col_len==20:
    df_bat_final_naver.columns = ['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율',
                                  '1','2','3','4','5','6','7','8','9','10']
if col_len==19:
    df_bat_final_naver.columns = ['player_id','이름','타수','득점','안타','타점','홈런','볼넷','삼진','타율',
                                  '1','2','3','4','5','6','7','8','9']
df_pit_final_naver.columns =['player_id','이름','이닝','피안타','실점','자책','4사구','삼진','피홈런','타자','타수','투구수',
                             '경기','승리','패전','세이브','평균자책']

#기록 숫자로 변환
df_bat_final_naver.iloc[:, 2:10] = df_bat_final_naver.iloc[:, 2:10].apply(pd.to_numeric, errors='ignore')
df_pit_final_naver['이닝'] = df_pit_final_naver['이닝'].str.replace('⅓', '1/3')
df_pit_final_naver['이닝'] = df_pit_final_naver['이닝'].str.replace('⅔', '2/3')
df_pit_final_naver.iloc[:, 2:] = df_pit_final_naver.iloc[:, 2:].apply(pd.to_numeric, errors='ignore')

with pd.ExcelWriter(f'{game_day}'+'.xlsx') as writer:
    df_bat_final_naver.to_excel(writer, sheet_name='네이버_타격', index=False)
    df_pit_final_naver.to_excel(writer, sheet_name='네이버_피칭', index=False)
    df_etc_final_naver.to_excel(writer, sheet_name='네이버_기타', index=False)
    
print("완료되었습니다")
```

.

.

![image-20221009133346008](../../../../images/2022-10-09-naver_record/image-20221009133346008.png)

![image-20221009133358172](../../../../images/2022-10-09-naver_record/image-20221009133358172.png)