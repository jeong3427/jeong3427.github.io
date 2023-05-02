---
layout: post

title:  "파이썬 알고리즘 연습 03"

categories : Python

tag : [python, 알고리즘, chatGpt, 최적, 재귀함수]

---


도시 3개를 선택하여 
인구의 합계가 500만에 가장 가까운 조합과 그 인구를 구하는 방법

.

### 1. 재귀


```python
#목표 값
goal = 5000000

#각 도시의 인구
city_pop = [3416918,2925967,2453041,1525849,1496172,
            1193894,1147037,1068641,1061440,1044579,
            942649,840047,828947,818760,702545,
            654963,652845,650599,565392,542713,
            521642,506494,489202] 

#500만에 가까운 인구 수
min_total = 0

#두 지역의 인구 수를 저장하는 임시 변수
local_temp=0

#지역 1~3 인구 수의 인덱스를 저장
local_index1=0
local_index2=0
local_index3=0

def search(total, index):
    global min_total, local_temp, local_index1, local_index2, local_index3 
    #global - 전역변수. def 함수 안에 있으면 원래는 함수 실행이 끝날 때 초기화.
    #global 선언을 하면 함수 실행이 끝나도 유지.
    
    if index >= len(city_pop):
        return
    
    if total < goal:
        if abs(goal-(total+city_pop[index])) < abs(goal-min_total):
            min_total = total + city_pop[index]
            local_temp = total
            local_index1 = index
            
        search(total+city_pop[index], index+1)
        search(total, index+1)
        
    for local_index2 in range(22):
        for local_index3 in range(22):
            if local_temp - city_pop[local_index2] == city_pop[local_index3]:
                break
            
        break
    
search(0,0)
print(min_total)
print(city_pop[local_index1]) 
print(city_pop[local_index2]) 
print(city_pop[local_index3]) 
```

    5000000
    521642 
    3416918
    1061440





2, 3은 chatGPT를 통해 얻은 코드



### 2. 조합
```python
from itertools import combinations

city_pop = [3416918, 2925967, 2453041, 1525849, 1496172,
            1193894, 1147037, 1068641, 1061440, 1044579,
            942649, 840047, 828947, 818760, 702545,
            654963, 652845, 650599, 565392, 542713,
            521642, 506494, 489202]

target_pop = 5000000
closest_pop = float('inf') #무한대(인피니트) 선언
closest_combo = None

for combo in combinations(city_pop, 3):
    pop_sum = sum(combo)
    if abs(pop_sum - target_pop) < abs(closest_pop - target_pop):
        closest_pop = pop_sum
        closest_combo = combo

print(closest_combo)
```

    (3416918, 1061440, 521642)




​    

### 3. 재귀2


```python
city_pop = [3416918, 2925967, 2453041, 1525849, 1496172,
            1193894, 1147037, 1068641, 1061440, 1044579,
            942649, 840047, 828947, 818760, 702545,
            654963, 652845, 650599, 565392, 542713,
            521642, 506494, 489202]

target_pop = 5000000
closest_pop = float('inf')
closest_combo = None

def find_closest_pop_combos(populations, target, n, combo=[]):
    """
    A recursive function that finds all combinations of n elements from a list
    of populations and returns the combination with the population sum closest to
    the target value.
    """
    global closest_pop, closest_combo
    
    if n == 0:
        # base case: we have reached the desired combination size
        pop_sum = sum(combo)
        if abs(pop_sum - target) < abs(closest_pop - target):
            closest_pop = pop_sum
            closest_combo = tuple(combo)
        return
    
    if len(populations) < n:
        # base case: there are not enough populations to form a combination of size n
        return
    
    # recursive case: try including the first population in the combination
    find_closest_pop_combos(populations[1:], target, n-1, combo + [populations[0]])
    
    # recursive case: try excluding the first population from the combination
    find_closest_pop_combos(populations[1:], target, n, combo)

# find all combinations of size 3
find_closest_pop_combos(city_pop, target_pop, 3)

print(closest_combo)
```

    (3416918, 1061440, 521642)

