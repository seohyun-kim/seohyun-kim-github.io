---
layout: single
title:  "MySQL Query 모음"
author: seohyun-kim
date: 2021-08-19 18:00
comments: true
---


## 1. 분석 테이블 생성 쿼리

```sql

# 1시간 단위 테이블 생성
create table hourly_stats(
	rid INT PRIMARY KEY AUTO_INCREMENT,
	start_datetime TIMESTAMP , #시작 시간
    avg_person float,
    std_person float,
	avg_vehicle float,
    std_vehicle float
);

# 일 별 테이블 생성
create table day_stats(
	rid INT PRIMARY KEY AUTO_INCREMENT,
	date date ,
    avg_person float,
    std_person float,
	avg_vehicle float,
    std_vehicle float
);

# 이상값 백업테이블 #
create table backup(
	  rid int NOT NULL,
      date_time TIMESTAMP ,
      n_person int ,
      n_car int,
      n_truck int,
      n_bus int,
      n_bicycle int,
      n_motorcycle int,
      n_electric_scooter int,
      msg varchar(32)
  ); 

```  

<details>
    <summary> Table 생성 코드 더보기 </summary>  
  
 <div markdown="1">  
   
```sql
   
# 주 별 테이블 생성
create table week_stats(
	rid INT PRIMARY KEY AUTO_INCREMENT,
    start_date date,
    end_date date,
    avg_person float,
    std_person float,
	avg_vehicle float,
    std_vehicle float
);

# 월 별 테이블 생성
create table month_stats(
	rid INT PRIMARY KEY AUTO_INCREMENT,
    start_date date, #월 포맷이 없어서 그냥 해당 달의 첫날로함
    avg_person float,
    std_person float,
	avg_vehicle float,
    std_vehicle float
);

# 연도별 테이블 생성
create table year_stats(
	rid INT PRIMARY KEY AUTO_INCREMENT,
    year year, #년도
    avg_person float,
    std_person float,
	avg_vehicle float,
    std_vehicle float
);
   
```
</div>  
  
</details>   

<br>  

## 분석 이벤트 생성  

