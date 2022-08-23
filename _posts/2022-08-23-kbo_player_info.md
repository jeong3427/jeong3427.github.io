---
layout: post
title:  "kbo 등록 선수 정보 크롤링"
categories : Python
tag : [python, 크롤링, 야구]
---

.

.

동적 페이지 크롤링

kbo홈페이지의 선수 정보는 .aspx로 구성되어서 url 이 변경되지 않는다



request의 post를 사용하여 크롤링

.

.

```python
from math import ceil
from unittest import result
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.select import Select
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

import collections
if not hasattr(collections,'Callable'):
    collections.Callable = collections.abc.Callable
    
import re
import time
import pandas as pd
import requests
from bs4 import BeautifulSoup

url = 'https://www.koreabaseball.com/Player/Search.aspx'
driver = webdriver.Chrome()
driver.get(url)

wait = WebDriverWait(driver,10)

team_code=['OB','LT','SS','WO','HH','HT','KT','LG','NC','SK']

#최종 정보가 들어갈 리스트
player_total = []

#팀별로 총 선수가 몇인지 세는 함수 정의
def player_count(driver):
    rs = requests.Session()
    html = driver.page_source
    soup = BeautifulSoup(html, 'lxml')
    
    VIEWSTATE = soup.find('input',{'name':'__VIEWSTATE'})['value']
    VIEWSTATEGENERATOR = soup.find('input',{'name':'__VIEWSTATEGENERATOR'})['value']
    EVENTVALIDATION = soup.find('input',{'name':'__EVENTVALIDATION'})['value']
    
    data = {
        '__VIEWSTATE' : VIEWSTATE,
        '__VIEWSTATEGENERATOR' : VIEWSTATEGENERATOR,
        '__EVENTVALIDATION' : EVENTVALIDATION,
        'ctl00$ctl00$ctl00$cphContents$cphContents$cphContents$ddlTeam' : f'{team_code[t]}'    
    }

    result_html = requests.post(url, verify=False, data=data)
    result_soup = BeautifulSoup(result_html.content, 'lxml')
    
    total_count = result_soup.find('span',attrs={'class':'point'}).get_text()
    
    return total_count

#선수 정보 크롤링 하는 함수 정의
def player_crawl(driver):
    rs=requests.Session()
    
    #강제로 쉬는 시간 없으면 이전 페이지가 불러와지는 경우 있어서 추가
    time.sleep(1)
    
    html = driver.page_source
    soup = BeautifulSoup(html, 'lxml')    
    
    VIEWSTATE = soup.find('input',{'name':'__VIEWSTATE'})['value']
    VIEWSTATEGENERATOR = soup.find('input',{'name':'__VIEWSTATEGENERATOR'})['value']
    EVENTVALIDATION = soup.find('input',{'name':'__EVENTVALIDATION'})['value']
    
    data = {
        '__VIEWSTATE' : VIEWSTATE,
        '__VIEWSTATEGENERATOR' : VIEWSTATEGENERATOR,
        '__EVENTVALIDATION' : EVENTVALIDATION,
        'ctl00$ctl00$ctl00$cphContents$cphContents$cphContents$ddlTeam' : f'{team_code[t]}'    
    }
    
    result_html = requests.post(url, verify=False, data=data)
    result_soup = BeautifulSoup(result_html.content, 'lxml')
    
    player_info_list =[]
    
    player_info = result_soup.find_all('td')
    people_count = int(len(player_info)/7)
    
    for i in range(people_count):
        count = i*7
        player_id = re.sub(r'[^0-9]','',str(player_info[count+1])) #표에는 안 보이지만 player_id도 가져오기
        back_no = player_info[count].text
        player_name = player_info[count+1].text
        player_team = player_info[count+2].text
        player_pos = player_info[count+3].text
        player_birth = player_info[count+4].text
        player_physical = player_info[count+5].text
        player_school = player_info[count+6].text
        
        player_info_list.append(([player_id, back_no, player_name, player_team,
                                  player_pos, player_birth, player_physical, player_school]))
    
    #print(player_info_list)
    return player_info_list


# 10개 팀이 있으니 10번 반복
for t in range(10):
    select_team = Select(driver.find_element(By.ID,'cphContents_cphContents_cphContents_ddlTeam'))
    select_team.select_by_value(team_code[t])
    
    total_count = int(player_count(driver))
    
    page_num = ceil(total_count/20)
    
    result1 = player_crawl(driver)
    player_total.extend(result1)
    
    page=1
    
    while page<=page_num:
        try:
            page_button = wait.until(EC.element_to_be_clickable(
                (By.XPATH,'//*[@id="cphContents_cphContents_cphContents_ucPager_btnNo'+f'{page+1}'+'"]')))
            page_button.click()
            
            page+=1
            
            result2 = player_crawl(driver)
            
            if result1 != result2:
                player_total.extend(result2)
                
                if len(result2)<20:
                    break
                else:
                    page==page
                    
        except:
            break
        
player_final = pd.DataFrame(player_total)

player_final.columns = ['player_id','등번호','선수명','팀명','포지션','생년월일','체격','출신교']
player_final.to_csv('kbo_result.txt',index=False, header=True)

driver.quit()
```

.

.

텍스트 파일에서 엑셀로 옮기면 다음과 같다

![image-20220823230705268](../../../../images/2022-08-23-kbo_player_info/image-20220823230705268.png)

.

.

22년 8월23일 현재 872명 크롤링 

.

.

계속 1페이지만 긁어와지는 삽질 여러 번 끝에 time.sleep() 추가하여 해결 