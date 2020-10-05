 

# Hive

- hive는 hiveQL이라는 SQL문과 매우 유사한 언어를 제공
- 대부분의 기능은 SQL과 유사하지만 다음과 같은 차이점이 있음
  - hive에서 사용하는 데이터가 HDFS에 저장되는데, HDFS가 한번 저장 한 파일은 수정할 수 없기 때문에 UPDATE와 DELETE를 사용할 수 없다.
  - 같은 이유로 INSERT도 비어 있는 데이블에 입력하거나, 이미 입력된 데이터를 overwrite하는 경우만 가능하다. 그래서 hive는 INSERT OVERWRITE라는 키워드를 사용한다.
  - SQL은 어떠한 절에서도 서브쿼리를 사용할 수 있지만 hiveQL은 FROM절에서만 서브쿼리를 사용할 수 있다.
  - SQL의 뷰는 업데이트 할 수 있고, 구페과된 뷰 또는 인라인 뷰를 지원한다. 하지만 hiveQL의 뷰는 읽기 전용이며 인라인 뷰는 지원하지 않는다.
  - SELECT 문을 사용할 때 HAVING 절을 사용할 수 없다.
  - 저장 프로시저를 지원하지 않는다. 대신 맵리듀스 스크립트를 실행 할 수 있다.



### Airline Delay Table을 만들어 보자. 

1. csv의 파일의 헤더를 다 가져온다. 근데 ""표시를 vi에서 다 지워주면 된다.

   1. :g/"/s///g
      1. go문을 사용하면 편하다.

2. CREATE TABLE을 하자

   1. create table airline_delay(YEAR INT, MONTH INT, DAY_OF_MONTH INT, DAY_OF_WEEK INT, FL_DATE STRING, UNIQUE_CARRIER STRING, TAIL_NUM STRING, FL_NUM INT, ORIGIN_AIRPORT_ID INT, ORIGIN STRING, ORIGIN_STATE_ABR STRING, DEST_AIRPORT_ID INT, DEST STRING, DEST_STATE_ABR STRING, CRS_DEP_TIME INT, DEP_TIME INT, DEP_DELAY INT, DEP_DELAY_NEW INT, DEP_DEL15 INT, DEP_DELAY_GROUP INT, TAXI_OUT INT, WHEELS_OFF STRING, WHEELS_ON STRING, TAXI_IN INT, CRS_ARR_TIME INT, ARR_TIME INT, ARR_DELAY INT, ARR_DELAY_NEW INT, ARR_DEL15 INT, ARR_DELAY_GROUP INT, CANCELLED INT, CANCELLATION_CODE STRING, DIVERTED INT, CRS_ELAPSED_TIME INT, ACTUAL_ELAPSED_TIME INT, AIR_TIME INT, FLIGHTS INT, DISTANCE INT, DISTANCE_GROUP INT, CARRIER_DELAY STRING, WEATHER_DELAY STRING, NAS_DELAY STRING, SECURITY_DELAY STRING, LATE_AIRCRAFT_DELAY STRING)

        -> PARTITIONED BY (delayYear INT)

        -> ROW FORMAT DELIMITED

        -> FIELDS TERMINATED BY ','

        -> LINES TERMINATED BY '\n'

        -> STORED AS TEXTFILE;

3. 메타데이터 안에 이 값을 추가 하는것.

4. HDFS에 hive-data라는 폴더를 만든다.

5. 거기에 골라낸 csv파일을 넣는다.

6. hive>  load data inpath '/user/companion_tazo/hive-data/2003.csv\' overwrite into table airline_delay  partition (delayYear='2003');

7. 2002년도 동일시.

8. bin/hdfs dfs -ls /user/hive/warehouse/airline_delay/delayyear=2002 으로 확인해보면 들어가있다.

9. 복사가 아니고 이동이다. 



## 맵 리듀스 실습

- 그냥 airline_delay의 값을 뽑아보면 (2002, 2003년의 갯수)

~~~ sql
select count(*) from airline_delay;
5829525
~~~

- 헤드 값을 선택하고 2002년도에서 뽑아본다. 20개 제한.

