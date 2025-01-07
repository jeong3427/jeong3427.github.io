---
layout: post
title:  "하버사인 공식 계산하는 법"
categories : SQL
tag : [SQL, Haversine, 최단거리]
---

https://solvesql.com/problems/find-unnecessary-station-1/

![image-20250107171637507](../../../../images/2025-01-07-haversine/image-20250107171637507.png)

![image-20250107171727699](../../../../images/2025-01-07-haversine/image-20250107171727699.png)

```sql
SELECT 
    A.station_id,
    A.name
FROM 
    station AS A
JOIN 
    station AS B ON A.station_id != B.station_id
WHERE 
    2 * 6356 * ASIN(
        SQRT(
            POWER(SIN(RADIANS((B.lat - A.lat) / 2)), 2) +
            COS(RADIANS(A.lat)) * COS(RADIANS(B.lat)) *
            POWER(SIN(RADIANS((B.lng - A.lng) / 2)), 2)
        )
    ) <= 0.3  -- 300m를 km 단위로 환산
    AND B.updated_at > A.updated_at -- A보다 나중에 생기거나 업그레이드된 정류소
GROUP BY 
    A.station_id, A.name
HAVING 
    COUNT(B.station_id) >= 5;
```



