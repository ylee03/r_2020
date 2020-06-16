# 자료철 불러오기(제3강 준비 파일)

## Orthodont 자료

Potthoff와 Roy가 어린이 피험자 27명에서 연령에 따라 추적하여 측정한 치아의 성장(pterygomaxilary fissure의 길이) 자료이다. 업계에서 저명한 종적자료의 하나로 꼽힌다.

Orthodont 자료는 R의 기본 설치에 포함된 nlme 패키지에 들어 있지만 nlme를 로딩 하는 부담 없이도 다음처럼 불러 들일 수 있다.


```r
> data(Orthodont, package = "nlme")
> str(Orthodont)
```

```
## Classes 'nfnGroupedData', 'nfGroupedData', 'groupedData' and 'data.frame':	108 obs. of  4 variables:
##  $ distance: num  26 25 29 31 21.5 22.5 23 26.5 23 22.5 ...
##  $ age     : num  8 10 12 14 8 10 12 14 8 10 ...
##  $ Subject : Ord.factor w/ 27 levels "M16"<"M05"<"M02"<..: 15 15 15 15 3 3 3 3 7 7 ...
##  $ Sex     : Factor w/ 2 levels "Male","Female": 1 1 1 1 1 1 1 1 1 1 ...
##  - attr(*, "outer")=Class 'formula'  language ~Sex
##   .. ..- attr(*, ".Environment")=<environment: R_GlobalEnv> 
##  - attr(*, "formula")=Class 'formula'  language distance ~ age | Subject
##   .. ..- attr(*, ".Environment")=<environment: R_GlobalEnv> 
##  - attr(*, "labels")=List of 2
##   ..$ x: chr "Age"
##   ..$ y: chr "Distance from pituitary to pterygomaxillary fissure"
##  - attr(*, "units")=List of 2
##   ..$ x: chr "(yr)"
##   ..$ y: chr "(mm)"
##  - attr(*, "FUN")=function (x)  
##   ..- attr(*, "source")= chr "function (x) max(x, na.rm = TRUE)"
##  - attr(*, "order.groups")= logi TRUE
```


## 학교 이름 자료

표를 결합하는 연습에 쓰이는 비교적 간단한 세 개의 테이블이므로 손으로 입력해 본다.


```r
> schools_left <- data.frame(
+ 	id = c(1, 2, 5, 6),
+ 	left_school = c(
+ 			"Oak Street School",
+ 			"Roosevelt High School",
+ 			"Washington Middle School",
+ 			"Jefferson High School"),
+ 	stringsAsFactors = FALSE
+ 	)
> schools_right <- data.frame(
+ 	id = c(1, 2, 3, 4, 6),
+ 	right_school = c(
+ 			 "Oak Street School",
+ 			 "Roosevelt High School",
+ 		   "Morrison Elementary",
+ 		   "Chase Magnet Academy",
+ 		   "Jefferson High School"),
+ 	stringsAsFactors = FALSE
+ 	)
> schools_dupl <- rbind(
+ 	schools_right,
+ 	data.frame(
+ 			id = 1,
+ 			right_school = "Every Valley School")
+ 	)
```

학교 자료는 다음처럼 보이면 완성된 것이다.


```r
> schools_left
```

```
##   id              left_school
## 1  1        Oak Street School
## 2  2    Roosevelt High School
## 3  5 Washington Middle School
## 4  6    Jefferson High School
```

```r
> schools_right	
```

```
##   id          right_school
## 1  1     Oak Street School
## 2  2 Roosevelt High School
## 3  3   Morrison Elementary
## 4  4  Chase Magnet Academy
## 5  6 Jefferson High School
```

```r
> schools_dupl
```

```
##   id          right_school
## 1  1     Oak Street School
## 2  2 Roosevelt High School
## 3  3   Morrison Elementary
## 4  4  Chase Magnet Academy
## 5  6 Jefferson High School
## 6  1   Every Valley School
```

## nycflights 자료

