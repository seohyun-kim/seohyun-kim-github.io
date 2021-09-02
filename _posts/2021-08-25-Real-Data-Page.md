---
layout: single
title:  "실제 데이터 페이지 제작 "
author: seohyun-kim
date: 2021-08-25 18:00
comments: true
---

## 메인 페이지 UI  

<br>  

![image](https://user-images.githubusercontent.com/61939286/131847732-17de40ff-d7c0-42df-82e2-af357616efe5.png)  

![image](https://user-images.githubusercontent.com/61939286/131847586-00fda4fc-df0a-4812-8c9b-5d3bd278f61f.png)


#### ✔ Done  
```
1. 계획했던 레이아웃과 유사하게 배치 
   ( 속도, 건물 출입 등등 아직 판별 불가한 데이터는 그냥 위치만 잡음 )
    
2. 일간/ 기간 버튼 탭 설정 ( 페이지 변화, 탭 색상 변화 )

3. 해당 날짜의 데이터 텍스트 출력, 그래프, 파이차트 출력
   ( 현재는 전체 데이터로 불러오게 됨. 추후에 1시간 단위로 계산된 테이블로부터 계산되도록 수정 )
   
4. 접속 시 디폴트로 오늘 날짜 데이터 출력

5. 실시간 데이터 입력 시 마다 자동 새로고침 되도록

6. 
```


<br>  
<br>  

## 로그인 정보 암호화  

#### 사용자 정보 테이블
![image](https://user-images.githubusercontent.com/61939286/130837541-06c571ca-c452-41e6-b9fc-aa4ab3d7971c.png)

node.js 에서 입력받은 패스워드를 md5 암호화를 한 뒤, 그 값이 db정보와 같은 지 비교
