---
layout: post

title:  "하노이의 탑"

categories : Python

tag : [python, 재귀함수, 알고리즘, 하노이의탑]
---

.

여러 가지 버전

```python
#횟수 세는 함수
def counter():
    i=0
    def count():
        nonlocal i
        i+=1
        return i
    return count

c = counter()


#버전1
def tower_of_hanoi(n, source, awxiliary, target):
    c()
    if n==1:
        print("Move disk 1 from source", source, "to target", target)
        return
    tower_of_hanoi(n-1, source, target, awxiliary)
    print("Move disk", n, "from source", source, "to target", target)
    tower_of_hanoi(n-1, awxiliary, source, target)

print("버전1") 
tower_of_hanoi(4,"A","B","C")
print(c()-1)
print("")
print("")

#버전2
c = counter()

def tower_of_hanoi_2(n, fro, tmp, to):
    c()
    if n==1:
        print("원판 1 : %s-->%s" %(fro,to))
        
    else:
        tower_of_hanoi_2(n-1, fro, to, tmp)
        print("원판 %d : %s-->%s" %(n,fro,to))
        tower_of_hanoi_2(n-1, tmp, fro, to)

print("버전2")         
tower_of_hanoi_2(4,"A","B","C")
print(c()-1)
print("")
print("")  

#버전3
def tower_of_hanoi_3(n, start,to,mid,answer):
    if n==1:
        return answer.append([start, to])
    tower_of_hanoi_3(n-1, start, mid, to, answer)
    answer.append([start, to])
    tower_of_hanoi_3(n-1, mid, to, start, answer)
    
def solution(n):
    answer = []
    tower_of_hanoi_3(n,1,3,2,answer)
    return answer, len(answer)

print("버전3")  
print(solution(4))
print("")
print("")


#버전4
print("버전4") 
MOVE_MESSAGE = "{}번 원반을 {}에서 {}로 이동"

def move(N, start, destination):
    print(MOVE_MESSAGE.format(N, start, destination))
    
def tower_of_hanoi_4(n, start, destination, via):
    """
    하노이의 탑 규칙에 따라 원반을 옮기고
    옮길 때마다 원반의 시작 위치와 이동한 위치를 출력합니다.
    마지막으로 총 이동 횟수를 반환합니다.

    :params
        n: 총 원반 개수
        start : 시작 기둥
        destination : 도착 기둥
        via : 보조 기둥:
    :return count:
    """
    
    #원반이 1개일 때 시작 기둥에서 도착 기둥까지 한 번에 이동
    
    if n<=1:
        move(1, start, destination)
        return 1
    
    count = 0
    
    #원반 n-1개를 시작 기둥에서 보조 기둥으로 이동
    count += tower_of_hanoi_4(n-1, start, via, destination)
    
    #가장 큰 원반을 시작 기둥에서 도착 기둥으로 이동
    count += 1
    move(n, start, destination)
    
    #원반 n-1개를 보조 기둥에서 도착 기둥으로 이동
    count += tower_of_hanoi_4(n-1, via, destination, start)
    
    return count

if __name__ == '__main__':
    n=4
    start =1
    destination =3
    via =2
    
    total_count = tower_of_hanoi_4(n, start, destination, via)
    print("총 이동 횟수 : ", total_count)
```

.

.

결과

![image-20230204235637566](../../../../images/2023-02-04-tower_of_hanoi/image-20230204235637566.png)

![image-20230204235659640](../../../../images/2023-02-04-tower_of_hanoi/image-20230204235659640.png)