# 자료철 헤쳐 보기

## 헤쳐 보기 전에

테이블 형식의 자료는 그때그때의 필요에 따라 자유롭게 분철하고 결합하는 게 자료를 가공하는 첫 출발이다. 포괄적 의미의 분철(subset)은 행-선택(row filter 또는 좁은 의미로서 subset)과 열-선택(column select)으로 구분할 수 있다. 결합(merge 또는 join)은 테이블에 없었던 정보를 찾아서 적절한 위치에 붙이는 기술이다. 테이블을 피벗을 중심으로 펼치거나(widen) 연장(lengthen)하는 것도 적절히 구사할 수 있어야 한다.

자료의 가공 기술
1) 행-선택
2) 열-선택
3) 결합
4) 펼치거나 연장

R에는 행과 열 선택을 모두 처리할 수 있는 브라켓([) 함수와 subset 함수가 있다. 열-선택만을 할 때엔 이중브라켓([[) 함수, 또는 \$ 함수를 쓰면 된다. 결합에는 merge 함수를 쓸 수 있고, 지금까지 배운 R의 기본 함수들을 동원해서 테이블을 펼치거나 연장할 수 있다. 그밖에도 기계적인 조건에 따라 분철된 자료에 특정 함수를 연산하도록 하는 apply, tapply 등의 함수가 있다.

문자열 자료에 대해서는 숫자들과는 조금 다른 접근법을 취해야 하는데 우리 수업에서는 본격적으로는 다루지 않을 것이므로 예문이 등장할 때마다 숙지하는 정도로 족하다.

## Orthodont 자료철

피험자들의 치아 크기(distance)를 남녀 간에 구분해서 평균과 표준편차를 요약해 보았다. 


```r
> tapply(Orthodont$distance, Orthodont$Sex, mean)
```

```
##     Male   Female 
## 24.96875 22.64773
```

```r
> tapply(Orthodont$distance, Orthodont$Sex, sd)
```

```
##     Male   Female 
## 2.900294 2.398109
```

동일한 논리에 따라서 연령에 따른 요약값도 쉽게 얻을 수 있다. 전체를 with 함수로 묶으면 데이터프레임 이름을 반복하지 않아도 변수를 인식할 수 있다.


```r
> with( Orthodont, tapply(distance, age, mean) )
```

```
##        8       10       12       14 
## 22.18519 23.16667 24.64815 26.09259
```

tapply 함수는 단일한 출력에만 작동하기 때문에 mean과 sd를 다음처럼 c로 묶어서는 오류가 발생할 것이다.


```r
> tapply(Orthodont$distance, Orthodont$Sex, c(mean, sd))
```

```
## Error in match.fun(FUN): 'c(mean, sd)' is not a function, character or symbol
```

하지만 똑같은 내용을 사용자가 만든 무명의 함수에 구겨 넣으면 우리가 원하는 대로 작동한다.


```r
> tapply(Orthodont$distance, Orthodont$Sex, function(x) c(mean(x), sd(x))) #
```

```
## $Male
## [1] 24.968750  2.900294
## 
## $Female
## [1] 22.647727  2.398109
```

여기까지 살펴 본 예문들을 직관적으로 살필 때 `tapply(var, condition, what)`로 쓰면 변수 var가 조건 condition에 따라 분별된 계산값 what이 나옴을 알 수 있다. var는 단일 변수야 함은 지당하고 what도 단일값이어야 하지만 여러 값을 묶으려면 사용자가 만든 함수를 쓰면 되며 조건 condition도 여러 조건을 묶어서 지정할 수 있는데 그러려면 list로 전달하여야만 한다. 즉, 성별/연령별 평균과 표준편차는 이렇게 구한다.


```r
> tapply(Orthodont$distance, list(Orthodont$Sex, Orthodont$age), mean) #
```

```
##               8       10       12       14
## Male   22.87500 23.81250 25.71875 27.46875
## Female 21.18182 22.22727 23.09091 24.09091
```

```r
> tapply(Orthodont$distance, list(Orthodont$Sex, Orthodont$age), sd) #
```

```
##               8       10       12       14
## Male   2.452889 2.136001 2.651847 2.085416
## Female 2.124532 1.902152 2.364510 2.437398
```

Orthodont 자료는 길이와 폭이 짧으므로 사람의 눈으로 자료의 모든 속성을 파악하는 데 어려움이 없다. 그럼에도 불구하고 치아의 크기가 측정되었던 피험자들의 나이를 확인하고자 하면 unique 함수를 쓸 수 있다.


```r
> unique( Orthodont$age )
```

```
## [1]  8 10 12 14
```

이미 알고 있는 table 함수를 써서 각 나이에 대응하는 레코드의 숫자도 세어 본다. 자주 언급하는 점이지만, table 함수가 만드는 모든 결과물이 matrix나 array로 귀결되는 것은 아니다. 이 경우에는 길이가 4인 벡터가 만들어진다.


```r
> table( Orthodont$age )
```

```
## 
##  8 10 12 14 
## 27 27 27 27
```

```r
> is.matrix( table( Orthodont$age ) )
```

```
## [1] FALSE
```

```r
> length( table( Orthodont$age ) )
```

```
## [1] 4
```

```r
> # cf.
> table( Orthodont$Sex, Orthodont$age )
```

```
##         
##           8 10 12 14
##   Male   16 16 16 16
##   Female 11 11 11 11
```

```r
> is.matrix( table( Orthodont$Sex, Orthodont$age ) )
```

```
## [1] TRUE
```

```r
> dim ( table( Orthodont$Sex, Orthodont$age ) ) # r x c matrix
```

```
## [1] 2 4
```


## 학교 자료

학교 자료는 한눈에 파악할 수 있는 간단한 테이블들로 구성되어 있으므로 표 결합을 연습할 때 좋다. 결합과 관련하여 R에서 통일된 용어는 없다. SQL의 용어를 빌려서 적용하면 이렇다.

1) 교집합 결합(inner join): 왼쪽 테이블과 오른쪽 테이블을 "열쇠" 기준으로 결합하되 양쪽에 공통적으로 존재하는 레코드(행)에 국한한다.
2) 좌측중심 결합(left join): "열쇠"를 기준으로 좌우를 결합하되 오른쪽에는 없더라도 왼쪽에만 있으면 NA를 채워 넣어서 레코드 수를 유지한다.
3) 우측중심 결합(right join): 좌측중심 결합의 거울상으로 보면 된다.
4) 좌우 결합(full join): 좌측중심 결합과 우측중심 결합의 합집합으로 볼 수 있다.

