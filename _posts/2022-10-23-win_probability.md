---
layout: post
title:  "뒷 세자리만 주어질 때 당첨 확률 구하기"
categories : Python
tag : [python, 확률, lambda]
---



***123

***987

과 같이 6자리 숫자가 랜덤으로 부여되고

6자리 숫자의 합이 가장 큰 사람이 당첨이 되고 

6자리 숫자 합이 같을 경우 전체 숫자가 큰 사람이 당첨되는 시스템.

.

하지만 유저에게는 뒷 자리 3개만 보여지는 상황에서 

뒷 자리 숫자와 신청 인원, 당첨 인원을 보고 당첨 확률이 얼마일지 구하는 코드.

.

.

 (6자리 합, 6자리 숫자) 형식으로 zip과 list 함수를 사용하여 결과물 리스트 생성.

**number_list.sort(key=lambda x:(x[0], x[1]), reverse=True)** 에서 6자리 합으로 줄 세우고, 6자리 숫자로 줄 세웠다.

```python
#내 정보
my_info = list(zip(number_sum, six_digit))

#다른 유저 정보
number_sum.append(hidden_1 + hidden_2 + hidden_3 + hidden_4 + hidden_5 + hidden_6)
six_digit.append(hidden_1*100000 + hidden_2*10000 + hidden_3*1000 + hidden_4*100 + hidden_5*10 + hidden_6*1)     

info = list(zip(number_sum, six_digit))

#key의 첫번째 값으로 줄 세우고, 두 번째 값으로 줄 세우고
number_list.sort(key=lambda x:(x[0], x[1]), reverse=True)
```

![image-20221023225150505](../../../../images/2022-10-23-win_probability/image-20221023225150505.png)

.

.

그 후에는 1천 번 실행으로 확률 계산

당첨되면 1, 떨어지면 0으로 1천 번 실행시 통계로 확률 계산.

.

```python
import random


my_num = input("뒷자리를 입력하세요 : ")
win = input("당첨 인원을 입력하세요 : ")
total = input("신청자 수를 입력하세요 : ")

def mine(my_num):
    my_info = []
    
    first = int(my_num[0])
    second = int(my_num[1])
    third = int(my_num[2])
    
    hidden_1 = random.randint(0,9)
    hidden_2 = random.randint(0,9)
    hidden_3 = random.randint(0,9)
    
    number_sum = []
    six_digit = []
    
    number_sum.append(first + second + third + hidden_1 + hidden_2 + hidden_3)
    six_digit.append(hidden_1*100000 + hidden_2*10000 + hidden_3*1000 + first*100 + second*10 + third*1)
    
    my_info = list(zip(number_sum, six_digit))
    # print(my_info)
    return my_info

print(mine(my_num))

def lotto(total):
    number_list = []
    
    for try_member in range(int(total)-1):
        
        hidden_1 = random.randint(0,9)
        hidden_2 = random.randint(0,9)
        hidden_3 = random.randint(0,9)
        hidden_4 = random.randint(0,9)
        hidden_5 = random.randint(0,9)
        hidden_6 = random.randint(0,9)

        number_sum = []
        six_digit = []
        
        number_sum.append(hidden_1 + hidden_2 + hidden_3 + hidden_4 + hidden_5 + hidden_6)
        six_digit.append(hidden_1*100000 + hidden_2*10000 + hidden_3*1000 + hidden_4*100 + hidden_5*10 + hidden_6*1)     
            
        info = list(zip(number_sum, six_digit))
        number_list.append(info[0])
        
    number_list.sort(key=lambda x:(x[0], x[1]), reverse=True)
    # print(number_list)
    return number_list

print(lotto(total))

def check(my_info, number_list, win):
    ranking = 0
    success = 1
    fail = 0
    
    for i in range(len(number_list)):
        
        ranking+=1
        
        if int(my_info[0][0]) > int(number_list[i][0]):
            break
        elif int(my_info[0][0]) == int(number_list[i][0]):
            if int(my_info[0][1]) >= int(number_list[i][1]):
                break
            else:
                ranking+=1
                break
            
    if ranking <= int(win):
        return success
    else:
        return fail
    
    
result_list = []

for i in range(1000):
    
    result = check(mine(my_num),lotto(total),win)
    
    result_list.append(result)
    
mean = round((sum(result_list)/len(result_list))*100,3)

print("당첨될 확률은 "+str(mean)+"%입니다.") 
```

.

결과

![image-20221023225824404](../../../../images/2022-10-23-win_probability/image-20221023225824404.png)

.

.

단점은, 전체 정보를 input에서 받은 신청자 수 전체로 생성하기 때문에

1만 명 신청 중 3,000등 당첨 같은 큰 수일 때는 시간이 오래 걸리게 된다.

.

적당히 100명 중 30명 당첨 같은 형식으로 깎아서 사용 필요