~~~ sql
select YEAR,MONTH,DAY_OF_MONTH,DAY_OF_WEEK,FL_DATE,UNIQUE_CARRIER,TAIL_NUM,FL_NUM,ORIGIN_AIRPORT_ID,ORIGIN,ORIGIN_STATE_ABR,DEST_AIRPORT_ID,DEST,DEST_STATE_ABR,CRS_DEP_TIME,DEP_TIME,DEP_DELAY,DEP_DELAY_NEW,DEP_DEL15,DEP_DELAY_GROUP,TAXI_OUT,WHEELS_OFF,WHEELS_ON,TAXI_IN,CRS_ARR_TIME,ARR_TIME,ARR_DELAY,ARR_DELAY_NEW,ARR_DEL15,ARR_DELAY_GROUP
   \> from airline_delay
    \> where delayYear='2002'
		\> limit 20;
~~~

- 결과

~~~sql
select count(*) from airline_delay
    > where delayYear='2002';
    
    2620287
~~~



- 마찬가지로 2003년

~~~sql
select YEAR,MONTH,DAY_OF_MONTH,DAY_OF_WEEK,FL_DATE,UNIQUE_CARRIER,TAIL_NUM,FL_NUM,ORIGIN_AIRPORT_ID,ORIGIN,ORIGIN_STATE_ABR,DEST_AIRPORT_ID,DEST,DEST_STATE_ABR,CRS_DEP_TIME,DEP_TIME,DEP_DELAY,DEP_DELAY_NEW,DEP_DEL15,DEP_DELAY_GROUP,TAXI_OUT,WHEELS_OFF,WHEELS_ON,TAXI_IN,CRS_ARR_TIME,ARR_TIME,ARR_DELAY,ARR_DELAY_NEW,ARR_DEL15,ARR_DELAY_GROUP
   \> from airline_delay
    \> where delayYear='2003'
		\> limit 20;
~~~

- 결과 

~~~sql
select count(*) from airline_delay
    > where delayYear='2002';
    
    3209238
~~~



### Group By

~~~ sql
select year, month, count(*) AS arrive_delay_count
    > from airline_delay
    > where ARR_DELAY > 0
    > group by year, month;
~~~

- 결과

~~~ sql
year	month	arrive_delay_count
2002	3	199497
2003	2	201884
2002	4	172803
2003	3	199159
2002	5	171711
2003	4	162603
2002	1	176118
2002	6	186988
2003	5	184623
2002	2	148683
2003	1	195954
2003	6	210356
~~~



### AVG

~~~sql
SELECT YEAR, MONTH, AVG(ARR_DELAY) AS AVG_ARRIVAL_DELAY, AVG(DEP_DELAY) AS AVG_DEPARTURE_DELAY
    > FROM airline_delay
    > where arr_delay > 0
    > group by year, month;
~~~

- 결과

~~~sql
year	month	avg_arrival_delay	avg_departure_delay
2002	3	23.488007338456217	17.977287879015723
2003	2	26.894513681123815	19.48604644251154
2002	4	22.689901216992762	16.36929335717551
2003	3	24.32151697889626	17.239030121661585
2002	5	23.17580702459365	16.486177356139095
2003	4	21.59345768528256	15.358732618709372
2002	1	22.570804801326382	16.043652551130492
2002	6	28.277600701649305	22.033558303206622
2003	5	23.074779415349116	15.820504487523223
2002	2	19.484648547581095	13.986736883167545
2003	1	23.4672831378793	16.373138593751595
2003	6	23.718838540379167	17.205423187358573
~~~





### 어느 항공사가 지연이 많은지

1. 먼저 다른 테이블 생성 

~~~ sql
create table carrier_code(Code STRING, Description STRING)
    > ROW FORMAT DELIMITED
    > FIELDS TERMINATED BY ','
    > LINES TERMINATED BY '\n'
    > STORED AS TEXTFILE;
~~~

2. 그 안에 파일을 load을 넣어준다. 

~~~sql
load data local inpath '/mnt/Share/download/carriers.csv' into table carrier_code;
~~~

