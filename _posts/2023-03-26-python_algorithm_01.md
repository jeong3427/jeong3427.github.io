---
layout: post

title:  "파이썬 알고리즘 연습 01"

categories : Python

tag : [python, 알고리즘, 순서도, 클래스, 객체, 상속]
---


### 1. 클래스를 정의하여 인스턴스 생성
파이썬으로 클래스를 만들 때는 클래스명을 대문자로 시작,
메서드를 정의할 때는 인수에 self를 반드시 작성


```python
class User:#User 클래스 정의 - 반드시 대문자로 시작
    def __init__(self, name, password): #생성자(__init__) 정의
        self.name = name
        self.password = password
    def login(self, password): #로그인 메서드 정의
        if self.password == password:
            return True
        else:
            return False
        
    def logout(self): #로그아웃 메서드 정의
        print('logout')

```


```python
a = User('admin', 'password') #사용자 이름 admin, 암호 password 인 사용자 만들기

if a.login('password'):
    a.logout()
```

    logout



```python
a.name
```




    'admin'




```python
a.password
```




    'password'



### 2.클래스 상속


```python
class GuestUser(User): #User 클래스를 상속하여 GuestUser 클래스를 정의
    def __init__(self):
        super().__init__('guest', 'guest')
```


```python
b = GuestUser()
if b.login('guest'):
    b.logout()
```

    logout



```python
c = GuestUser()
if c.login('hi'):
    c.logout()
```


```python
c.name
```




    'guest'



### 3. 순서도, 알고리즘

<img src="../../../../images/2023-03-26-python_algorithm_01/KakaoTalk_20230326_205033648.jpg" alt="KakaoTalk_20230326_205033648" style="zoom:67%;" />


```python
# 1부터 50까지의 숫자를 순서대로 출력하는데, 
# 3의 배수일 때는 숫자대신 'Fizz', 5의 배수일 때는 'Buzz', 3과 5의 공배수일 때는 'FizzBuzz'를 출력
```


```python
for i in range(1,51):
    if i%15==0:
        print('FizzBuzz', end=' ')
    elif i%5==0:
        print('Buzz', end=' ')
    elif i%3 ==0 :
        print('Fizz', end=' ')
    else:
        print(i, end=' ')
```

    1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz 16 17 Fizz 19 Buzz Fizz 22 23 Fizz Buzz 26 Fizz 28 29 FizzBuzz 31 32 Fizz 34 Buzz Fizz 37 38 Fizz Buzz 41 Fizz 43 44 FizzBuzz 46 47 Fizz 49 Buzz 


```python
for i in range(1,51):
    if (i%3==0) and (i%5==0):
        print('FizzBuzz', end=' ')
    elif i%5==0:
        print('Buzz', end=' ')
    elif i%3 ==0 :
        print('Fizz', end=' ')
    else:
        print(i, end=' ')
```

    1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz 16 17 Fizz 19 Buzz Fizz 22 23 Fizz Buzz 26 Fizz 28 29 FizzBuzz 31 32 Fizz 34 Buzz Fizz 37 38 Fizz Buzz 41 Fizz 43 44 FizzBuzz 46 47 Fizz 49 Buzz 

<img src="../../../../images/2023-03-26-python_algorithm_01/KakaoTalk_20230326_205033648_01.jpg" alt="KakaoTalk_20230326_205033648_01" style="zoom:67%;" />

