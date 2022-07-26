---
layout: post

title:  "슬랙 퍼블릭 채널 통계 내기"

categories : Python

tag : [python, pandas, slack]
---





슬랙 퍼블릭 채널에 사람이 몇 명이 있는지

각 인원은 몇 개의 퍼블릭 채널에 있는지 알아보기

.

.

```python
import pandas as pd
import json
from slack_sdk import WebClient


#슬랙 인증
slack_token = '토큰 코드'
client = WebClient(token=slack_token)


#슬랙 id와 이름으로 구성된 데이터프레임 준비
user_list = client.users_list()['members']
user_info=[]
for user in user_list :
    member_name= user['profile']['real_name']
    member_id= user['id']
    user_info.append((member_id,member_name))

df_user = pd.DataFrame(user_info)
df_user.columns=['id','name']



#채널 정보 탐색
channel_list = client.conversations_list(types=("public_channel" or "mpim" or"im"),exclude_archived=1, limit=1000)['channels']
channel_name = channel_list[0]['name']
channel_member_count = channel_list[0]['num_members']
channel_id = channel_list[0]['id']

channel_num = len(channel_list)


#채널 리스트, 인원수 출력
# for i in range(channel_num):
#     print(f'{i+1}' + ". " +  f"{channel_list[i]['name']}" + " (" + f"{channel_list[i]['num_members']}" + "명)" )



# 채널 id 준비
channel_id_list = []

for i in range(channel_num):
    channel_id_list.append(channel_list[i]['id'])

# print(channel_id_list)


# print(channel_id_list[0])
# check = client.conversations_members(channel='C04P77BEV')['members']


# 몇 개의 채널에 있는지 알기 위해 멤버 정보를 하나의 리스트에 이어 붙이기
member_list = []

for channel_id_info in channel_id_list:
    members = client.conversations_members(channel=channel_id_info)['members']
    member_list = member_list + members
    members =[]


# df_user 와 이어 붙이기 위해서 컬럼명 id로 설정
df_member = pd.DataFrame(member_list, columns=['id'])

#처음에 만든 이름 정보와 붙이기
df_member = df_member.merge(df_user, on='id', how='left')


#이름이 몇 개 있는지 카운트
df_result = df_member.groupby(['name']).agg({'id':'count'}).reset_index(drop=False)
df_result.columns=['name','count']

#개수대로 내림차순
df_result = df_result.sort_values(by='count', ascending=False).reset_index(drop=True)

print(df_result)
```