단일 열쇠를 이용하는 모든 결합을 R에서는 merge 함수로 할 수 있다.

### 교집합 결합(inner join)

학교 자료를 id에 따라 교집합 결합을 하면 다음처럼 보이게 된다.


```r
> merge(schools_left, schools_right, by.x = "id", by.y = "id")
```

```
##   id           left_school          right_school
## 1  1     Oak Street School     Oak Street School
## 2  2 Roosevelt High School Roosevelt High School
## 3  6 Jefferson High School Jefferson High School
```

### 좌측중심 결합(left join)

all.x = TRUE 플래그가 관건이다. 왼쪽 테이블에는 있지만 오른쪽 테이블에는 없는 5번 학교 자리의 오른쪽 이름이 NA이다.


```r
> merge(schools_left, schools_right, by.x = "id", by.y = "id", all.x = TRUE)
```

```
##   id              left_school          right_school
## 1  1        Oak Street School     Oak Street School
## 2  2    Roosevelt High School Roosevelt High School
## 3  5 Washington Middle School                  <NA>
## 4  6    Jefferson High School Jefferson High School
```

###  우측중심 결합(right join)

예측할 수 있듯이 all.y = TRUE 플래그가 관건이다. 이번에는 왼쪽에는 없고 오른쪽에만 있는 3번 4번 학교 자리가 비어 있다.


```r
> merge(schools_left, schools_right, by.x = "id", by.y = "id", all.y = TRUE)
```

```
##   id           left_school          right_school
## 1  1     Oak Street School     Oak Street School
## 2  2 Roosevelt High School Roosevelt High School
## 3  3                  <NA>   Morrison Elementary
## 4  4                  <NA>  Chase Magnet Academy
## 5  6 Jefferson High School Jefferson High School
```

### 좌우 결합(full join)

all.x = TRUE, all.y = TRUE 


```r
> merge(schools_left, schools_right, by.x = "id", by.y = "id", all.x = TRUE, all.y = TRUE)
```

```
##   id              left_school          right_school
## 1  1        Oak Street School     Oak Street School
## 2  2    Roosevelt High School Roosevelt High School
## 3  3                     <NA>   Morrison Elementary
## 4  4                     <NA>  Chase Magnet Academy
## 5  5 Washington Middle School                  <NA>
## 6  6    Jefferson High School Jefferson High School
```

