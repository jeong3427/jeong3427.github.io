---
layout: post

title:  "파이썬 알고리즘 연습 04"

categories : Python

tag : [python, 알고리즘, 최적, 재귀함수, 데이크스트라, 벨만-포드]

---

```python
#경로의 수
# M은 가로 칸 수, N은 세로 칸 수
# m+n C m (m+n 콤비네이션 m)으로 구할 수 있다


### 방법1
M, N = 6, 5

route = [[0 for i in range(N+1)] for j in range(M+1)]

print(route)

#가로 방향의 첫 1행을 설정
for i in range(M+1):
    route[i][0] =1
    
print(route)


for i in range(1,N+1):
    #세로 방향의 첫 1열을 설정
    route[0][i]=1
    print(route)
    
    for j in range(1, M+1):
        # 왼쪽과 아래쪽의 교차점에 적혀 있는 숫자를 더함
        route[j][i] = route[j-1][i] + route[j][i-1]
        
print(route)
print(route[M][N])


### 방법2
import functools

M, N = 6,5

# 파이썬에서는 아래 1행만 추가하면 재귀 처리를 메모이제이션(memoization)할 수 있음
@functools.lru_cache(maxsize=None)

def search(m,n):
    if (m==0) or (n==0):
        return 1
    
    return search(m-1,n) + search(m, n-1)

print(search(M, N))



#1.벨만-포드 알고리즘
#변의 가중치

def bellman_ford(edges, num_v):
    dist =[float('inf') for i in range(num_v)]  #초기 값을 무한대로 설정
    dist[0] = 0 
    
    changed = True
    
    while changed:
        changed = False
        for edge in edges:
            if dist[edge[1]] > dist[edge[0]] + edge[2]:
                #정점까지의 비용을 갱신할 수 있으면 갱신
                dist[edge[1]] = dist[edge[0]] + edge[2]
                changed=True
                
    return dist

#변의 리스트 (출발점, 끝점, 비용의 리스트)
edges = [
    [0,1,4],[0,2,3],[1,2,1],[1,3,1],
    [1,4,5],[2,5,2],[4,6,2],[5,4,1],
    [5,6,4]
]

print(bellman_ford(edges,7))


#### 데이크스트라 알고리즘

def dijkstra(edges, num_v):
    dist = [float('inf')] * num_v
    dist[0] = 0
    
    q = [i for i in range(num_v)]
    
    while len(q)>0:
        #가장 비용이 적은 정점을 찾기
        r = q[0]
        
        for i in q:
            if dist[i] < dist[r]:
                r=i #비용이 적은 정점이 발견되면 갱신
                
                
        #가장 비용이 적은 정점을 꺼내기
        u = q.pop(q.index(r))
        
        for i in edges[u]: #꺼낸 정점의 변을 반복
            if dist[i[0]] > dist[u] + i[1]:
                #정점까지의 비용을 갱신할 수 있다면 갱신하기
                dist[i[0]] = dist[u] + i[1]
                
    return dist

#변의 리스트(끝점과 비용의 리스트)

edges=[
    [[1,4],[2,3]],
    [[2,1], [3,1], [4,5]],
    [[5,2]],
    [[4,3]],
    [[6,2]],
    [[4,1], [6,4]],
    []
]

print(dijkstra(edges,7))


import heapq

def dijkstra2(edges, num_v):
    dist = [float('inf')] * num_v
    dist[0] = 0
    
    q=[]
    heapq.heappush(q, [0,0])
    
    while len(q)>0:
        #힙에서 요소 꺼내기
        _, u = heapq.heappop(q)
        
        for i in edges[u]:
            if dist[i[0]] > dist[u] + i[1]:
                #정점까지의 비용을 갱신할 수 있다면 갱신해 힙에 등록
                dist[i[0]]= dist[u] + i[1]
                heapq.heappush(q, [dist[u] + i[1], i[0]])
                
    return dist

edges=[
    [[1,4],[2,3]],
    [[2,1], [3,1], [4,5]],
    [[5,2]],
    [[4,3]],
    [[6,2]],
    [[4,1], [6,4]],
    []
]

print(dijkstra2(edges,7))
```