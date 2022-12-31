---
layout: post

title:  "소수 관련 연습"

categories : Python

tag : [python, 알고리즘]
---

2023은 소수일까? 하는 물음에서 가볍게 작성

.

.

```python
#소수 구하기 기본
import statistics

def prime_number(num):
    num_list = []
    for i in range(2,num):
        if num%i==0:
            num_list.append(i)
        else:
            pass
    
    print(num_list)
    
    if len(num_list)==0:
        print("소수입니다.")
    else:
        print(num_list[0])
    
    if len(num_list)%2 != 0:
        print("제곱수입니다. "+f'{statistics.median(num_list)}')
        
        
        
prime_number(2023)
# 결과는 [7, 17, 119, 289]
```



statistics.median(x)를 하면 x의 중앙값이 나온다.

.

.

소수를 구할 때 몫이 짝을 지어서 나오는 부분을 고려하면, 큰 소수를 판별할 때는 제곱근까지만 판별하면 더 빠르다.

12를 예를 들면,

2 x 6 =12

3 x 4 = 12

4 x 3 = 12 

6 x 2 = 12



12의 제곱근인 3까지만 봐도 소수가 아닌 걸 알 수 있다.

```python
#소수 구하기 빠르게
import math

def prime_number_quick(num):
    num_list = []
    for i in range(2,int(math.sqrt(num))+1):
        
        if num%i==0:
            num_list.append(i)
        else:
            pass
        
    if len(num_list)==0:
        print("소수입니다.")
    else:
        print(num_list[0])
        
        
prime_number_quick(2023)
```

.

.

```python
#특정 수 이상일 때 최소의 소수 구하기
import math

def min_prime_number(num):
    
    for x in range(num, num*num+1):
        
        num_list =[]
        
        for k in range(2, int(math.sqrt(x))+1):
            if x % k ==0:
                num_list.append(k)
            else:
                pass
        
        if len(num_list) == 0:
            print(x)
            break
            
min_prime_number(2023)           
#결과는 2027
```

min_prime_number(100000000) 했을 때 결과는 100000007

.

.

```python
#제곱근까지만 구할 때와 기본일 때 속도 차이
import time

start = time.time()
prime_number_quick(100000007)
end = time.time()
print(f'{end - start:.5f} sec')

start = time.time()
prime_number(100000007)
end = time.time()
print(f'{end - start:.5f} sec')
```

![image-20221231214816525](../../../../images/2022-12-31-prime_number/image-20221231214816525.png)