해들리 위컴은(http://hadley.nz) R에서 관계형DB 명령을 구현하는 함수 모음집인 dplyr을 제작하면서 연습용 자료철 nycflights13 (우리는 nycflights로 부른다)을 함께 배포하였다. R 패키지 nycflights13에 들어있지만 자동으로 tibble 패키지가 함께 로딩 되므로 우리는 그 패키지 없이 깃허브의 원본 csv 파일을 사용할 것이다.

nycflights는 2013년에 미국 뉴욕시에서 출발한 모든 항공 관련 시간대 기록 장부를 다섯 개의 테이블로 정리한 자료철이다.  자료를 통틀어서 결측값에는 NA가 들어 있고 숫자들의 측정단위는 미국내 표준으로서 피트, 마일, 화씨 온도, 밀리바 등으로 읽으면 된다.




csv 파일을 불러오면서 각각에 같은 이름의 자료철 이름을 부여했다면 테이블로(여기서는 데이터프레임) 구성되어 있다: airlines, airports, flights, planes, weather. 

이름이 내용을 강변하므로 긴 설명이 필요치 않겠지만 알파벳 순서대로 정리해 보았다. 부족한 내용은 이 자료철에 대응하는  R 공식 설명문(https://cran.r-project.org/web/packages/nycflights13/nycflights13.pdf)을 참고해서 보충하면 좋겠다.

### airlines

미국 국내 항공사의 이름과 코드를 담고 있다. 깃허브의 원본 csv 파일을 읽어 들일 때 맨 윗 줄은 헤더 즉 변수명임을 지시해야  하고 stringsAsFactors = FALSE로 놓아 문자열들이 요인 변수로 읽히는 것을 예방한다.


```r
> airlines <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/airlines.csv", header = TRUE, stringsAsFactors = FALSE) #
> str(airlines)
```

```
## 'data.frame':	16 obs. of  2 variables:
##  $ carrier: chr  "9E" "AA" "AS" "B6" ...
##  $ name   : chr  "Endeavor Air Inc." "American Airlines Inc." "Alaska Airlines Inc." "JetBlue Airways" ...
```

### airports

미국 내 1458개 공항 목록이다. 미연방항공국이 관리하는 코드명(faa), 영문 이름, 지리 좌표, 고도, 시간대 등의 정보를 함께 가지고 있다. dst 필드는 일광절약시간(daylight saving time zone) 여부가 문자열 "A", "N", "U"로 기재되어 있다. "A"는 표준일광절약시간대, "N"은 아님, "U"는 모름이다.


```r
> airports <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/airports.csv", header = TRUE, stringsAsFactors = FALSE) #
> str(airports)
```

```
## 'data.frame':	1458 obs. of  8 variables:
##  $ faa  : chr  "04G" "06A" "06C" "06N" ...
##  $ name : chr  "Lansdowne Airport" "Moton Field Municipal Airport" "Schaumburg Regional" "Randall Airport" ...
##  $ lat  : num  41.1 32.5 42 41.4 31.1 ...
##  $ lon  : num  -80.6 -85.7 -88.1 -74.4 -81.4 ...
##  $ alt  : int  1044 264 801 523 11 1593 730 492 1000 108 ...
##  $ tz   : int  -5 -6 -6 -5 -5 -5 -5 -5 -5 -8 ...
##  $ dst  : chr  "A" "A" "A" "A" ...
##  $ tzone: chr  "America/New_York" "America/Chicago" "America/Chicago" "America/New_York" ...
```

### flights

nycflights의 중심 테이블이다. 약 33만 7천 개의 항공기 이착륙 정보를 담고 있다. 이륙 날짜와 시간, 도착지 코드, 항공 시간, 거리 등이 들어 있다. 


```r
> flights <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/flights.csv", header = TRUE, stringsAsFactors = FALSE) #
> str(flights)
```

```
## 'data.frame':	336776 obs. of  19 variables:
##  $ year          : int  2013 2013 2013 2013 2013 2013 2013 2013 2013 2013 ...
##  $ month         : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ day           : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ dep_time      : int  517 533 542 544 554 554 555 557 557 558 ...
##  $ sched_dep_time: int  515 529 540 545 600 558 600 600 600 600 ...
##  $ dep_delay     : int  2 4 2 -1 -6 -4 -5 -3 -3 -2 ...
##  $ arr_time      : int  830 850 923 1004 812 740 913 709 838 753 ...
##  $ sched_arr_time: int  819 830 850 1022 837 728 854 723 846 745 ...
##  $ arr_delay     : int  11 20 33 -18 -25 12 19 -14 -8 8 ...
##  $ carrier       : chr  "UA" "UA" "AA" "B6" ...
##  $ flight        : int  1545 1714 1141 725 461 1696 507 5708 79 301 ...
##  $ tailnum       : chr  "N14228" "N24211" "N619AA" "N804JB" ...
##  $ origin        : chr  "EWR" "LGA" "JFK" "JFK" ...
##  $ dest          : chr  "IAH" "IAH" "MIA" "BQN" ...
##  $ air_time      : int  227 227 160 183 116 150 158 53 140 138 ...
##  $ distance      : int  1400 1416 1089 1576 762 719 1065 229 944 733 ...
##  $ hour          : int  5 5 5 5 6 5 6 6 6 6 ...
##  $ minute        : int  15 29 40 45 0 58 0 0 0 0 ...
##  $ time_hour     : chr  "2013-01-01 05:00:00" "2013-01-01 05:00:00" "2013-01-01 05:00:00" "2013-01-01 05:00:00" ...
```
2051번째 행은 JFK 공항을(origin) 2013년 1월(month) 3일(day) 10시 00분에 출발하기로 예정된(sched_dep_time) DL사(carrier) 항공기 N703TW가(tailnum) 실제로는 9시 36분(dep_time)에 이륙해서 도착지인 SFO 공항(dest)에 13시 12분(arr_time)에 도착했음을 기록하고 있다. 출발 지연 없이 3분 일찍 출발하였으면(dep_delay)이고 비행시간은 3시간 47분(air_time), 비행거리는 2586마일(distance)이다. 분석자들의 편의를 제공하고자 변수 time_hour에는 예정된 출발 날짜와 시간이 POSIXct 형태로 들어 있으며 시간과 분으로 나누어서 hour 열과 minute 열에도 쪼개서 담고 있다. 


```r
> flights[2051, ]
```

```
##      year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
## 2051 2013     1   3      957           1000        -3     1312           1341
##      arr_delay carrier flight tailnum origin dest air_time distance hour minute
## 2051       -29      DL   1765  N703TW    JFK  SFO      347     2586   10      0
##                time_hour
## 2051 2013-01-03 10:00:00
```

이 flights 테이블이 전체 자료철의 핵심 정보인 항공기의 비행정보를 담고 있다. 이착륙 공항의 코드는 faa 코드를 통해서 airport 테이블과 연관되어 있고 항공기의 개별 정보는 항공기 꼬리번호인 tailnum로 planes 테이블과 연관되었다. 날찌 정보는 날짜와 시간, 출발지 정보와 연관하여 weather 테이블로, 항공사는 carrier 코드를 통해 airlines 테이블과 연관되어 있다.

### planes

 연방항공국에 등록된 꼬리번호로(tailnum) 식별할 수 있는 3322대의 개별 항공기의 기본 정보를 담고 있다. 
 

```r
> planes <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/planes.csv", header = TRUE, stringsAsFactors = FALSE) #
>  str(planes)
```

```
## 'data.frame':	3322 obs. of  9 variables:
##  $ tailnum     : chr  "N10156" "N102UW" "N103US" "N104UW" ...
##  $ year        : int  2004 1998 1999 1999 2002 1999 1999 1999 1999 1999 ...
##  $ type        : chr  "Fixed wing multi engine" "Fixed wing multi engine" "Fixed wing multi engine" "Fixed wing multi engine" ...
##  $ manufacturer: chr  "EMBRAER" "AIRBUS INDUSTRIE" "AIRBUS INDUSTRIE" "AIRBUS INDUSTRIE" ...
##  $ model       : chr  "EMB-145XR" "A320-214" "A320-214" "A320-214" ...
##  $ engines     : int  2 2 2 2 2 2 2 2 2 2 ...
##  $ seats       : int  55 182 182 182 55 182 182 182 182 182 ...
##  $ speed       : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ engine      : chr  "Turbo-fan" "Turbo-fan" "Turbo-fan" "Turbo-fan" ...
```


### weather

뉴욕시의 세 개 공항의 시간대별 기상 기록이다. 기상정보는 소재지(origin), 날짜/시간, 기온(temp), 수분응결점(dewp), 상대습도(humid), 바람 방향(wind_dir), 속도(wind_speed), 돌풍속도(wind_gust), 강수량(precip), 기압(pressure), 시야(visib)가 들어 있다. 
 

```r
> weather <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/weather.csv", header = TRUE, stringsAsFactors = FALSE) #
>  str(weather)
```

```
## 'data.frame':	26115 obs. of  15 variables:
##  $ origin    : chr  "EWR" "EWR" "EWR" "EWR" ...
##  $ year      : int  2013 2013 2013 2013 2013 2013 2013 2013 2013 2013 ...
##  $ month     : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ day       : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ hour      : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ temp      : num  39 39 39 39.9 39 ...
##  $ dewp      : num  26.1 27 28 28 28 ...
##  $ humid     : num  59.4 61.6 64.4 62.2 64.4 ...
##  $ wind_dir  : int  270 250 240 250 260 240 240 250 260 260 ...
##  $ wind_speed: num  10.36 8.06 11.51 12.66 12.66 ...
##  $ wind_gust : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ precip    : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ pressure  : num  1012 1012 1012 1012 1012 ...
##  $ visib     : num  10 10 10 10 10 10 10 10 10 10 ...
##  $ time_hour : chr  "2013-01-01 01:00:00" "2013-01-01 02:00:00" "2013-01-01 03:00:00" "2013-01-01 04:00:00" ...
```

시야를 SI 단위인 km로 환산하려면 다음처럼 하면 된다.


```r
> weather$visib_km <- weather$visib * 1.6
> head(cbind(weather$visib, weather$visib_km))
```

```
##      [,1] [,2]
## [1,]   10   16
## [2,]   10   16
## [3,]   10   16
## [4,]   10   16
## [5,]   10   16
## [6,]   10   16
```
