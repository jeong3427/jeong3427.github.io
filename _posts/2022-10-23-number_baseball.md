---
layout: post
title:  "숫자 야구 게임"
categories : Python
tag : [python, 야구, 게임, 제작]
---



숫자 야구 게임(number baseball game) 제작해보기

.

**(str.isdigit(try_num)==False)** -> 숫자가 아닌 경우 체크

.

.

```python
import random
import os

print("----------------------------------------------")
print("0부터 9까지 서로 다른 세 자리 숫자를 입력하여")
print("위치와 숫자가 모두 일치하면 스트라이크,")
print("숫자만 일치하면 볼,")
print("모두 틀리면 아웃입니다.")
print("")

quiz = random.sample([x for x in range(9)], 3)
# print(quiz)

def try_num():
    try_num = input("숫자를 입력해주세요 : ")
    return try_num

def number_check(try_num):
    
    while ((len(try_num)!=3) or 
           (str.isdigit(try_num)==False) or 
           ((try_num[0]==try_num[1]) or (try_num[0]==try_num[2]) or (try_num[1]==try_num[2]))):
        try:
            print("----------------------------------------------")
            print("0부터 9까지 서로 다른 세 자리 숫자여야 합니다")
            print("")
            try_num = input("숫자를 입력해주세요 : ")
        except:
            pass
        
    three_digit = [int(i) for i in list(try_num)]
    
    return three_digit


def num_baseball(quiz, three_digit):
    
    s=0
    b=0
    
    for i in range(3):
        if quiz[i] in three_digit:
            if quiz[i] == three_digit[i]:
                s+=1
            else:
                b+=1
    
    result_list = [s, b]
    
    if sum(result_list)==0:
        print("아웃!")
    else:
        print(str(f'{result_list[0]}')+"스트라이크 "+str(f'{result_list[1]}')+"볼")
    
    return result_list


count = 1
result_list = num_baseball(quiz, number_check(try_num()))

while result_list[0]!=3:
    print("----------------------------------------------")
    
    result_list = num_baseball(quiz, number_check(try_num()))
    count+=1
    
    if result_list[0]==3:
        print("정답입니다!")
        print("시도 횟수는 " + str(f'{count}')+"회입니다.")
    
        break
    
else:
    print("정답입니다!")
    print("시도 횟수는 " + str(f'{count}')+"회입니다.")  
    
    
print("")
print("")

os.system('pause')
```

결과

![image-20221023232322430](../../../../images/2022-10-23-number_baseball/image-20221023232322430.png)

.

.

잘못된 케이스

![image-20221023232402188](../../../../images/2022-10-23-number_baseball/image-20221023232402188.png)
