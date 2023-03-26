---
layout: post

title:  "파이썬 알고리즘 연습 02"

categories : Python

tag : [python, 알고리즘, 2진수, 2진법, 피보나치, 수열, 윤년]
---

### 1. 2진수 변환


```python
n = 18

result = ""

while n > 0:
    result = str(n % 2) + result #나머지를 문자열의 왼쪽에 추가해 나감
    n = n//2  # 2로 나눈 몫을 다시 대입
    
print(result)
```

    10010
    


```python
18//2
```




    9




```python
# base 진수법으로 바꾸기

def convert(n, base):
    result = ""
    while n>0:
        result = str(n % base) + result
        n = n//base
        #n //= base
    return result

n = 18

print(convert(n,2))
print(convert(n,3))
print(convert(n,10))
```

    10010
    200
    18
    

### 2. 10진수 변환

예를 들어 456은 4x100 + 5 x10 + 6x1


```python
n = '10010'

result =0

for i in range(len(n)):
    number = int(n[i])*(2**(len(n)-i-1))
    result = result+number

print(result)
```

    18
    


```python
# 파이썬에는 bin 이라는 함수가 2진수 변환

a = 18
print(bin(a))
print(type(a))
```

    0b10010
    <class 'int'>
    


```python
# n진수를 10진법으로 바꾸는 방법

b = '10010'
print(int(b,2))
print(type(b))
```

    18
    <class 'str'>
    


```python
# 맨 앞에 0b 를 붙이면 2진수로 인식

c = 0b10010
print(c)
print(type(c))
```

    18
    <class 'int'>
    

### 3. 피보나치 수열 만들기


```python
def fibonnacci(n):
    if (n==1) or (n==2):
        return 1
    return fibonnacci(n-2) + fibonnacci(n-1)
```


```python
print(fibonnacci(1))
print(fibonnacci(2))
print(fibonnacci(3))
print(fibonnacci(4))
print(fibonnacci(5))
```

    1
    1
    2
    3
    5
    


```python
print(fibonnacci(11))
```

    89
    


```python
# 메모이제이션으로 처리 속도 향상 시키기

memo = {1:1,2:1} # 사전에 조건 입력

def fibonnacci_memo(n):
    if n in memo:
        return memo[n] # 사전에 등록되어 있으면 그 값을 반환
    
    memo[n] = fibonnacci_memo(n-2) + fibonnacci_memo(n-1) # 사전에 등록되지 않았으면 계산하여 사전에 등록
    
    return memo[n]
```


```python
fibonnacci_memo(11)
```




    89




```python
# 그냥 리스트에 넣기

def fibonnacci_list(n):
    fib = [1,1]
    
    for i in range(2, n):
        fib.append(fib[i-2] + fib[i-1])
        
    return fib[n-1]
```


```python
fibonnacci_list(11)
```




    89



### 4. 윤년 계산
4로 나누어 떨어지는 해는 윤년이다, 하지만 100으로 나누어 떨어지고 400으로 나누어 떨어지지 않는 해는 윤년이 아니다


```python
def leap_year(year):
    if (year % 100 ==0) and (year % 400 !=0):
        return False
    elif (year % 4 ==0):
        return True
    else :
        return False
```


```python
print(leap_year(2019))
print(leap_year(2020))
print(leap_year(2000))
```

    False
    True
    True
    


```python
for i in range(2000,2051):
    check = leap_year(i)
    if check == True:
        print(str(i) + "년은 윤년입니다.")
    else:
        print(str(i) + "년은 윤년이 아닙니다.")
```

    2000년은 윤년입니다.
    2001년은 윤년이 아닙니다.
    2002년은 윤년이 아닙니다.
    2003년은 윤년이 아닙니다.
    2004년은 윤년입니다.
    2005년은 윤년이 아닙니다.
    2006년은 윤년이 아닙니다.
    2007년은 윤년이 아닙니다.
    2008년은 윤년입니다.
    2009년은 윤년이 아닙니다.
    2010년은 윤년이 아닙니다.
    2011년은 윤년이 아닙니다.
    2012년은 윤년입니다.
    2013년은 윤년이 아닙니다.
    2014년은 윤년이 아닙니다.
    2015년은 윤년이 아닙니다.
    2016년은 윤년입니다.
    2017년은 윤년이 아닙니다.
    2018년은 윤년이 아닙니다.
    2019년은 윤년이 아닙니다.
    2020년은 윤년입니다.
    2021년은 윤년이 아닙니다.
    2022년은 윤년이 아닙니다.
    2023년은 윤년이 아닙니다.
    2024년은 윤년입니다.
    2025년은 윤년이 아닙니다.
    2026년은 윤년이 아닙니다.
    2027년은 윤년이 아닙니다.
    2028년은 윤년입니다.
    2029년은 윤년이 아닙니다.
    2030년은 윤년이 아닙니다.
    2031년은 윤년이 아닙니다.
    2032년은 윤년입니다.
    2033년은 윤년이 아닙니다.
    2034년은 윤년이 아닙니다.
    2035년은 윤년이 아닙니다.
    2036년은 윤년입니다.
    2037년은 윤년이 아닙니다.
    2038년은 윤년이 아닙니다.
    2039년은 윤년이 아닙니다.
    2040년은 윤년입니다.
    2041년은 윤년이 아닙니다.
    2042년은 윤년이 아닙니다.
    2043년은 윤년이 아닙니다.
    2044년은 윤년입니다.
    2045년은 윤년이 아닙니다.
    2046년은 윤년이 아닙니다.
    2047년은 윤년이 아닙니다.
    2048년은 윤년입니다.
    2049년은 윤년이 아닙니다.
    2050년은 윤년이 아닙니다.
    
