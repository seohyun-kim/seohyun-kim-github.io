---
layout: single
title:  "MySQL Query 모음"
author: seohyun-kim
date: 2021-08-19 18:00
comments: true
---

## 1. 분석 테이블 생성 쿼리

```sql
# 일 별 테이블 생성
create table day_stats(
	rid INT PRIMARY KEY AUTO_INCREMENT,
	date TIMESTAMP ,
    avg_person float,
    std_person float,
	avg_vehicle float,
    std_vehicle float
);

```


<details>
    <summary> Table 생성 코드 더보기 </summary>  
  
 <div markdown="1">  
   
```sql
   
   
```
</div>  
  
</details>   
