---
layout: post

title:  "슬랙 주가봇에 국내 주식 추가하기"

categories : Python

tag : [python, pandas, aws, 크롤링]
---



해외 주식은 yfinance 라이브러리를 사용하지만

국내주식은 야후에서 검색 시 데이터가 하루치밖에 안 나올 때가 있다.



그래서 국내 주식은 네이버 검색 후 크롤링 방식을 쓰기로 결정


보통 크롤링은 selenium을 쓰지만 
aws lambda에서 selenium을 쓰려니 여간 어려운 게 아니어서 모두 beautifulsoup으로 처리했다.


```python
# -*- coding:utf-8 -*-
import requests
from bs4 import BeautifulSoup
import pandas as pd
from slack_sdk import WebClient
from tabulate import tabulate
import os

client = WebClient(token="토큰 코드")
response = {"isBase64Encoded": True, "statusCode": 200, "headers": {"X-Slack-No-Retry": 1}, "body": ""}


stock = "펄어비스"
search_url = "https://search.naver.com/search.naver?where=nexearch&sm=top_hty&fbm=0&ie=utf8&query=" + stock + "주가"

response = requests.get(search_url)
soup = BeautifulSoup(response.content, 'html.parser')

stock_area= soup.find('div',{'class':'api_cs_wrap'})

kr_stock_name = stock_area.find('span',{'class':'stk_nm'}).getText()
kr_ticker = stock_area.find('em',{'class':'t_nm'}).getText()
print(kr_stock_name)
print(kr_ticker)

stock_url = "https://finance.naver.com/item/sise.nhn?code=" + kr_ticker


response2 = requests.get(stock_url)
html = response2.content.decode('euc-kr','replace')
soup2 = BeautifulSoup(html, 'html.parser')

ks_or_kq = soup2.find('img')['class']


if "q" in str(ks_or_kq):
    stock_type = '코스닥'
else :
    stock_type ='코스피'
    
print(stock_type)


stock_data = pd.DataFrame()

for page in range(6):
    df_data = pd.DataFrame()
    chart_url = "https://finance.naver.com/item/sise_day.nhn?code=" + kr_ticker + "&page=" + f'{page+1}'
    req = requests.get(chart_url, headers = {'User-Agent' : 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.162 Safari/537.36'})
    df_data= pd.read_html(req.text, encoding="euc-kr")[0]
    stock_data = stock_data.append(df_data)

stock_data.drop(stock_data[stock_data['날짜'].isna()].index, inplace=True)
stock_data=stock_data.reset_index(drop=True)
stock_data= stock_data.sort_index(ascending=False).reset_index(drop=True)


import matplotlib.pyplot as plt
from mpl_finance import candlestick_ohlc
import matplotlib.dates as mpl_dates

stock_data['전날'] = stock_data['종가'].shift(1)
stock_data['전날 대비'] = ((stock_data['종가']-stock_data['전날'])/stock_data['전날']).apply('{:.2%}'.format)
data_5d=stock_data[-5:]
data_5d=data_5d.loc[:,['날짜','시가','고가','종가','전날 대비','거래량']].reset_index(drop=True)
print(data_5d)

text_graph= tabulate(data_5d, headers ='keys', tablefmt='presto', showindex=False)
client.chat_postMessage(channel = "채널명", text= "주식 이름은 : "+ f'{kr_stock_name}'+"  종목은 : "+f'{stock_type}' + " 티커는 : " + f'{kr_ticker}', as_user=True)
client.chat_postMessage(channel = "채널명", text= text_graph, as_user=True)

plt.style.use('ggplot')
chart_data = stock_data.reset_index(drop=False)
ohlc = chart_data.loc[:, ['날짜', '시가', '고가', '저가', '종가']]
ohlc['날짜'] = pd.to_datetime(ohlc['날짜'])
ohlc['날짜'] = ohlc['날짜'].apply(mpl_dates.date2num)
ohlc = ohlc.astype(float)

# 표 만들기 - 상승은 빨간색, 하강은 파랑색
fig, ax = plt.subplots()
candlestick_ohlc(ax, ohlc.values, width=0.6, colorup='red', colordown='blue', alpha=1)

# 레이블과 제목
ax.set_xlabel('Date')
ax.set_ylabel('Price')
fig.suptitle(" Recent 3 Months Close")

# date 표기 처리 및 표 최대로 키우기
date_format = mpl_dates.DateFormatter('%Y-%m-%d')
ax.xaxis.set_major_formatter(date_format)
fig.autofmt_xdate()
fig.tight_layout()

plt.savefig("stock_image_kr.png")

client.files_upload(file="stock_image_kr.png",channels="채널명")
```