## nycflight 자료
### 총 항공기의 숫자

각 테이블(구체적으로 R에서는 data.frame)의 메타데이터 정보를 일견하였다면 개별 항공기의 정보는 planes 테이블에 있음을 알고 있다. 총 레코드 숫자와 꼬리번호의 고유값 숫자가 3322대로 일치함을 알 수 있다. 


```r
> nrow(planes)
```

```
## [1] 3322
```

```r
> length( unique(planes$tailnum) ) 
```

```
## [1] 3322
```
 
 

### 총 항공기 이륙의 숫자

항공기의 이착륙 정보가 flights 테이블에 있음을 알고 있다. 이 테이블에는 고유 키가 없으므로 총 레코드의 숫자가 총 이륙의 숫자라고 보아도 잠정적으로 문제가 되지 않을 것이다. 33만 6776회이다.


```r
> nrow(flights)
```

```
## [1] 336776
```

```r
> head(flights)
```

```
##   year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
## 1 2013     1   1      517            515         2      830            819
## 2 2013     1   1      533            529         4      850            830
## 3 2013     1   1      542            540         2      923            850
## 4 2013     1   1      544            545        -1     1004           1022
## 5 2013     1   1      554            600        -6      812            837
## 6 2013     1   1      554            558        -4      740            728
##   arr_delay carrier flight tailnum origin dest air_time distance hour minute
## 1        11      UA   1545  N14228    EWR  IAH      227     1400    5     15
## 2        20      UA   1714  N24211    LGA  IAH      227     1416    5     29
## 3        33      AA   1141  N619AA    JFK  MIA      160     1089    5     40
## 4       -18      B6    725  N804JB    JFK  BQN      183     1576    5     45
## 5       -25      DL    461  N668DN    LGA  ATL      116      762    6      0
## 6        12      UA   1696  N39463    EWR  ORD      150      719    5     58
##             time_hour    speed
## 1 2013-01-01 05:00:00 592.0705
## 2 2013-01-01 05:00:00 598.8370
## 3 2013-01-01 05:00:00 653.4000
## 4 2013-01-01 05:00:00 826.7541
## 5 2013-01-01 06:00:00 630.6207
## 6 2013-01-01 05:00:00 460.1600
```
 
flights 테이블은 변수가 많고 레코드의 수도 많기 때문에 한 화면에 모두를 출력하는 것은 의미가 없다. 두 번째 명령으로 앞에서 여섯 개의 레코드만 출력했지만 옆으로도 출력이 넘쳤다. 날짜, 출발시각, 도착시각, 출발지, 도착지만 출력했다.


```r
> head ( subset(flights, select = c(month, day, dep_time, arr_time, origin, dest) ) ) #
```

```
##   month day dep_time arr_time origin dest
## 1     1   1      517      830    EWR  IAH
## 2     1   1      533      850    LGA  IAH
## 3     1   1      542      923    JFK  MIA
## 4     1   1      544     1004    JFK  BQN
## 5     1   1      554      812    LGA  ATL
## 6     1   1      554      740    EWR  ORD
```

subset 함수 없이 R의 브래킷만으로도(실은 브래킷도 함수다) 구현할 수 있다. 다음 명령은 위와 동일하다.


```r
> flights[1:6, c("month", "day", "dep_time", "arr_time", "origin", "dest")]
```

```
##   month day dep_time arr_time origin dest
## 1     1   1      517      830    EWR  IAH
## 2     1   1      533      850    LGA  IAH
## 3     1   1      542      923    JFK  MIA
## 4     1   1      544     1004    JFK  BQN
## 5     1   1      554      812    LGA  ATL
## 6     1   1      554      740    EWR  ORD
```

행도 번호로 취했듯이 선택하고자 하는 열도 번호만 고르는 것이 더 편리할 때도 있다.


```r
> flights[1:6, c(2:4, 7, 13, 14)]
```

```
##   month day dep_time arr_time origin dest
## 1     1   1      517      830    EWR  IAH
## 2     1   1      533      850    LGA  IAH
## 3     1   1      542      923    JFK  MIA
## 4     1   1      544     1004    JFK  BQN
## 5     1   1      554      812    LGA  ATL
## 6     1   1      554      740    EWR  ORD
```

