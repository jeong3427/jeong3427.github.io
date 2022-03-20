---
layout: post

title:  "탭을 이동하며 크롤링"

categories : Python

tag : [python, 크롤링, selenium]
---





사이트 내에서 같은 행동을 반복해야 하는데

사이트의 로그인 값이 계속 갱신되어서 로그인을 반복해야 하는 이슈 발생

.

selenium에서 탭을 이동하는 방식으로 처리

.

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import pandas as pd
import time


driver = webdriver.Chrome()

#기본 처리
url = 'https://dashboard.gamepot.ntruss.com/vq5e0v7ura/c2f666eb-f192-4e32-84f1-1384462a0106/dashboard/analysis'

driver.get(url)
driver.implicitly_wait(10)


#게임팟 로그인
driver.find_element_by_id('username').send_keys("id")
driver.find_element_by_id('password').send_keys("password")
select = driver.find_element_by_xpath('//*[@id="root"]/div/div[2]/div/div/div/form/div[3]/div/div/span/button')
select.click()
driver.implicitly_wait(10)


#회원 목록으로 이동
select2 = driver.find_element_by_xpath('//*[@id="root"]/div/section/aside/div[1]/ul/li[8]/div')
select2.click()

driver.find_element_by_xpath('//*[@id="/:ncpId/:projectId/member$Menu"]/li[1]/a').click()
driver.find_element_by_xpath('//*[@id="root"]/div/section/section/main/div/div/div[2]/div/div/div/div[3]/div/div/div/div[2]/div/table/tbody/tr[21]/td[3]/span/a[1]').click()

time.sleep(3)


#삭제 처리할 id 리스트
id_list = pd.read_csv('gamepot_id.csv')


# 하나씩 제거
try:
    for i in id_list.index:
        driver.execute_script("window.open('');")
        driver.switch_to.window(driver.window_handles[1])
        base_url = 'https://dashboard.gamepot.ntruss.com/vq5e0v7ura/c2f666eb-f192-4e32-84f1-1384462a0106/member/list/detail/'
        id_index = id_list['id'][i]
        work_url = base_url + id_index
        driver.get(work_url)

        driver.find_element_by_xpath('//*[@id="root"]/div/section/section/main/div/div/div[2]/div/div/div/div[1]/div/button[3]').click()

        time.sleep(2)

        driver.find_element_by_xpath('/html/body/div[2]/div/div/div/div[2]/div/div[2]/button[2]').click()
        time.sleep(2)

        driver.close()
        driver.switch_to.window(driver.window_handles[0])
except:
    print("에러 발생")


print("정상 작동")

driver.quit()
```

.

.

driver.execute_script("window.open('');") 으로 새로운 탭을 열고 작업.

driver.close() 로 해당 탭을 닫는다

.

.

driver.close() 뒤에

driver.switch_to.window(driver.window_handles[0]) 처리를 하지 않으면 

다음과 같은 에러 메시지가 발생한다

.

selenium.common.exceptions.NoSuchWindowException: Message: no such window: target window already closed
from unknown error: web view not found

.

.

driver 상태는 이미 두 번째 탭인데  또 다시 두 번째 탭으로 이동하라는 이슈가 생겨서 발생

-> hanldes[0]으로 첫번째 탭으로 돌려놔야 해결