![image-20220206190008631](../../../../img/2022-02-06-naver-stock/image-20220206190008631.png)

---

.

.

문제는 aws lambda에서 샐행해보니

df_data= pd.read_html(req.text, encoding="euc-kr")[0]

이 부분이 작동을 하지 않았다.



read_html 을 못 하는 것으로 보였는데

어찌됐든 편한 방법을 쓰지 못하게 되어서 무식하게 크롤링 하는 방향으로 수정하였다.

```python
# -*- coding:utf-8 -*-
import json
import pandas as pd
import matplotlib.pyplot as plt
from mpl_finance import candlestick_ohlc
import matplotlib.dates as mpl_dates
from slack_sdk import WebClient
from tabulate import tabulate
import os
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
    
            
    if txt.startswith('!주식'):
            
        stock = txt[4:]

        try:
            search_url = "https://search.naver.com/search.naver?where=nexearch&sm=top_hty&fbm=0&ie=utf8&query=" + f'{stock}' + "주가"
            
            response2 = requests.get(search_url)
            
            soup = BeautifulSoup(response2.content, 'html.parser')



            stock_area= soup.find('div',{'class':'api_cs_wrap'})
            

            kr_stock_name = stock_area.find('span',{'class':'stk_nm'}).getText()
            
            kr_ticker = stock_area.find('em',{'class':'t_nm'}).getText()
            
            stock_url = "https://finance.naver.com/item/sise.nhn?code=" + f'{kr_ticker}'
            

            response3 = requests.get(stock_url)
            

            html = response3.content.decode('euc-kr','replace')
            
            soup2 = BeautifulSoup(html, 'html.parser')

            ks_or_kq = soup2.find('img')['class']


            if "q" in str(ks_or_kq):
                stock_type = '코스닥'
            else :
                stock_type ='코스피'
            
            client.chat_postMessage(channel = ch, text= "주식 이름은 : "+ f'{kr_stock_name}'+"  종목은 : "+f'{stock_type}' + " 티커는 : " + f'{kr_ticker}', as_user=True)
            

            stock_data = pd.DataFrame(columns=['날짜','종가','시가','고가','저가','거래량'])

            for page in range(6):
                chart_url = "https://finance.naver.com/item/sise_day.nhn?code=" + kr_ticker + "&page=" + f'{page+1}'
                html = requests.get(chart_url, headers = {"upgrade-insecure-requests": "1", "User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:79.0) Gecko/20100101 Firefox/79.0"}).text
                soup = BeautifulSoup(html, 'html.parser')

                table_data = soup.find('table',{"class":"type2"})


                range_num = len(table_data.find_all('td'))

                df = pd.DataFrame()

                for i in range(0,range_num):
                    sample=[]
                    num_data = table_data.find_all('td')[i].text
                    sample.append(num_data)
                    df = df.append(sample)


                df.columns=['data']
                df2=df.reset_index(drop=True)
                del_con1 = df2[df2['data'].str.contains('\n')].index
                del_con2 = df2[(df2['data'].str.len())==0].index
                df2.drop(del_con1, inplace=True)
                df2.drop(del_con2, inplace=True)
                df2 = df2.reset_index(drop=True)

                df_final=pd.DataFrame()
                df_final=pd.DataFrame(columns=['날짜','종가','시가','고가','저가','거래량'])

                for i in range(10):
                    a=i*6
                    df_data=pd.DataFrame()
                    df_data = df2[a:a+6].transpose()
                    df_data.columns=['날짜','종가','시가','고가','저가','거래량']
                    df_final=df_final.append(df_data)


                df_final=df_final.reset_index(drop=True)
                stock_data = stock_data.append(df_final) 

            stock_data=stock_data.reset_index(drop=True)
            
            stock_data= stock_data.sort_index(ascending=False).reset_index(drop=True)
        
            stock_data=stock_data.drop_duplicates(['날짜'],keep='first')
            
            if len(stock_data.index)<10:
                stock_data=stock_data.loc[1:]

            stock_data=stock_data.reset_index(drop=True)
            
            stock_data['날짜'] = pd.to_datetime(stock_data['날짜'])
            stock_data['날짜'] = stock_data['날짜'].dt.strftime('%Y-%m-%d')
            
            stock_data['종가'] = stock_data['종가'].astype(str)
            stock_data['시가'] = stock_data['시가'].astype(str)
            stock_data['고가'] = stock_data['고가'].astype(str)
            stock_data['저가'] = stock_data['저가'].astype(str)
            stock_data['거래량'] = stock_data['거래량'].astype(str)
            
            
            stock_data['종가'] = stock_data['종가'].apply(lambda x: x.replace(',', ''))
            stock_data['시가'] = stock_data['시가'].apply(lambda x: x.replace(',', ''))
            stock_data['고가'] = stock_data['고가'].apply(lambda x: x.replace(',', ''))
            stock_data['저가'] = stock_data['저가'].apply(lambda x: x.replace(',', ''))
            stock_data['거래량'] = stock_data['거래량'].apply(lambda x: x.replace(',', ''))
            
            
            stock_data['종가'] = pd.to_numeric(stock_data['종가'])
            stock_data['시가'] = pd.to_numeric(stock_data['시가'])
            stock_data['고가'] = pd.to_numeric(stock_data['고가'])
            stock_data['저가'] = pd.to_numeric(stock_data['저가'])
            stock_data['거래량'] = pd.to_numeric(stock_data['거래량'])
            
            stock_data['전날'] = stock_data['종가'].shift(1)
            
            
            stock_data['전날 대비'] = ((stock_data['종가']-stock_data['전날'])/stock_data['전날']).apply('{:.2%}'.format)
            data_5d=stock_data[-5:]
            data_5d=data_5d.loc[:,['날짜','시가','고가','종가','전날 대비','거래량']].reset_index(drop=True)
        

            text_graph= tabulate(data_5d, headers ='keys', tablefmt='presto', showindex=False)
            client.chat_postMessage(channel = ch, text= text_graph, as_user=True)

            plt.style.use('ggplot')
            chart_data = stock_data.reset_index(drop=False)
            ohlc = chart_data.loc[:, ['날짜', '시가', '고가', '저가', '종가']]
            ohlc['날짜'] = pd.to_datetime(ohlc['날짜'])
            ohlc['날짜'] = ohlc['날짜'].apply(mpl_dates.date2num)
            ohlc = ohlc.astype(float)

            # 표 만들기 - 상승은 빨간색, 하강은 파랑색
            fig, ax = plt.subplots()
            candlestick_ohlc(ax, ohlc.values, width=0.6, colorup='red', colordown='blue', alpha=1)

            # 레이블과 제목
            ax.set_xlabel('Date')
            ax.set_ylabel('Price')
            fig.suptitle(" Recent 3 Months Close")

            # date 표기 처리 및 표 최대로 키우기
            date_format = mpl_dates.DateFormatter('%Y-%m-%d')
            ax.xaxis.set_major_formatter(date_format)
            fig.autofmt_xdate()
            fig.tight_layout()

            os.chdir('/tmp')
            plt.savefig("stock_image_kr.png")

            client.files_upload(file="stock_image_kr.png",channels=ch)

        except:
            client.chat_postMessage(channel = ch, text= "검색어를 확인해주세요", as_user=True)

```

![image-20220206190547629](../../../../img/2022-02-06-naver-stock/image-20220206190547629.png)