\$ 함수와 cbind 함수를 결합해서 이렇게 쓰기도 하지만 cbind의 결과물은 언제나 data.frame이 아니라 matrix이므로 자료형이 가장 높은 수준으로 강제로 변환됨을 명심해야 한다. 다음 명령문은 월과 도착지를 단순히 열로 묶은 것이 아니라 도착지 이름이 문자열이기 때문에 월의 숫자마저 문자열로 뒤바뀌었음을 알 수 있다.


```r
> cbind( flights$month, flights$dest )[1:6, ]
```

```
##      [,1] [,2] 
## [1,] "1"  "IAH"
## [2,] "1"  "IAH"
## [3,] "1"  "MIA"
## [4,] "1"  "BQN"
## [5,] "1"  "ATL"
## [6,] "1"  "ORD"
```


#### 월별/요일별 이륙 항공기의 숫자

월 이름은 테이블에 입력되어 있지만 요일은 입력되어 있지 않다. R은 weekdays 함수를 내장하고 있어서 Date 클래스의 날짜를 인수로 전달하면 요일을 알 수 있다. Date 클래스는 문자열로 된 날짜에 as.Date 선언만으로 얻을 수 있다.


```r
> weekdays( as.Date("2020-06-09") ) # test
```

```
## [1] "Tuesday"
```

```r
> table(flights$month)
```

```
## 
##     1     2     3     4     5     6     7     8     9    10    11    12 
## 27004 24951 28834 28330 28796 28243 29425 29327 27574 28889 27268 28135
```

```r
> table( weekdays( as.Date(flights$time_hour) ) )
```

```
## 
##    Friday    Monday  Saturday    Sunday  Thursday   Tuesday Wednesday 
##     50308     50690     38720     46357     50219     50422     50060
```

요일이 일월화수목금토 순서가 아니라 알파벳 순서로 출력되는 것이 마땅치 않아 보이므로 다음으로 코딩을 수정했다. 문자열 요일을 요인으로 선언하면서 레벨을 강제로 지정하는 방식이다. factor(... , levels = )의 형식이다.


```r
> (t <- table( factor(weekdays(as.Date(flights$time_hour)), levels = c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday") ) ))  #
```

```
## 
##    Sunday    Monday   Tuesday Wednesday  Thursday    Friday  Saturday 
##     46357     50690     50422     50060     50219     50308     38720
```

연 합계와 요일 평균을 얻고 이름을 붙였다. 


```r
> t <- c(t, sum(t), sum(t) / 7)
> names(t)[8:9] <- c("Sum", "Day-Avg")
> print(t)
```

```
##    Sunday    Monday   Tuesday Wednesday  Thursday    Friday  Saturday       Sum 
##  46357.00  50690.00  50422.00  50060.00  50219.00  50308.00  38720.00 336776.00 
##   Day-Avg 
##  48110.86
```


 

#### 총 착륙 횟수와 공항의 이름 결합하기

이륙 공항은 세 군데뿐이지만 착륙 공항은 전국에 걸쳐서 기록되어 있다. 전국의 공항을 연간 착륙의 수로 열거하여 10위까지를 찾은 뒤 공항의 실제 이름을 가져다 붙였다. match 함수가 초보적인 형태의 결합을 구현하는 열쇠를 찾는 구실을 한다.


```r
> # Using match
> t <- sort( table( flights$dest ), decreasing = TRUE)[1:10]
> names(t) <- airports$name[match(names(t), airports$faa)]
> print(t)
```

```
##                 Chicago Ohare Intl    Hartsfield Jackson Atlanta Intl 
##                              17283                              17215 
##                   Los Angeles Intl General Edward Lawrence Logan Intl 
##                              16174                              15508 
##                       Orlando Intl             Charlotte Douglas Intl 
##                              14082                              14064 
##                 San Francisco Intl     Fort Lauderdale Hollywood Intl 
##                              13331                              12055 
##                         Miami Intl      Ronald Reagan Washington Natl 
##                              11728                               9705
```
 
표 t의 이름에 처음에는 연방항공국 코드인 faa가 들어 있고 그것을 airports 테이블의 faa 변수에서 일치하는 것을 찾아서 그 이름을 name 변수에서 골라서 다시 표 t의 이름에 가져다 붙이는 과정이다. R에서 자료 행렬을 결합하고자 할 때 쓸 수 있는 더 복잡한 함수는 merge인데 merge 함수는 벡터에는 작용하지 않아서 t를 데이터프레임으로 만들면서 다시 시도하였다.


