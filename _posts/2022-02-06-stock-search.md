---
layout: post

title:  "슬랙 주가봇에 검색 기능 추가하기"

categories : Python

tag : [python, pandas, aws, 크롤링]
---





티커를 모두 외우기는 어려워서 티커 검색기를 간단하게 추가했다.

티커를 한글로 검색하는 적절한 사이트가 없어서 

그냥 네이버에서 "주식 이름"+ "주가" 검색했을 때 나오는 내용을 크롤링 했다.

.

.

네이버가 조금 특이하게 처리를 해뒀는데

국내 주식이면 'api_cs_wrap' 하위에  't_nm' 이라는 class로 종목 번호를 넣어두고

해외 주식이면 't_nm_s'라는 class로 티커를 넣어뒀다. 



```python
# -*- coding:utf-8 -*-
import json
import pandas as pd
from slack_sdk import WebClient
from tabulate import tabulate
from bs4 import BeautifulSoup
import requests

client = WebClient(token="토큰 코드")
response = {"isBase64Encoded": True, "statusCode": 200, "headers": {"X-Slack-No-Retry": 1}, "body": ""}

def lambda_handler(event, context):
    try:
        if int(event['headers']['x-slack-retry-num']) > 0:
            return response
    except:
        pass

    slack_body = event.get("body")
    slack_event = json.loads(slack_body)
    ch = slack_event['event']['channel']
    txt = slack_event['event']['text']


    if "challenge" in slack_event:
        return {
            'statusCode': 200,
            'body': slack_event.get("challenge")
        }

    if txt.startswith('!검색'):
        
        search_stock = txt[4:]
        
        try:
            search_url = "https://search.naver.com/search.naver?where=nexearch&sm=top_hty&fbm=0&ie=utf8&query=" + search_stock + "주가"

            response4 = requests.get(search_url)
            soup = BeautifulSoup(response4.content, 'html.parser')

            stock_area= soup.find('div',{'class':'api_cs_wrap'})

            search_stock_name = stock_area.find('span',{'class':'stk_nm'}).getText()
            search_ticker = stock_area.find('em',{'class':'t_nm_s'}).getText()
            client.chat_postMessage(channel = ch, text= "주식 이름은 : "+ f'{search_stock_name}'+"  티커는 : " + f'{search_ticker}', as_user=True)
    
        except:
            client.chat_postMessage(channel = ch, text= "오류입니다. 네이버에선 검색이 안 되네요", as_user=True)
```

![image-20220206192156627](../../../../img/2022-02-26-stock-search/image-20220206192156627.png)



![image-20220206192234205](../../../../img/2022-02-26-stock-search/image-20220206192234205.png)