![image](https://user-images.githubusercontent.com/61939286/130327422-587d8c5a-c706-4696-9d69-157c2d7634f0.png)  


![image](https://user-images.githubusercontent.com/61939286/130488173-3ea18185-dd8a-4734-9b2b-9a942f7d535a.png)  
(데이터가 들어오는 것이 없어서 NULL상태, 시간에 맞추어 잘 작동됨)

```sql  

 # 0). 매시 00분에 1시간 치 데이터 평균과 기준값을 `hourly_stats` 테이블에 저장   
CREATE EVENT `hourly_stats`
    ON SCHEDULE EVERY 1 hour
    STARTS DATE_ADD( DATE_FORMAT(now(), '%Y-%m-%d %H:00:00'), INTERVAL 1 hour) #다가오는 정시부터 시작
	ON COMPLETION NOT PRESERVE ENABLE
    COMMENT 'average and reference value of every hour'
    DO 
	INSERT INTO hourly_stats(start_datetime, avg_person, std_person, avg_vehicle, std_vehicle) 
		SELECT DATE_ADD( DATE_FORMAT(now(), '%Y-%m-%d %H:00:00'), INTERVAL -1 hour) , CONVERT(AVG(n_person), float), CONVERT(AVG(n_person)+1.5*STDDEV(n_person), float),
				CONVERT(AVG(n_car+n_truck+n_bus+n_bicycle+n_motorcycle+n_electric_scooter), float), 
                CONVERT(AVG(n_car+n_truck+n_bus+n_bicycle+n_motorcycle+n_electric_scooter)+1.5*STDDEV(n_car+n_truck+n_bus+n_bicycle+n_motorcycle+n_electric_scooter), float)
                FROM cctv_data 
                WHERE date_time BETWEEN DATE_ADD( DATE_FORMAT(now(), '%Y-%m-%d %H:00:00'), INTERVAL -1 hour) AND DATE_ADD( DATE_FORMAT(now(), '%Y-%m-%d %H:59:59'), INTERVAL -1 hour)  
```                
<br>  

![image](https://user-images.githubusercontent.com/61939286/130488313-a055bbf2-b5da-44de-8e51-7b734580732c.png)  


(데이터가 들어오는 것이 없어서 NULL 상태, 시간에 맞추어 잘 작동됨)
 ```sql               
# 1). 매일 00:00:00 에 하루치 데이터 평균과 기준값을 `day_stats` 테이블에 저장   
CREATE EVENT `day_stats`
    ON SCHEDULE EVERY 1 Day 
    STARTS CURDATE() #오늘부터 시작
	ON COMPLETION NOT PRESERVE ENABLE
    COMMENT 'average and reference value of day'
    DO 
	INSERT INTO day_stats(date, avg_person, std_person, avg_vehicle, std_vehicle) 
		SELECT  DATE_ADD(CURDATE(),INTERVAL -1 day)  , CONVERT(AVG(n_person), float), CONVERT(AVG(n_person)+1.5*STDDEV(n_person), float),
				CONVERT(AVG(n_car+n_truck+n_bus+n_bicycle+n_motorcycle+n_electric_scooter), float), 
                CONVERT(AVG(n_car+n_truck+n_bus+n_bicycle+n_motorcycle+n_electric_scooter)+1.5*STDDEV(n_car+n_truck+n_bus+n_bicycle+n_motorcycle+n_electric_scooter), float)
                FROM cctv_data  WHERE DATE(date_time) =  DATE_ADD(CURDATE(), INTERVAL -1 day) #어제 데이터



```

<details>
    <summary> Event 생성 코드 더보기 </summary>  
  
 <div markdown="1">  
   
   
   ![image](https://user-images.githubusercontent.com/61939286/130839143-866fe411-3bae-4fb0-b8e6-52f5aac8719c.png)  

```sql
  #### 2). 매주 월요일에 `day_stats` table의 일주일 치(7개의 row) 평균을 `week_stats` 테이블에 저장                 
  CREATE EVENT `week_stats`
    ON SCHEDULE
	EVERY 1 WEEK
	STARTS CURRENT_DATE + INTERVAL 7- WEEKDAY(CURRENT_DATE) DAY #가장 가까운 미래의 월요일
	ON COMPLETION NOT PRESERVE ENABLE
    COMMENT 'average and reference value of week'
    DO 
	INSERT INTO week_stats(start_date, end_date, avg_person, std_person, avg_vehicle, std_vehicle) 
		SELECT CURRENT_DATE + INTERVAL -7- WEEKDAY(CURRENT_DATE) DAY,#지난 주 월요일
			CURRENT_DATE + INTERVAL -1 - WEEKDAY(CURRENT_DATE) DAY, #지난 주 일요일
			CONVERT(AVG(avg_person), float), CONVERT(AVG(std_person), float),
			CONVERT(AVG(avg_vehicle), float), CONVERT(AVG(std_vehicle), float)
			FROM day_stats WHERE date BETWEEN DATE_ADD(DATE_FORMAT(now(), '%Y-%m-%d 00:00:00'),INTERVAL -1 WEEK ) 
      AND DATE_ADD( DATE_FORMAT(now(), '%Y-%m-%d 23:59:59'), INTERVAL -1 day) 
            #오늘(매주 월요일 0시) 기준으로 지난 일주일 간

   
# 3). 매달 1일에 `day_stats` table의 한달 치 평균을 `month_stats` 테이블에 저장              
 CREATE EVENT `month_stats`
    ON SCHEDULE
	EVERY 1 MONTH
	STARTS LAST_DAY(NOW()) + interval 1 DAY #가장 가까운 1일부터 시작
	ON COMPLETION NOT PRESERVE ENABLE
    COMMENT 'average and reference value of month'
    DO 
	INSERT INTO month_stats(start_date, avg_person, std_person, avg_vehicle, std_vehicle) 
		SELECT LAST_DAY(DATE_FORMAT(now(), '%Y-%m-%d 00:00:00') - interval 2 month) + interval 1 DAY, #지난 달의 1일
			CONVERT(AVG(avg_person), float), CONVERT(AVG(std_person), float),
			CONVERT(AVG(avg_vehicle), float), CONVERT(AVG(std_vehicle), float)
			FROM day_stats 
   WHERE date BETWEEN LAST_DAY(DATE_FORMAT(now(), '%Y-%m-%d 00:00:00') - interval 2 month) + interval 1 DAY 
   AND  LAST_DAY(DATE_FORMAT(now(), '%Y-%m-%d 23:59:59') - interval 1 month) 
   
   
# 4). 매년 1월1일에 `month_stats` table의 일년 치 평균(12개의 row)을 `year_stats` 테이블에 저장     
 CREATE EVENT `year_stats`
    ON SCHEDULE
	EVERY 1 YEAR
	STARTS LAST_DAY(DATE_ADD(NOW(), INTERVAL 12-MONTH(NOW()) MONTH))+ interval 1 DAY #가장 가까운 1월 1일부터 시작
	ON COMPLETION NOT PRESERVE ENABLE
    COMMENT 'average and reference value of year'
    DO 
	INSERT INTO year_stats(_year, avg_person, std_person, avg_vehicle, std_vehicle) 
		SELECT YEAR(CURDATE()-interval 1 DAY), # 지난 년도 (현재 1월1일)
			CONVERT(AVG(avg_person), float), CONVERT(AVG(std_person), float),
			CONVERT(AVG(avg_vehicle), float), CONVERT(AVG(std_vehicle), float)
			FROM month_stats 
            where start_date BETWEEN DATE_ADD(DATE_FORMAT(now(), '%Y-01-01 00:00:00'),INTERVAL -1 YEAR ) 
            AND DATE_ADD(DATE_FORMAT(now(), '%Y-12-31 23:59:59'),INTERVAL -1 YEAR ) #지난 일년   
            
```
   
</div>  
  
</details>   

<br>  

## 이상값 탐지 트리거  

```sql  
DELIMITER //
CREATE TRIGGER detection 
	AFTER INSERT 
    ON cctv_data
	FOR EACH ROW
BEGIN
	DECLARE people_ref_val, vehicle_ref_val FLOAT DEFAULT NULL;
    
	IF (SELECT COUNT(_id) FROM year_stats) >=1 THEN 	
		SET people_ref_val = (SELECT AVG(std_person) FROM year_stats);
		SET vehicle_ref_val = (SELECT AVG(std_vehicle) FROM year_stats);
        
	ELSE IF (SELECT COUNT(_id) FROM month_stats) >=1 THEN
		SET people_ref_val = (SELECT AVG(std_person) FROM month_stats);
		SET vehicle_ref_val = (SELECT AVG(std_vehicle) FROM month_stats);
        
	ELSE IF (SELECT COUNT(_id) FROM week_stats) >=1 THEN
		SET people_ref_val = (SELECT AVG(std_person) FROM week_stats);
		SET vehicle_ref_val = (SELECT AVG(std_vehicle) FROM week_stats);
			
	ELSE IF (SELECT COUNT(_id) FROM day_stats) >=1 THEN
		SET people_ref_val = (SELECT AVG(std_person) FROM day_stats);
		SET vehicle_ref_val = (SELECT AVG(std_vehicle) FROM day_stats);
        
	ELSE IF (SELECT COUNT(_id) FROM hourly_stats) >=1 THEN
		SET people_ref_val = (SELECT AVG(std_person) FROM day_stats);
		SET vehicle_ref_val = (SELECT AVG(std_vehicle) FROM day_stats);
					END IF;
				END IF;
			END IF;
		END IF;
	END IF;

	# 설정된 기준 값으로 이상값 탐지
    IF  NEW.n_person > (SELECT @people_ref_val) THEN
		IF  SUM(NEW.n_car +NEW.n_truck+NEW.n_bus+NEW.n_bicycle+NEW.n_motorcycle+NEW.n_electric_scooter) > (SELECT @vehicle_ref_val) THEN
			INSERT INTO backup VALUES(NEW.rid, NEW.date_time ,NEW.n_person, NEW.n_car, NEW.n_truck, NEW.n_bus, NEW.n_bicycle, NEW.n_motorcycle, NEW.n_electric_scooter, 'Outlier People & Vehicle');
		ELSE 
			INSERT INTO backup VALUES(NEW.rid, NEW.date_time ,NEW.n_person, NEW.n_car, NEW.n_truck, NEW.n_bus, NEW.n_bicycle, NEW.n_motorcycle, NEW.n_electric_scooter, 'Outlier People');
		END IF;
	ELSEIF SUM(NEW.n_car +NEW.n_truck+NEW.n_bus+NEW.n_bicycle+NEW.n_motorcycle+NEW.n_electric_scooter) > (SELECT @vehicle_ref_val) THEN
		INSERT INTO backup VALUES(NEW.rid, NEW.date_time ,NEW.n_person, NEW.n_car, NEW.n_truck, NEW.n_bus, NEW.n_bicycle, NEW.n_motorcycle, NEW.n_electric_scooter, 'Outlier Vehicle');
    END IF;
    
   
END //
DELIMITER ;

```