```r
> # Using merge
> t <- sort( table( flights$dest ), decreasing = TRUE)[1:10]
> t <- data.frame(t)
> merge(t, airports, by.x = "Var1", by.y = "faa", sort = FALSE)[, 1:3]
```

```
##    Var1  Freq                               name
## 1   ORD 17283                 Chicago Ohare Intl
## 2   ATL 17215    Hartsfield Jackson Atlanta Intl
## 3   LAX 16174                   Los Angeles Intl
## 4   BOS 15508 General Edward Lawrence Logan Intl
## 5   MCO 14082                       Orlando Intl
## 6   CLT 14064             Charlotte Douglas Intl
## 7   SFO 13331                 San Francisco Intl
## 8   FLL 12055     Fort Lauderdale Hollywood Intl
## 9   MIA 11728                         Miami Intl
## 10  DCA  9705      Ronald Reagan Washington Natl
```

정규표현식인 grep 함수를 이용하는 다음의 코드는 일견할 때 매우 초보적인 논리에 입각해서 자료 결합을 구현하는 것처럼 보이지만 하나의 열쇠 변수(여기서는 faa)가 두 개 이상의 행과 일치할 때는  의도하지 못했던 결과를 초래할 수 있으므로 자료의 구조와 내용을 모르는 경우에는 사용하지 않는 것이 좋다.



```r
> # Using regular expression
> t <- sort( table( flights$dest ), decreasing = TRUE)[1:10]
> names(t) <- airports$name[sapply(names(t), function(x) grep(x, airports$faa))]
> print(t)
```

```
##                 Chicago Ohare Intl    Hartsfield Jackson Atlanta Intl 
##                              17283                              17215 
##                   Los Angeles Intl General Edward Lawrence Logan Intl 
##                              16174                              15508 
##                       Orlando Intl             Charlotte Douglas Intl 
##                              14082                              14064 
##                 San Francisco Intl     Fort Lauderdale Hollywood Intl 
##                              13331                              12055 
##                         Miami Intl      Ronald Reagan Washington Natl 
##                              11728                               9705
```

마지막으로 반복문을 이용하는 방법이 있는데, 작은 크기의 자료에서는 문제가 초래되지 않지만 긴 자료에서는 코드 효율이 나쁘다. 다만, 자료를 결합하는 조건이 복잡한 경우에는 최선의 선택이 되기도 할 것이다. 바로 위에서 쓴 것처럼 grep 함수를 반복문 안에서 사용해도 되지만 논리적인 전개를 위하여 두 개의 반복문을 중첩하였다.


```r
> # Using loop
> t <- sort( table( flights$dest ), decreasing = TRUE)[1:10]
> for (i in 1:length(t))
+ 	for (j in 1:nrow(airports))
+ 	    if (names(t)[i] == airports$faa[j]) names(t)[i] <- airports$name[j]
> print(t)
```

```
##                 Chicago Ohare Intl    Hartsfield Jackson Atlanta Intl 
##                              17283                              17215 
##                   Los Angeles Intl General Edward Lawrence Logan Intl 
##                              16174                              15508 
##                       Orlando Intl             Charlotte Douglas Intl 
##                              14082                              14064 
##                 San Francisco Intl     Fort Lauderdale Hollywood Intl 
##                              13331                              12055 
##                         Miami Intl      Ronald Reagan Washington Natl 
##                              11728                               9705
```
자료철 결합은 자료를 처리할 때 가장 중요한 비중을 차지한다고 하여도 손색이 없으므로  간단한 예로 가능한 절차를 소개해 보았으니 상황에 맞는 것을 골라서 쓰면 된다. 여기서 소개하지 않은 마지막 절차는 패키지 dplyr에 포함된 inner_join, left_join, right_join, full_join 등의 함수이다. RDB의 대명사인 SQL의 쿼리 문법에 그대로 대응해서 사용할 수 있도록 편리하게 만들어져 있으면서 상당히 빠른 실행속도를 자랑하지만 R의 data.frame을 tibble 형식으로 변환하는 수고와 희생을 치러야 하기에 이 수업에서는 dplyr 패키지를 처음부터 배제하였다. 




### 시카고 인근 공항

가령 이와 같은 형식의 질문이 주어질 수 있다. 뉴욕에서 시카고로 가는 가장 빠른 항공편은 과연 어느 공항에서 어느 공항으로 갔으며, 며칠에 출발했을까? 우선, 공항 이름을 소문자로 바꾸었을 때 "chicago"를 포함하는 공항은 모두 다섯 곳이다.


