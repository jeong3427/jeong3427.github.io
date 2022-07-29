---
layout: post

title:  "연차인 사람 슬랙 알림 봇"

categories : Python

tag : [python, pandas, slack]
---



flex 로그인 + 슬랙 알림

.

.

```python
from bs4 import BeautifulSoup
import pandas as pd
import requests
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import re




driver = webdriver.Chrome('chromedriver.exe')


# 플렉스 로그인
login_url ='https://flex.team/auth/login'
headers = {"upgrade-insecure-requests": "1", "User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:79.0) Gecko/20100101 Firefox/79.0"}

driver.get(login_url)
driver.implicitly_wait(5)

email = 'id'
password = 'pw'


wait = WebDriverWait(driver, 100)

driver.find_element_by_id('radix-1').send_keys(email)


continue_button = wait.until(EC.element_to_be_clickable((By.XPATH,'//*[@id="__next"]/main/div/div/div/article/div/div[2]/div/div/form/div/label/div/div[2]/button')))
continue_button.click()

wait.until(EC.presence_of_all_elements_located((By.ID,'radix-9')))

driver.find_element_by_id('radix-9').send_keys(password)

login_button = wait.until(EC.element_to_be_clickable((By.XPATH,'//*[@id="__next"]/main/div/div/div/article/div/div[2]/form/div/button[1]')))
login_button.click()


# 로그인 완료 후 휴가 정보 보기
more_info_button = wait.until(EC.element_to_be_clickable((By.XPATH,'//*[@id="app-shell-root"]/div/div/div/div/main/section/div/div/div[2]/ul/li[4]/div/div[2]/button')))
more_info_button.click()


# 전체 휴가자 리스트 작성
vacation_list = []

for i in range(2,7):
    date_list = driver.find_element_by_xpath('/html/body/div[5]/div/div[2]/div/div/div[2]/div/div/div/div['+ f'{i}' +']/div[2]/div[1]/span').text
    name_list = driver.find_element_by_xpath('/html/body/div[5]/div/div[2]/div/div/div[2]/div/div/div/div['+  f'{i}'+']/div[2]/div[2]/ul').text
    vacation_list.append((date_list, name_list))


df_vacation = pd.DataFrame(vacation_list)
df_vacation.columns = ['date','list']

df_vacation['count'] = df_vacation.apply(lambda x: len(re.findall('\n', x['list']))+1, axis=1)


# 오늘 날짜 정보 가져오기
result_html = driver.page_source
result_soup = BeautifulSoup(result_html, 'html.parser')
today_info = result_soup.find('span', {'class':'c-lnuBr c-lnuBr-dvlTtX-type-today'}).get_text()


check_date = df_vacation.index[df_vacation['date']==today_info].tolist()
person_num = df_vacation['count'][check_date[0]]


# 오늘 휴가자 
today_vacation_list =[]

for p in range(person_num):
    name = driver.find_element_by_xpath('/html/body/div[5]/div/div[2]/div/div/div[2]/div/div/div/div['+f'{check_date[0]+2}' +']/div[2]/div[2]/ul/li['+f'{1+p}'+']/div/span').text
    today_vacation_list.append(name)


df_today_vacation = pd.DataFrame(today_vacation_list, columns=[''])

driver.quit()



final_person_num = df_today_vacation[df_today_vacation[''].str.contains('휴가')][''].str[:4].drop_duplicates(keep='first').count()



from slack_sdk import WebClient
from tabulate import tabulate


#슬랙 인증
client = WebClient(token="토큰 코드")
response = {"isBase64Encoded": True, "statusCode": 200, "headers": {"X-Slack-No-Retry": 1}, "body": ""}


message = df_today_vacation
slack_post = tabulate(message, headers ='keys', tablefmt='presto', showindex=False)
client.chat_postMessage(channel = '채널 코드', text= '오늘은 '+ f'{today_info}'+'일입니다. 휴가자는 '+ f'{final_person_num}'+'명입니다.', as_user=True)
client.chat_postMessage(channel = '채널 코드', text= slack_post, as_user=True)



# print("문제 없이 보냈습니다.")

```

.

약간 쓸데 없는 크롤링이 있는 편

.

자동 배치 파일

- 윈도우 스케쥴러에서 실행

```python
@echo off
"C:\Users\GD-hjim\anaconda3\python.exe" "C:\Users\GD-hjim\Desktop\vscode\flex_vacation.py"
pause
```

