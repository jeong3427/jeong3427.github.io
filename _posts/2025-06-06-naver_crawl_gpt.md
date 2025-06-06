---
layout: post
title:  "네이버 야구 경기 결과 크롤링"
categories : Python
tag : [python, 크롤링, 야구, chatGPT]
---



네이버 야구 결과 ui 변화로 코드 전면 수정

```python
### 날짜 세팅 ###

#-*- encoding: utf-8 -*-
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options as ChromeOptions
from webdriver_manager.chrome import ChromeDriverManager

options = ChromeOptions()

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
pd.set_option('mode.chained_assignment',  None) 

# Selenium 옵션 설정
options = ChromeOptions()
options.add_argument("--headless")  # GUI 없이 실행 (성능 개선)
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")

# 크롬 드라이버 최신 버전 설정
service = ChromeService(executable_path=ChromeDriverManager().install())


##0. 날짜 입력 칸 세팅
game_day = input('알고 싶은 날짜 입력(YYYYMMDD) : ')
year = game_day[:4]
month = game_day[4:6]
day =  game_day[6:]
date_num = year+'-'+month+'-'+day
        
# chrome driver
driver = webdriver.Chrome(service=service, options=options)


### kbo 홈페이지
##1. 날짜 선택
kbo_url = 'https://www.koreabaseball.com/Schedule/GameCenter/Main.aspx'
driver.get(kbo_url)

##1-1. kbo 홈페이지에서 달력 버튼 누르기
calender_button = WebDriverWait(driver,10).until(
    EC.element_to_be_clickable((By.XPATH,'//*[@id="contents"]/div[2]/div/img'))
)

calender_button.click()

##1-2. 달력에서 설정한 연도, 월, 일 누르기
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

##1-3. 그 날 진행된 게임 결과 정리
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
```

.

.