```r
> grep("chicago", tolower(airports$name), value = TRUE)
```

```
## [1] "gary chicago international airport"    
## [2] "chicago midway intl"                   
## [3] "chicago ohare intl"                    
## [4] "chicago executive"                     
## [5] "chicago rockford international airport"
```
 
다섯 공항에 착륙한 항공편을 모두 찾는 명령을 한 줄로 쓸 수 있다. 임시 자료철 `t`에 저장한 뒤 air_time이 최소값을 가지는 행을 골라냈다.


```r
> subset(flights, dest %in% airports$faa[ grep("chicago", tolower(airports$name)) ]) -> t
> which.min(t$air_time)
```

```
## [1] 18823
```
행 번호가 18823으로 나왔기에 flights의 18823번째 행을 분철해내면 엉뚱한 행이 나온다.


```r
> flights[18823, ]
```

```
##       year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
## 18823 2013     1  22     1637           1619        18     1729           1723
##       arr_delay carrier flight tailnum origin dest air_time distance hour
## 18823         6      EV   4271  N11187    EWR  ALB       30      143   16
##       minute           time_hour speed
## 18823     19 2013-01-22 16:00:00 457.6
```
이 번호는 이미 시카고행 항공만 따로 고른 t 자료철의 번호이다.


```r
> t[18823, ]
```

```
##        year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
## 299649 2013     8  21     1604           1555         9     1707           1729
##        arr_delay carrier flight tailnum origin dest air_time distance hour
## 299649       -22      UA    220  N489UA    EWR  ORD       87      719   15
##        minute           time_hour    speed
## 299649     55 2013-08-21 15:00:00 793.3793
```

### 항공기의 실제 속도

항공기의 이론적인 속력은 planes 테이블에 들어 있지만 결측값이 많고, 매 항공마다의 거리와 시간을 이융하면 실제 항공 속도를 계산할 수 있다. 주어진 대로 계산하면 mile/h로 계산되는데 처음부터 km/h로 변환해서 속도를 얻는다. 새로 만들어진 변수 flights\$speed에 속성까지 부여했다.


```r
> flights$speed <- with(flights, distance * 1.6 / air_time * 60)
> attributes(flights$speed) <- list(meaning = "real flying speed", unit = "km/h")
> str(flights$speed)
```

```
##  num [1:336776] 592 599 653 827 631 ...
##  - attr(*, "meaning")= chr "real flying speed"
##  - attr(*, "unit")= chr "km/h"
```
 
### 항공기의 연간 총 비행시간, 횟수, 선령

항공기의 선령은 2013년에서 제작 연도(planes\$year)를 빼서 얻을 수 있고 연간 총 비행시간은 개별 비행시간(flights\$air_time)을 항공기 별로 합해서 얻을 수 있다. 항공기별로 특정한 관찰값에 대해서 연산(여기서는 합)하려면 테이블을 개념적으로 분철할 수 있어야 한다. 가령 항공기 꼬리번호 "993DL"의 연간 항공시간의 모두는 이렇게 분철해 낸다. 길이와 합을 구하면 비행 횟수와 총시간이 계산되는데 비행시간에는 결측값이 많아서 na.rm 인자를 TRUE로 놓지 않으면 NA로 답할 것이다. 총 55회 비행에 총 6837분을 비행했다. 비행시간이 기록되지 않은 비행은 총 2회이다. 

```r
> subset(flights, tailnum == "N993DL")$air_time
```

```
##  [1] 152 184 129 146 148 154 132 139  95 138 136 116 122  79 137 115 110 161  91
## [20]  89 139 132 116 130  76 159  NA 191 176 135 135 147 147 111 114 128 125 136
## [39] 112 109  NA 114 127 129 104 110 120 155  97 112 115 146 129 141 147
```

```r
> length( subset(flights, tailnum == "N993DL")$air_time )
```

```
## [1] 55
```

이어지는 세 번째 코드는 눈여겨보라. `is.na` 함수로 `air_time`이 결측값인 자리를 찾았고 결측값의 개수를 셀 때 `length`가 아니라 `sum`을 사용했다.


```r
> sum( is.na(subset(flights, tailnum == "N993DL")$air_time) )
```

```
## [1] 2
```

```r
> sum( subset(flights, tailnum == "N993DL")$air_time, na.rm = TRUE) 
```

```
## [1] 6837
```

