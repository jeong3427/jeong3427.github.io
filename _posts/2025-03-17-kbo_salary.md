---
layout: post
title:  "kbo 전체 연봉 크롤링"
categories : Python
tag : [python, 크롤링, 야구, chatGPT, lambda]
---





### 1. 전체 선수 정보 크롤링

![image-20250317091727988](../../../../images/2025-03-17-kbo_salary/image-20250317091727988.png)

kbo 선수 조회 페이지에서 팀을 선택하면

![image-20250317091813019](../../../../images/2025-03-17-kbo_salary/image-20250317091813019.png)

화면에 표시되지는 않지만 a href 태그 끝에 player_id 가 보인다



```python
from math import ceil
import re
import time
import pandas as pd
import requests
import collections
if not hasattr(collections, 'Callable'):
    collections.Callable = collections.abc.Callable
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support.select import Select
from selenium.webdriver.support import expected_conditions as EC

# SSL 경고 메시지 숨기기
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# 웹 드라이버 설정
options = ChromeOptions()
service = ChromeService(executable_path=ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)
driver.get("https://www.koreabaseball.com/Player/Search.aspx")

# WebDriverWait 설정
wait = WebDriverWait(driver, 10)

# KBO 팀 코드 리스트
team_codes = ['OB', 'LT', 'SS', 'WO', 'HH', 'HT', 'KT', 'LG', 'NC', 'SK']

# 전체 선수 데이터 리스트
player_total = []

def get_viewstate(driver):
    """현재 페이지의 VIEWSTATE 관련 정보를 가져오는 함수"""
    html = driver.page_source
    soup = BeautifulSoup(html, 'lxml')
    
    return {
        '__VIEWSTATE': soup.find('input', {'name': '__VIEWSTATE'})['value'],
        '__VIEWSTATEGENERATOR': soup.find('input', {'name': '__VIEWSTATEGENERATOR'})['value'],
        '__EVENTVALIDATION': soup.find('input', {'name': '__EVENTVALIDATION'})['value'],
    }

def get_player_count(driver, team_code):
    """선수 총 인원 수 가져오기"""
    viewstate_data = get_viewstate(driver)
    
    payload = {
        **viewstate_data,
        'ctl00$ctl00$ctl00$cphContents$cphContents$cphContents$ddlTeam': team_code
    }

    response = requests.post(driver.current_url, data=payload, verify=False)
    soup = BeautifulSoup(response.content, 'lxml')

    total_count = soup.find('span', class_='point')
    return int(total_count.text) if total_count else 0

def crawl_players(driver, team_code):
    """현재 페이지에서 선수 정보를 크롤링하는 함수"""
    time.sleep(1)  # 페이지 로딩 대기

    viewstate_data = get_viewstate(driver)

    payload = {
        **viewstate_data,
        'ctl00$ctl00$ctl00$cphContents$cphContents$cphContents$ddlTeam': team_code
    }

    response = requests.post(driver.current_url, data=payload, verify=False)
    soup = BeautifulSoup(response.content, 'lxml')

    players = []
    player_data = soup.find_all('td')
    player_count = len(player_data) // 7  # 한 선수당 7개의 데이터 존재

    for i in range(player_count):
        idx = i * 7
        player_id = re.sub(r'[^0-9]', '', str(player_data[idx + 1]))
        players.append([
            player_id,
            player_data[idx].text.strip(),  # 등번호
            player_data[idx + 1].text.strip(),  # 선수명
            player_data[idx + 2].text.strip(),  # 팀명
            player_data[idx + 3].text.strip(),  # 포지션
            player_data[idx + 4].text.strip(),  # 생년월일
            player_data[idx + 5].text.strip(),  # 체격
            player_data[idx + 6].text.strip()  # 출신교
        ])
    
    return players

# 팀별 선수 크롤링 진행
for team_code in team_codes:
    select_team = Select(driver.find_element(By.ID, 'cphContents_cphContents_cphContents_ddlTeam'))
    select_team.select_by_value(team_code)

    total_count = get_player_count(driver, team_code)
    total_pages = ceil(total_count / 20)

    # 첫 페이지 크롤링
    player_total.extend(crawl_players(driver, team_code))

    # 페이지네이션 크롤링
    for page in range(2, total_pages + 1):
        try:
            page_button = wait.until(EC.element_to_be_clickable(
                (By.XPATH, f'//*[@id="cphContents_cphContents_cphContents_ucPager_btnNo{page}"]')
            ))
            page_button.click()
            time.sleep(1)  # 페이지 로딩 대기

            new_players = crawl_players(driver, team_code)
            if new_players:
                player_total.extend(new_players)
            else:
                break  # 데이터가 더 이상 없으면 중단
        except Exception as e:
            print(f"페이지 {page} 크롤링 중 오류 발생: {e}")
            break

# 데이터프레임 생성 및 엑셀 저장
player_df = pd.DataFrame(player_total, columns=['player_id', '등번호', '선수명', '팀명', '포지션', '생년월일', '체격', '출신교'])
player_df.to_excel('kbo_players.xlsx', index=False, encoding='utf-8-sig')

# 드라이버 종료
driver.quit()

```

