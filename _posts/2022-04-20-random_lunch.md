---
layout: post

title:  "슬랙 이모지에 반응한 사람들로 그루핑하기"

categories : Python

tag : [python, pandas, json]
---



슬랙 특정 채널에서 특정 단어로 시작하는 메시지에 이모지로 반응한 사람들 리스트를 불러와서,

3명씩 짝을 지어주고 그 결과를 슬랙에 다시 보내주는 코드

```python
import requests as rq
import pandas as pd
import json
from slack_sdk import WebClient
import os.path
import random
import re

#회사 사람들 이름 준비
slack_token = 'xoxb토큰코드'
client = WebClient(token=slack_token)

user_list = client.users_list()['members']
user_info=[]
for user in user_list :
    member_name= user['profile']['real_name'][1:]
    member_id= user['id']
    user_info.append((member_id,member_name))

df_user = pd.DataFrame(user_info)
df_user.columns=['id','name']



#메시지 정보 가져오기
msg_url = 'https://slack.com/api/conversations.history'
msg_headers = {'content-type': 'application/x-www-form-urlencoded'}

def msg_count(channel_code, count):
    msg_params = {
        'token': 'xoxb토큰코드',
        'channel': '채널코드',
        'limit': count
    }
    return msg_params

#어떤 채널에서, 메시지 몇 개 가져올지
msg_params= msg_count('채널코드',100)

msg_res = rq.get(url=msg_url, headers=msg_headers, params=msg_params)
msg_list = []

time_stamp = [i['ts'] for i in msg_res.json()['messages']]
text = [i['text'] for i in msg_res.json()['messages']]

df_text = pd.DataFrame((zip(text,time_stamp)))
df_text.columns=['text','time_stamp']


#메시지 특정하기 - 여기서는 *[베타 랜덤 으로 시작하는 가장 최근 메시지 하나만
ts_list =[]

for i in df_text.index:
    if df_text['text'][i].startswith('*[베타 랜덤'):
        ts_list.append(df_text['time_stamp'][i])
        if len(ts_list) > 0:
            break



#이모지 데이터
emj_data = []
for ts in ts_list:
    url = 'https://slack.com/api/reactions.get'
    head = {'content-type': 'application/x-www-form-urlencoded'}
    params = {
        'token': 'xoxb토큰코드',
        'channel': '채널코드',
        'timestamp': ts
    }
    res = rq.get(url=url, headers=head, params=params)
    # print(res.json())

    try:
        for i in res.json()['message']['reactions']:
            emj_data.append({'users': ','.join(i['users']), 'count': i['count']})
    except:
        pass

emj_df = pd.DataFrame(emj_data)


#이모지별로 체크한 사람들 한 줄로 만들기
df_group = emj_df.users.str.split(',')
df_group = df_group.apply(lambda x : pd.Series(x))
df_group= df_group.stack().reset_index(drop=True).to_frame('id')
df_group=df_group.drop_duplicates('id',keep='first')
member_count = len(df_group)

#회사 사람들 이름 붙이기
df_group.insert(1,'name',df_group['id'].map(df_user.set_index('id')['name']))
random_list = df_group['name'].values.tolist()


#랜덤 실행
random.shuffle(random_list)

# 몇 그룹 만들지 
set_num = member_count//3


#매칭 시작
match_list=[]

for i in range(set_num+1):
    try:
        match_list.append(random_list[:3])
        del random_list[:3]
    except:
        match_list.append(random_list)


# 메시지로 준비
message =[]
for i in range(set_num+1):
    if i < set_num:
        contents = f'{i+1}' + "그룹 : " + str(match_list[i])
        message.append(contents)
    else:
        contents = "깍두기 : " + str(match_list[i])
        message.append(contents)


#리스트 형식 탈피
def clean_text(inputString):
  text_rmv = re.sub('[-=+#/\?^.@*\"※~ㆍ!』‘|\(\)\`\'…》\”\“\’·]', ' ', inputString)
  text_rmv = text_rmv.rstrip(']')
  text_rmv = text_rmv.lstrip('[')
  return text_rmv


#슬랙에 메시지 보내기
slack_message = clean_text(str(message))

#어떤 채널로 
client.chat_postMessage(channel='채널코드', text=f'{slack_message}')

print("문제 없이 보냈습니다.")
```