NA가 포함된 벡터에서 is.na 함수는 NA를 TRUE로, NA가 아닌 것을 FALSE로 보는 내부 연산을 고려할 때 TRUE의 개수는 length가 아니라 sum으로 찾아야 하는 게 원리이다.


```r
> x <- c(1, 3, 11, NA, 9, NA, 2, 5, NA)
> length(x)
```

```
## [1] 9
```

```r
> sum(x)
```

```
## [1] NA
```

```r
> x_na <- is.na( x )
> length(x_na)
```

```
## [1] 9
```

```r
> sum(x_na)
```

```
## [1] 3
```
 
모든 꼬리번호에 대해서 통째로 합산을 할 때 테이블에 대해 연산함수를 부가하는 함수인 `tapply`가 무난하다. `tapply` 함수로 테이블을 분철해서 비행시간을 합산하려면 다음의 문형을 준수한다. 결과는 벡터인 표이므로 데이터프레임으로 바꾸었다.


```r
> t_sum <- tapply(flights$air_time, flights$tailnum, sum, na.rm = TRUE )
> t_freq <- tapply(flights$air_time, flights$tailnum, length )
> Air_times <- data.frame(
+ 	sum = t_sum,
+ 	freq = t_freq)
```
새로이 만들어진 `Air_times` 테이블에는 항공기별 연간 총 비행시간과 비행 횟수가 들어 있으며 항공기 식별번호는 행이름에만 들어 있다. 어차피 꼬리번호는 제작연도를 가져올 때만 필요하므로 `merge`의 인수에서 `by.x = "row.names"`처럼 맞춘다. 결합된 자료철에 새 이름인 `Air_times2`를 배당했다.


```r
> Air_times2 <- merge(Air_times, planes[, c("tailnum", "year")], 
+ 	by.x = "row.names", by.y = "tailnum")
> Air_times2$age <- 2013 - Air_times2$year
```
선령에 대해서 


```r
> par(mfrow = c(1, 2))
> plot(sum ~ age, Air_times2)
> plot(freq ~ age, Air_times2)
```

![plot of chunk unnamed-chunk-35](figure/unnamed-chunk-35-1.png)
  

### 이륙지연, 요일, 날씨

처리해야 할 작업이 크다면 자료를 분철부터 하는 게 좋다. flights 테이블에 날씨 정보를 붙여서 임시 테이블을 만드는 작업으로 시작하는 게 좋다. 단순히 날씨 정보만 붙인 게 아니라 항공 정보에서 항공 번호, 항공사, 항공기번호, 출발시각, 출발지연을 제외한 나머지 정보는 지우고 시작했다.


```r
> flightsWeather <- merge(
+ 	flights[, c("origin", "flight", "carrier", "tailnum", "month", "day", "hour", 
+ 		"dep_time", "dep_delay")], 
+ 	weather[, c("origin", "month", "day", "hour", "temp",  "dewp", "humid", "wind_dir",
+ 		"wind_speed", "wind_gust", "precip", "pressure", "visib")], 
+ 	by.x = c("origin", "month", "day", "hour"), 
+ 	by.y = c("origin", "month", "day", "hour"))
```

새로 만든 데이터프레임 flightsWeather에 요일을 만들어서 붙인다. 여기서 사용된 paste 함수는 인수로 전달 받은 값들을 모두 합쳐서 하나의 문자열을 만드는 접착제 구실을 한다. 열리고 닫히는 괄호를 잘 헤아려야 오류를 초래하지 않는다.


```r
> flightsWeather$weekday <- with(flightsWeather, 
+ 	weekdays( as.Date (paste(2013, month, day, sep = "-")) )
+ 	) # with
```

앞서 처리했던 것처럼 weekday 변수를 원하는 순서의 요인으로 다시 맞춘다. 들여쓰기는 자유롭게 하되, 최소단위 의미소를 분할해서는 아니 되며, 알아보기 쉬우면 그만이다.


```r
> flightsWeather$weekday <- factor( 
+ 	flightsWeather$weekday, 
+ 		levels = c("Sunday", "Monday", "Tuesday", 
+ 			"Wednesday", "Thursday", "Friday", 
+ 			"Saturday") 
+ 	)
> str(flightsWeather)	
```