```python
### 기록 ###

##2. 네이버 기록 크롤링
df_etc_crawl = pd.DataFrame()
df_bat_crawl = pd.DataFrame()
df_pit_crawl = pd.DataFrame()


for g in range(game_count):
    try:    
        if 'cancel' in str(df_total_game['status'][g]):
            continue  
        
        elif 'end' in str(df_total_game['status'][g]) :
            game_name = str(df_total_game['g_id'][g])

            away_team_name = game_name[8:10]
            home_team_name = game_name[10:12]
            dic_korean_team_name = {"SK" : "SSG","SS" : "삼성","OB":"두산","HT":"기아", "NC":"NC","WO":"키움","LG":"LG","KT":"kt","HH":"한화","LT":"롯데"}
            print(f"{dic_korean_team_name[away_team_name]}" " : " f"{dic_korean_team_name[home_team_name]} 시작")
            
        
            driver = webdriver.Chrome(service=service, options=options)
            url = 'https://m.sports.naver.com/game/'+df_total_game['g_id'][g]+year+'/record'

            driver.get(url)
            
            time.sleep(1)
            
            naver_page_loading = WebDriverWait(driver,15).until(
                EC.presence_of_all_elements_located((By.CLASS_NAME,'result_list_group'))
                )

            naver_html = driver.page_source
            naver_soup = bs(naver_html, 'lxml')

            ##2-1. 기타 항목
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
            
            # 최종 기타 데이터프레임
            df_naver_etc = pd.DataFrame(list(zip(etc_category_list, etc_name_list)), columns=['항목','이름'])

			#### 타자 ####
            ##2-2-a. 타자 player_id, 이름
            batter_data = []
            batter_uls = naver_soup.find_all('ul', class_='player_list PlayerRecord_type_batter__1rHnm')
            for ul in batter_uls:
                for a in ul.find_all('a', class_=re.compile(r'^PlayerRecord_player_info')):
                    href = a.get('href')
                    name_tag = a.find('span', class_='PlayerRecord_name__1W_c0')
                    if href and name_tag:
                        numeric_id = re.sub(r'\D', '', href)
                        player_name = name_tag.get_text(strip=True)
                        if numeric_id:
                            batter_data.append([numeric_id, player_name])

            batter_df = pd.DataFrame(batter_data, columns=["player_id", "이름"])

            ##2-2-b. 타자 포지션 정보
            li_tags = naver_soup.find_all('li', class_='PlayerRecord_player_item__3ECIB')

            records = []
            for li in li_tags:
                # 하위 요소들을 모두 span 또는 div 단위로 추출
                fields = [elem.get_text(strip=True) for elem in li.find_all(['span', 'div']) if elem.get_text(strip=True)]
                records.append(fields)

            max_len = max(len(row) for row in records)
            normalized_rows = [row + [''] * (max_len - len(row)) for row in records]
            columns = [f'field_{i+1}' for i in range(max_len)]

            df = pd.DataFrame(normalized_rows, columns=columns)
            df_replaced = df.replace('', pd.NA)

            # NaN이 하나라도 있는 행은 모두 제거
            df_cleaned = df_replaced.dropna().reset_index(drop=True)
            df_position = df_cleaned.drop(columns=['field_1', 'field_2', 'field_3', 'field_4'], errors='ignore')

            df_position.rename(columns={"field_5": "포지션"}, inplace=True)


            ##2-2-c. 타자 기록
            tables = naver_soup.find_all('table', class_='PlayerRecord_record_table__19F6_ PlayerRecord_type_batter__1rHnm eg-flick-panel')

            dataframes = []
            for table in tables:
                rows = []
                for tr in table.find_all('tr'):
                    row = []
                    for cell in tr.find_all(['th', 'td']):
                        spans = cell.find_all('span')
                        visible_texts = [s.get_text(strip=True) for s in spans if 'blind' not in s.get('class', [])]
                        cell_text = ''.join(visible_texts)
                        row.append(cell_text)
                    if row:
                        rows.append(row)

                max_len = max(len(r) for r in rows)
                normalized_rows = [r + [''] * (max_len - len(r)) for r in rows]
                df = pd.DataFrame(normalized_rows)
                dataframes.append(df)


            # 테이블 합치기
            df_combined = pd.concat(dataframes, ignore_index=True)

            # "타자명" 또는 "합계"인 행 제거
            df_cleaned = df_combined[
                (df_combined.iloc[:, 0] != "타자명") & 
                (df_combined.iloc[:, 0] != "합계")
            ].reset_index(drop=True)

            df_cleaned = df_cleaned.drop(df_cleaned.columns[0], axis=1).reset_index(drop=True)
			
            # 최종 타자 데이터프레임
            df_naver_bat = pd.concat([batter_df,df_position, df_cleaned], axis=1,ignore_index=True)

            
            #### 투수 ####
            ##2-3-a. 투수 player_id, 이름
            pitcher_data = []
            pitcher_uls = naver_soup.find_all('ul', class_='player_list PlayerRecord_type_pitcher__1TYaz')
            for ul in pitcher_uls:
                for a in ul.find_all('a', class_=re.compile(r'^PlayerRecord_player_info')):
                    href = a.get('href')
                    name_tag = a.find('span', class_='PlayerRecord_name__1W_c0')
                    if href and name_tag:
                        numeric_id = re.sub(r'\D', '', href)
                        player_name = name_tag.get_text(strip=True)
                        if numeric_id:
                            pitcher_data.append([numeric_id, player_name])

            pitcher_df = pd.DataFrame(pitcher_data, columns=["player_id", "이름"])


            ##2-3-b. 선발, 중계, 승홀세 정보
            #div 목록 수집
            player_divs_p = naver_soup.find_all('div', class_='PlayerRecord_player_list_area__gbho3')

            #2번째와 4번째 div만 선택
            selected_divs = [player_divs_p[i] for i in [1, 3] if i < len(player_divs_p)]

            #결과 저장
            all_rows = []

            for div in selected_divs:
                li_tags = div.find_all('li', class_='PlayerRecord_player_item__3ECIB')
                for idx, li in enumerate(li_tags):
                    # 기본 텍스트 필드 수집
                    fields = [elem.get_text(strip=True) for elem in li.find_all(['span', 'div']) if elem.get_text(strip=True)]

                    # em 정보 (선택적 - 승,홀,세,패 승일 경우는 em class 이름이 다름)
                    em_tag = li.find('em', class_=re.compile(r'^PlayerRecord_sub_info__gDKqR'))
                    sub_info = em_tag.get_text(strip=True) if em_tag else ''

                    # "합계" 행은 건너뜀
                    if fields and fields[0] == '합계':
                        continue

                    # 구분값 ('선발' or '중계') 삽입: field_1 다음 위치
                    role = '선발' if idx == 0 else '중계'
                    if len(fields) >= 1:
                        full_row = [fields[0], role] + fields[1:] + [sub_info]
                    else:
                        full_row = [''] + [role] + [] + [sub_info]

                    all_rows.append(full_row)

            # 열 정규화
            max_len = max(len(row) for row in all_rows)
            normalized_rows = [row + [''] * (max_len - len(row)) for row in all_rows]

            # 열 이름 생성
            columns = ['field_1', '구분'] + [f'field_{i+2}' for i in range(max_len - 3)] + ['sub_info']

            # DataFrame 생성
            df_sprp = pd.DataFrame(normalized_rows, columns=columns)
            
            df_sprp.columns=['이름','구분','승홀세']
            df_sprp=df_sprp.drop(columns=['이름'], errors='ignore')

 			##2-3-c. 투수 기록
            tables_p = naver_soup.find_all('table', class_='PlayerRecord_record_table__19F6_ PlayerRecord_type_pitcher__1TYaz eg-flick-panel')

            dataframes_p = []
            for table in tables_p:
                rows = []
                for tr in table.find_all('tr'):
                    row = []
                    for cell in tr.find_all(['th', 'td']):
                        spans = cell.find_all('span')
                        visible_texts = [s.get_text(strip=True) for s in spans if 'blind' not in s.get('class', [])]
                        cell_text = ''.join(visible_texts)
                        row.append(cell_text)
                    if row:
                        rows.append(row)

                max_len = max(len(r) for r in rows)
                normalized_rows = [r + [''] * (max_len - len(r)) for r in rows]
                df = pd.DataFrame(normalized_rows)
                dataframes_p.append(df)


            # 테이블 합치기
            df_combined_p = pd.concat(dataframes_p, ignore_index=True)

            # "투수명" 또는 "합계"인 행 제거
            df_cleaned_p = df_combined_p[
                (df_combined_p.iloc[:, 0] != "투수명") & 
                (df_combined_p.iloc[:, 0] != "합계")
            ].reset_index(drop=True)

            df_cleaned_p = df_cleaned_p.drop(df_cleaned_p.columns[0], axis=1).reset_index(drop=True)

            
            # 최종 투수 데이터 프레임
            df_naver_pit = pd.concat([pitcher_df,df_sprp,df_cleaned_p], axis=1,ignore_index=True)    

            
			# 중간 결과 프린트
            game_name = str(df_total_game['g_id'][g])
            away_team_name = game_name[8:10]
            home_team_name = game_name[10:12]
            dic_korean_team_name = {"SK" : "SSG","SS" : "삼성","OB":"두산","HT":"기아", "NC":"NC","WO":"키움","LG":"LG","KT":"kt","HH":"한화","LT":"롯데"}
            print(f"{dic_korean_team_name[away_team_name]}" " : " f"{dic_korean_team_name[home_team_name]} 완료")
            print("")
            print("")
            
        else:
            continue
        
    except requests.exceptions.RequestException as e:
        print(f"에러 발생: {e} | 제외된 URL: {url}")
        print(f"{df_total_game['g_id'][g]} 경기에서 문제 발생")

	#반복하면서 중첩
    df_etc_crawl = pd.concat([df_etc_crawl,df_naver_etc], axis=0, ignore_index=True)
    df_bat_crawl = pd.concat([df_bat_crawl, df_naver_bat], axis=0,ignore_index=True)
    df_pit_crawl = pd.concat([df_pit_crawl, df_naver_pit], axis=0,ignore_index=True)
    
    driver.quit()
```