3. 조회해본다.

~~~ sql
select * from carrier_code limit 20;

"02Q"	"Titan Airways"
"04Q"	"Tradewind Aviation"
"05Q"	"Comlux Aviation
"06Q"	"Master Top Linhas Aereas Ltd."
"07Q"	"Flair Airlines Ltd."
"09Q"	"Swift Air
"0BQ"	"DCA"
"0CQ"	"ACM AIR CHARTER GmbH"
"0FQ"	"Maine Aviation Aircraft Charter
"0GQ"	"Inter Island Airways
"0HQ"	"Polar Airlines de Mexico d/b/a Nova Air"
"0J"	"JetClub AG"
"0JQ"	"Vision Airlines"
"0KQ"	"Mokulele Flight Services
"0LQ"	"Metropix UK
"0MQ"	"Multi-Aero
"0Q"	"Flying Service N.V."
"16"	"PSA Airlines Inc."
"17"	"Piedmont Airlines"
"1I"	"Sky Trek Int'l Airlines"
~~~

4. 항공사별 지연연산

~~~ sql
select a.year, a.UNIQUE_CARRIER, b.Description, count(*)
    > from airline_delay a join carrier_code b
    > on a.UNIQUE_CARRIER = b.code
    > where a.arr_delay > 0
    > group by a.year, a.UNIQUE_CARRIER, b.Description;
    > order by a.year
~~~

- 결과

~~~sql
a.year	a.unique_carrier	b.description	_c3
2002	"AA"	"American Airlines Inc."	147449
2002	"CO"	"Continental Air Lines Inc."	60567
2002	"MQ"	"American Eagle Airlines Inc."	81575
2002	"US"	"US Airways Inc. (Merged with America West 9/05. Reporting for both starting 10/07.)"	107690
2002	"WN"	"Southwest Airlines Co."	182125
2003	"NW"	"Northwest Airlines Inc."	90553
2003	"TZ"	"ATA Airlines d/b/a ATA"	13719
2003	"UA"	"United Air Lines Inc."	86713
2002	"HP"	"America West Airlines Inc. (Merged with US Airways 9/05. Stopped reporting 10/07.)"	34187
2003	"AA"	"American Airlines Inc."	122712
2003	"CO"	"Continental Air Lines Inc."	57763
2003	"MQ"	"American Eagle Airlines Inc."	70576
2003	"OO"	"Skywest Airlines Inc."	57607
2003	"US"	"US Airways Inc. (Merged with America West 9/05. Reporting for both starting 10/07.)"	84267
2003	"WN"	"Southwest Airlines Co."	152983
2002	"AS"	"Alaska Airlines Inc."	35712
2002	"DL"	"Delta Air Lines Inc."	190548
2003	"B6"	"JetBlue Airways"	12932
2003	"HP"	"America West Airlines Inc. (Merged with US Airways 9/05. Stopped reporting 10/07.)"	39163
2003	"XE"	"Expressjet Airlines Inc."	60837
2003	"AS"	"Alaska Airlines Inc."	26371
2003	"DL"	"Delta Air Lines Inc."	128272
2003	"EV"	"Atlantic Southeast Airlines"	60927
2002	"NW"	"Northwest Airlines Inc."	105247
2002	"UA"	"United Air Lines Inc."	110700
2003	"DH"	"Independence Air"	58205
2003	"FL"	"AirTran Airways Corporation"	30979
~~~



========================================================================

## hive에서 sqoop을 이용해 mysql로 Export하기

========================================================================

## hive에서 HDFS로 테이블 Export하기

1. hive 내부 에서.

~~~mysql
hive> INSERT OVERWRITE DIRECTORY '/user/rhange/output'
    > ROW FORMAT DELIMITED
    > FIELDS TERMINATED BY ','
    > LINES TERMINATED BY '\n'
    > SELECT * FROM GUN_TEST LIMIT 100;
~~~

2. 만약 하둡 바깥으로 (리눅스) 빼려면

~~~mysql
[Host~/hive]$ bin/hive -e 'select * from gun_test limit 20' > testdata.txt
~~~