```
## 'data.frame':	335220 obs. of  19 variables:
##  $ origin    : chr  "EWR" "EWR" "EWR" "EWR" ...
##  $ month     : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ day       : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ hour      : int  10 10 10 10 10 10 10 10 10 10 ...
##  $ flight    : int  1294 1625 4277 4479 4322 985 503 688 455 75 ...
##  $ carrier   : chr  "UA" "UA" "EV" "EV" ...
##  $ tailnum   : chr  "N77258" "N81449" "N11189" "N11544" ...
##  $ dep_time  : int  1031 1005 1030 1056 1044 1038 1003 1105 1044 955 ...
##  $ dep_delay : int  1 5 -5 -3 -1 8 -7 18 -1 -5 ...
##  $ temp      : num  41 41 41 41 41 41 41 41 41 41 ...
##  $ dewp      : num  28 28 28 28 28 ...
##  $ humid     : num  59.6 59.6 59.6 59.6 59.6 ...
##  $ wind_dir  : int  260 260 260 260 260 260 260 260 260 260 ...
##  $ wind_speed: num  13.8 13.8 13.8 13.8 13.8 ...
##  $ wind_gust : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ precip    : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ pressure  : num  1012 1012 1012 1012 1012 ...
##  $ visib     : num  10 10 10 10 10 10 10 10 10 10 ...
##  $ weekday   : Factor w/ 7 levels "Sunday","Monday",..: 3 3 3 3 3 3 3 3 3 3 ...
```

여기까지 만들어졌으면 몇 가지 분석을 해 볼 수 있겠다. 요일별로 출발지연의 평균과 중간값이 어떻게 다른가?


```r
> with(flightsWeather, 
+ 	tapply(dep_time, weekday, function(x) c(mean(x), median(x))
+ 		)
+ 	)
```

```
## $Sunday
## [1] NA NA
## 
## $Monday
## [1] NA NA
## 
## $Tuesday
## [1] NA NA
## 
## $Wednesday
## [1] NA NA
## 
## $Thursday
## [1] NA NA
## 
## $Friday
## [1] NA NA
## 
## $Saturday
## [1] NA NA
```

원하는 결과를 얻지 못한 이유는 dep_time 안에 결측값(NA)들이 포함되었기 때문에 mean 함수와 median 함수가 계산을 포기했던 때문이다. R의 대부분의 함수들은 NA가 포함된 인수를 받으면 NA로 답하는 경우가 많다는 사실을 명심하라. na.rm = TRUE 옵션을 붙여서 NA를 어떻게 처리할지 적시한 채로 다시 분석할 수 있다.


```r
> with(flightsWeather, 
+ 	tapply( dep_delay, weekday, function(x)  
+ 		c(mean(x, na.rm = TRUE), median(x, na.rm = TRUE) )
+  		)
+ 	)
```

```
## $Sunday
## [1] 11.59224 -2.00000
## 
## $Monday
## [1] 14.76291 -1.00000
## 
## $Tuesday
## [1] 10.69781 -2.00000
## 
## $Wednesday
## [1] 11.81277 -2.00000
## 
## $Thursday
## [1] 16.03989 -1.00000
## 
## $Friday
## [1] 14.69008 -1.00000
## 
## $Saturday
## [1]  7.666614 -2.000000
```

소수점 미만의 값을 반올림하려면 이미 알고 있는 함수 round를 쓴다.


```r
> with(flightsWeather, 
+ 	tapply( dep_delay, weekday, function(x)  
+ 		round( c(mean(x, na.rm = TRUE), 
+ 			median(x, na.rm = TRUE) ), 0)
+  		)
+ 	)
```

```
## $Sunday
## [1] 12 -2
## 
## $Monday
## [1] 15 -1
## 
## $Tuesday
## [1] 11 -2
## 
## $Wednesday
## [1] 12 -2
## 
## $Thursday
## [1] 16 -1
## 
## $Friday
## [1] 15 -1
## 
## $Saturday
## [1]  8 -2
```

출발이 지연됐던 때와 아닐 때 시계(visib)의 평균이 어떻게 다른지 살피려면 tapply를 응용해서 적용한다.


```r
> with( flightsWeather, 
+ 	tapply(visib, dep_delay > 0, mean) )
```

```
##    FALSE     TRUE 
## 9.399725 9.114153
```



## 과제 1

비행기의 출발이 지연됐던 시간과 아닐 때의 시계뿐 아니라 나머지 기상정보(기온, 응결점, 습도, 바람 등)의 평균 차이를 구하라.





## 과제 2

도착이 지연된 항공기를 공항 및 항공사 별로 집계를 내되, 공항과 항공사의 이름은 코드가 아닌 이름으로 표현하라. (한 화면에 보이지 않아도 무방하다)