.

.

```python
### 엑셀용 정리 ###

##3-1. 행 이름 설정
#투수는 행 개수 고정
df_pit_crawl.columns=['player_id','이름','구분','승홀세','이닝','피안타','실점','자책','4사구','삼진','피홈런','타자','타수','투구수','경기','승리','패전','세이브','평균자책']

#행 데이터 숫자 변환
df_pit_crawl['이닝']=df_pit_crawl['이닝'].str.replace('⅓','1/3')
df_pit_crawl['이닝']=df_pit_crawl['이닝'].str.replace('⅔','2/3')

int_columns_p = ['player_id','피안타','실점',
               '자책','4사구','삼진','피홈런','타자','타수','투구수','경기','승리','패전','세이브']
float_columns_p = ['평균자책']

for col in int_columns_p:
    df_pit_crawl[col] = df_pit_crawl[col].astype(int)
for col in float_columns_p:
    df_pit_crawl[col] = df_pit_crawl[col].astype(float)

### 투수는 여기서 완료
df_pit_final = df_pit_crawl

#타자는 그 날 연장 이닝에 따라 달라짐
total_columns = df_bat_crawl.shape[1]
fixed_columns = ['player_id','이름','포지션','타수','득점','안타','타점','홈런','볼넷','삼진','도루','타율']
numbered_columns = [str(i+1) for i in range(total_columns - len(fixed_columns))]
df_bat_crawl.columns = fixed_columns + numbered_columns

#고정 행 데이터 숫자형으로 변환
int_columns = ['player_id','타수','득점','안타','타점','홈런','볼넷','삼진','도루']
float_columns = ['타율']

for col in int_columns:
    df_bat_crawl[col] = df_bat_crawl[col].astype(int)
for col in float_columns:
    df_bat_crawl[col] = df_bat_crawl[col].astype(float)

##3-2. 한 이닝 두 타석 처리
#player_id ~~ 타율까지 타자 12개행
df_bat_info = df_bat_crawl.iloc[:,0:12]
df_bat_check = df_bat_crawl.iloc[:,12:]

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

### 타자는 여기서 완료
df_bat_final = pd.concat([df_bat_info, df_second_check],axis=1)

##3-3. 기타 항목 정리
df_etc_crawl['정리'] = df_etc_crawl['이름'].str.replace(r'\(.*?\)', '',regex=True)
df_etc_crawl['정리2'] = df_etc_crawl['정리'].str.replace(r'\d.*?호', '',regex=True)
contetns_name = df_etc_crawl['항목']
contents_data = df_etc_crawl['정리2'].str.split(',',expand=True)
df_etc_contents = pd.concat([contetns_name,contents_data],axis=1)
df_etc_contents = df_etc_contents.set_index('항목')

df_etc_contents_t = df_etc_contents.transpose()
col_list = list(df_etc_contents_t.columns.unique())

df_etc_final=pd.DataFrame()

for c in range(len(col_list)):
    c_data = df_etc_contents_t[[col_list[c]]]
    c_data_stack = c_data.stack().reset_index(drop=True)
    df_stack = pd.DataFrame(c_data_stack)
    df_stack.columns=[col_list[c]]
    df_etc_final = pd.concat([df_etc_final,df_stack], axis=1,join='outer')

record_all = ['견제사','결승타','도루','도루자','실책','주루사','포일','폭투','홈런','3루타','2루타','병살타']
record_in = list(df_etc_final.columns)
something_new = list(set((record_in))-set(record_all))

for col in something_new:
    df_etc_final.drop(columns=[col], inplace=True)
    
record_in = list(df_etc_final.columns)    
record_check = list(set(record_all)-set(record_in))

for record in record_check:
    df_etc_final[record]=None

### 기타는 여기서 완료
df_etc_final=df_etc_final.reindex(columns=record_all)


##4. 엑셀에 정리
import os
save_path = os.getcwd()  # 실행된 위치
file_name = f"{game_day}.xlsx"
file_path = os.path.join(save_path, file_name)

with pd.ExcelWriter(file_path) as writer:
    df_bat_final.to_excel(writer, sheet_name='네이버_타격', index=False)
    df_pit_final.to_excel(writer, sheet_name='네이버_피칭', index=False)
    df_etc_final.to_excel(writer, sheet_name='네이버_기타', index=False)


print("크롤링 완료")

time.sleep(5)
```

.

.

결과

![1](../../../../images/2025-06-06-naver_crawl_gpt/1.png)

![2](../../../../images/2025-06-06-naver_crawl_gpt/2.png)

![3](../../../../images/2025-06-06-naver_crawl_gpt/3.png)