![image-20250317091940042](../../../../images/2025-03-17-kbo_salary/image-20250317091940042.png)

결과물



### 2. 연봉 정보 크롤링

![image-20250317092006804](../../../../images/2025-03-17-kbo_salary/image-20250317092006804.png)

연봉 정보는 페이지 끝에 player_id를 바꿔가면서 계속 크롤링을 해야 한다.

chatGPT를 활용해서 크롤링 안정성을 올린 코드.

```python
#앞서 크롤링한 선수 정보에서 player_id 리스트 가져오기
df_player = pd.read_excel('kbo_players.xlsx',sheet_name='Sheet1')
df_player_id=df_player['player_id']

from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options as ChromeOptions
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
import time

# 크롬 드라이버 설정
options = ChromeOptions()
options.add_argument("--headless")  # 브라우저 창을 띄우지 않음 (더 안정적)
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")

service = ChromeService(executable_path=ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)

# 크롤링할 선수 ID 리스트
salary_list = []
kbo_url = "https://www.koreabaseball.com/Record/Player/HitterDetail/Basic.aspx?playerId="

for i, player_id in enumerate(df_player_id):
    try:
        driver.get(kbo_url + str(player_id))

        # 요소가 로딩될 때까지 기다림
        salary_element = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.ID, "cphContents_cphContents_cphContents_playerProfile_lblSalary"))
        )
        salary = salary_element.text

    except Exception as e:
        print(f"Error at index {i}, player_id {player_id}: {e}")
        salary = "N/A"  # 오류 발생 시 기본값 설정
    
    salary_list.append(salary)
    print(f"Processed {i+1}/{len(df_player_id)}: Player ID {player_id}, Salary: {salary}")

    time.sleep(0.5)  # 너무 빠르게 요청하지 않도록 딜레이 추가

# 크롬 드라이버 종료
driver.quit()
```

![image-20250317092158958](../../../../images/2025-03-17-kbo_salary/image-20250317092158958.png)

### 3. pandas에서 정리

![image-20250317092413676](../../../../images/2025-03-17-kbo_salary/image-20250317092413676.png)

연봉 정보를 앞서 만든 엑셀 옆에 붙였는데 달러와 한화가 섞여있다. 

```python
df_player["currency"] = df_player["연봉"].apply(lambda x: "KRW" if "만원" in str(x) else "USD")
df_player["salary"] = df_player["연봉"].astype(str).str.replace("만원", "").str.replace("달러", "").astype(int)
df_player["최종"] = df_player.apply(lambda row: row["salary"] * 10000 if row["currency"] == "KRW" else row["salary"] * 1450, axis=1)
```

![image-20250317092535366](../../../../images/2025-03-17-kbo_salary/image-20250317092535366.png)