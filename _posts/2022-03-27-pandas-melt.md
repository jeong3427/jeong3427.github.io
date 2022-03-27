---
layout: post

title:  "pandas melt 함수 - 가로 세로 데이터 바꾸기"

categories : Python

tag : [python, pandas, melt]
---





한 번 팩을 열면 8개의 선수가 나오는 뽑기에서

선수당 1~5성 등급이 매겨져 있을 때, 

각 팩별로 설정한 확률대로 나오는지 검증하는 과정

.

.

![image-20220327162528080](../../../../img/2022-03-27-pandas-melt/image-20220327162528080.png)

.

DB에서 43만 줄 정도 꺼낸 결과

player_id1 ~ player_id8까지가 하나의 player_pack_id에 대한 결과 값이다

.

.

![image-20220327162714405](../../../../img/2022-03-27-pandas-melt/image-20220327162714405.png)

 player_id별로 cost는 다른 파일에 저장되어 있어서 df2로 설정

.

.

df2의 player_id에 따라 cost를 vlookup 하기 좋게 하기 위해

df1의 player_id1~ player_id8을 가로에서 세로로 변경

melt 관련 참고글

- https://rfriend.tistory.com/278
- https://cosmosproject.tistory.com/309

```python
df3 = pd.melt(df1,id_vars=['player_pack_id'], 
              value_vars=['player_id1','player_id2','player_id3','player_id4','player_id5','player_id6','player_id7','player_id8'])
```

![image-20220327163056350](../../../../img/2022-03-27-pandas-melt/image-20220327163056350.png)



.

.

그 후 df2의 cost 정보와 연결하고 group by로 묶으면 각 팩별로 어떤 cost가 나왔는지 집계 가능

```python
#보기 쉽게 헤더명 변경
df3.columns=['player_pack_id','player_slot','player_id']

#vlookup과 같은 기능
df3.insert(3,'cost',df3['player_id'].map(df2.set_index('player_id')['cost']))

#팩별, 코스트별로 각각 몇 번 나왔는지 집계
df4=df3.groupby(['player_pack_id','cost']).agg({'player_id':'count'})
```

![image-20220327163409528](../../../../img/2022-03-27-pandas-melt/image-20220327163409528.png)

![image-20220327163423253](../../../../img/2022-03-27-pandas-melt/image-20220327163423253.png)

.

.

마무리는 엑셀에서

