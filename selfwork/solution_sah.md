# SAH 데이터 - 추천 답안 R 스크립트

* 이윤석
* 2020년 6월 20일

---

## 데이터 로딩



```r
> patients <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/patients.csv", header = TRUE, stringsAsFactors = FALSE) #
> str(patients)
```

```
## 'data.frame':	1000 obs. of  3 variables:
##  $ id        : int  649 112 382 205 980 977 793 924 765 917 ...
##  $ gender    : chr  "M" "M" "M" "M" ...
##  $ birth_year: int  1977 1963 1965 1936 1948 1932 1967 1936 1939 1952 ...
```

```r
> admits <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/admits.csv", header = TRUE, stringsAsFactors = FALSE) #
> admits$admit_year <- as.integer( substring(admits$admit_date, 1, 4) )
> str(admits)
```

```
## 'data.frame':	562 obs. of  6 variables:
##  $ id            : int  485 89 789 956 469 609 249 255 130 130 ...
##  $ admit_id      : int  1902 4024 7762 14146 23171 34008 40243 48520 62152 62153 ...
##  $ type          : chr  "spontaneous SAH" "spontaneous SAH" "spontaneous SAH" "spontaneous SAH" ...
##  $ admit_date    : chr  "2008-09-26" "2012-07-02" "2009-04-22" "2015-09-14" ...
##  $ discharge_date: chr  "2008-09-26" "2012-08-24" "2009-06-03" "2016-01-28" ...
##  $ admit_year    : int  2008 2012 2009 2015 2009 2008 2016 2018 2011 2011 ...
```

```r
> deaths <- read.csv("https://raw.githubusercontent.com/ylee03/r_for_students_2020/master/dataset/deaths.csv", header = TRUE, stringsAsFactors = FALSE) #
> deaths$death_year <- as.integer( substring(deaths$death_date, 1, 4) )
> str(deaths)
```

```
## 'data.frame':	57 obs. of  3 variables:
##  $ id        : int  609 683 356 604 342 614 823 324 973 568 ...
##  $ death_date: chr  "2008-02-10" "2015-08-19" "2015-07-27" "2007-03-04" ...
##  $ death_year: int  2008 2015 2015 2007 2007 2014 2008 2010 2009 2008 ...
```


## 조별 과제 1, 2, 3, 4
SAH의 유형(군)에 따른 환자들의 입원 시점의 나이를 평균, 표준편차로 비교하고 성별의 도수를 비교한다.


```r
> t <- merge(admits, subset( patients, select = c(id, gender, birth_year) ),
+ 	by = "id") # inner-join
> with(t, 
+ 	tapply( admit_year - birth_year, type, 
+ 		function(x) c( mean(x), sd(x) ) )  ) #
```

```
## $`spontaneous SAH`
## [1] 56.21914 14.44231
## 
## $`traumatic SAH`
## [1] 55.43277 19.23982
```

```r
> t0 <- table(t$type, t$gender) 
> t0 <- cbind(t0, apply(t0, 1, 
+ 	function(x) round( x[1] / sum(x), 3 )) )
> colnames(t0)[3] <- "proportion (female)"
> print(t0)
```

```
##                   F   M proportion (female)
## spontaneous SAH 186 138               0.574
## traumatic SAH    52 186               0.218
```

SAH의 유형(군)에 따른 사망/생존을 비교한다.


```r
> t <- merge(admits, subset(deaths, select = c(id, death_year)), 
+ 	by = "id", all.x = TRUE) # left-join
> t0 <- table(t$type, is.na(t$death_year))
> colnames(t0) <- c("dead", "alive")
> t0 <- cbind(t0, apply(t0, 1, 
+ 	function(x) round( x[1] / sum(x), 3 )) )
> colnames(t0)[3] <- "proportion (dead)"
> print(t0)
```

```
##                 dead alive proportion (dead)
## spontaneous SAH   33   291             0.102
## traumatic SAH     24   214             0.101
```


## 중첩 환자 제외
두 번 이상 입원했던 환자가 여러 행에 등장하며서 그 환자의 나이, 성별 비교 때 또는 사망 여부 비교 때 여러 번 집계된다는 문제가 있었다. 따라서 한 환자 당 한 행씩만 골라내는 것이 필요한데, 나이와 성별을 비교할 때는 첫 입원 때 행을 쓰는 것이 타당해 보이고 사망 때는 마지막 입원 때 행을 쓰는 것이 타당해 보인다.


중첩 없는 깔끔한 목표 테이블 tm의 형태를 먼저 구상했는데, 뒤에 나올 개인과제까지 포괄하고자 다음과 같은 행을 가지도록 구상했다.


1. id : 환자 번호
2. gender : 성별
3. birth_year : 생년
4. death_date : 사망날짜
5. death_year : 사망년도
6. first_admit : 최초 입원일
7. last_discharge : 마지막 퇴원일
8. type : SAH 유형
9. age1 : 첫 입원 당시 연령
10. age2 : 마지막 입원 당시 연령
11. dur : 첫 입원에서 마지막 퇴원까지의 날짜



`admits` 테이블로부터 고유 `id`를 뽑아서 `data.frame`을 만든다.


```r
> id = unique(admits$id)
> tm <- as.data.frame(id)
> nrow(tm)
```

```
## [1] 500
```

뒤에서 무슨 조작을 하더라도 행 수 500이 넘지 않아야 한다. `gender`와 `birth_year`는 `patients` 테이블에 있다.


```r
> tm <- merge(tm, subset(patients, select = c(id, gender, birth_year)), 
+ 	by = "id") # inner-join
> nrow(tm)
```

```
## [1] 500
```

`death_date`와 `death_year`는 `deaths` 테이블에 있다.


```r
> tm <- merge(tm, subset(deaths, select = c(id, death_date, death_year)), 
+ 	by = "id", all.x = TRUE) # left-join
> nrow(tm)
```

```
## [1] 500
```

`first_admit`는 `admits` 테이블에서 환자별 첫 입원의 `admit_date`에 있고, `last_discharge`는 `admits` 테이블에서 환자별 마지막 입원의 `discharge_date`에 있다. `for` 반복구문을 만들어서 매 `id`마다 `admits` 테이블의 분철을 만들어서 첫 입원과 마지막 퇴원을 뽑는 게 절차의 요지이다.


```r
> tm$first_admit <- NA
> tm$last_discharge <- NA
> tm$type <- NA
> for (i in 1:nrow(tm)) {
+ 	u <- subset(admits, id == tm$id[i])
+ 	if (nrow(u) > 1) { # 여러 차례 입원
+ 		u <- u[order(u$admit_date), ]
+ 		tm$first_admit[i] <- u$admit_date[1]
+ 		tm$last_discharge[i] <- u$discharge_date[nrow(u)]
+ 	} else { # 한 번 입원
+ 		tm$first_admit[i] <- u$admit_date
+ 		tm$last_discharge[i] <- u$discharge_date
+ 	}
+ 	tm$type[i] <- u$type[1]	
+ }
> nrow(tm)
```

```
## [1] 500
```


병발 때 연령, 마지막 퇴원 때 연령, 사망까지의 날짜를 추출한다.  `age1`에는 병발 때 나이, `age2`는 마지막 입원 때 나이이고 `dur`에는 병발부터 사망까지의 기간을 날짜의 형태로 넣는다.


```r
> tm$age1 <- as.integer( substr(tm$first_admit, 1, 4) ) - tm$birth_year
> tm$age2 <- as.integer( substr(tm$last_discharge, 1, 4) ) - tm$birth_year
> tm$dur <- as.Date(tm$last_discharge) - as.Date(tm$first_admit)
```

이로써 의도했던 테이블이 완성되었다.


```r
> str( tm )
```

```
## 'data.frame':	500 obs. of  11 variables:
##  $ id            : int  1 2 3 4 5 10 12 17 19 22 ...
##  $ gender        : chr  "M" "F" "M" "M" ...
##  $ birth_year    : int  1973 1974 1937 1943 1958 1958 1940 1973 1971 1950 ...
##  $ death_date    : chr  NA NA NA "2012-10-28" ...
##  $ death_year    : int  NA NA NA 2012 NA NA NA NA 2016 NA ...
##  $ first_admit   : chr  "2011-06-22" "2011-12-19" "2016-12-06" "2012-10-28" ...
##  $ last_discharge: chr  "2011-07-23" "2011-12-20" "2017-02-06" "2012-10-28" ...
##  $ type          : chr  "spontaneous SAH" "traumatic SAH" "spontaneous SAH" "traumatic SAH" ...
##  $ age1          : int  38 37 79 69 60 59 70 45 45 68 ...
##  $ age2          : int  38 37 80 69 60 59 70 45 45 68 ...
##  $ dur           : 'difftime' num  31 1 62 0 ...
##   ..- attr(*, "units")= chr "days"
```

---

병발 때 연령을  다시 비교한다.


```r
> with(tm,
+ 	tapply( age1, type, function(x) c(mean(x), sd(x)))
+ 	)
```

```
## $`spontaneous SAH`
## [1] 56.41935 14.59303
## 
## $`traumatic SAH`
## [1] 55.67873 19.24037
```

병발 때 성별도 다시 비교. 자발적으로 발생하는 유형은 여성에게 많이 보이고 외상으로 발생하는 유형은 남성에게 많이 관찰되는 점은 변하지 않았다.


```r
> u <- with(tm,
+ 	table(type, gender))
> u <- cbind(u, apply(u, 1, function(x) 
+ 	round( x[1] / sum(x), 3) ) )
> colnames(u)[3] <- "proportion (f)"
> print(u)	
```

```
##                   F   M proportion (f)
## spontaneous SAH 164 115          0.588
## traumatic SAH    50 171          0.226
```

## 개인별 과제 5, 6
SAH 첫 이환 후 사망까지 걸린 날짜가 며칠까지를 SAH에 의한 사망으로 볼 수 있을지 알기 어려우며 합리적인 도출보다는 전문가 합의의 영역에 놓였다고 판단한다. 주어진 데이터로부터 60-일 이내 사망 여부를 SAH 유형(군)별로 비교한다.


```r
> t0 <- with(tm,
+ 	table(type, !is.na(death_date) & dur < 61) )
> t0 <- cbind(t0, apply(t0, 1, 
+ 	function(x) round(x[2] / sum(x), 3) ))
> colnames(t0) <- c("alive", "dead", "porportion (dead)")
> print(t0)
```

```
##                 alive dead porportion (dead)
## spontaneous SAH   248   31             0.111
## traumatic SAH     200   21             0.095
```

사망 당시 환자의 연령은 SAH 유형(군)별로 어떻게 다른가?


```r
> with(subset(tm, !is.na(death_date) & dur < 61),
+ 	tapply(age2, type, function(x) c(mean(x), sd(x)))
+ 	)
```

```
## $`spontaneous SAH`
## [1] 58.35484 11.45294
## 
## $`traumatic SAH`
## [1] 67.52381 21.89433
```


## 데이터의 결함

학생 중 일부가 이 데이터의 세 가지 결함을 보고하였다. 실세상 자료임을 고려할 때는 데이터의 결함이라기보다는 출제자의 의도를 벗어난 사실이 데이터 안에 존재한다는 뜻이다. 


### deaths 테이블 1

두 번 입원한 환자는 존재하지만 두 번째 입원하여 사망한 환자는 존재하지 않는다. 그래서 admits 테이블에서 집계하여도 사망자의 수가 맞게 나온다. (생존자의 수는 입원의 숫자만큼 부풀지언정)

### deaths 테이블 2

사망자 57명 중에서 1명은 SAH 환자가 아니다. deaths 테이블의 id 중에서 admits 테이블에 존재하지 않는 id가 하나라는 뜻이다. 다음으로 찾을 수 있다. 


```r
> t <- deaths$id %in% admits$id
> sum( !t )
```

```
## [1] 1
```

```r
> deaths$id[ grep(TRUE, !t) ]
```

```
## [1] 802
```
id = 802 환자는 버려야 한다는 뜻이 된다. 이 답안에서는 admits 테이블을 기준으로 사망자를 추출했으므로 이 사실만으로는 문제가 발생할 수 없다. 

### admits 테이블

그렇다면 이 지점에서 설명하기 곤란한 사실이 한 가지 더 드러난다. 애초에 admits 테이블로 집계했던 (중첩 환자를 제외하지 않았던) 사망자의 수가 `33 + 24 = 57`로서, 56명이었어야 했는데 deaths 테이블의 행의 수와 일치한다는 점이다.  즉, 사망자 중에서 누군가는 두 번 집계되었다고 해석할 수 있다.

누구인지 찾아서 왜 그렇게 되었는지 알 필요가 있다. 맨 앞에서 만들었던 것처럼 admits와 deaths를 결합하되 사망자만 고른다.



```r
> t <- merge(admits, subset(deaths, select = c(id, death_year)), by = "id", all.x = TRUE) #
> t <- subset(t, !is.na(death_year))
```

두 번 이상 나온 id를 찾는다.


```r
> subset(t, id == names( which(table(t$id) > 1) ),
+ 	select = -c(admit_year, death_year))
```

```
##      id admit_id            type admit_date discharge_date
## 503 895  1267772 spontaneous SAH 2010-10-07     2010-10-18
## 504 895  1267769   traumatic SAH 2010-10-07     2010-10-18
```

`id = 895` 환자의 레코드에 이해할 수 없는 부분이 있음을 알 수 있다. 같은 입원에 두 개의 admit_id가 부여되었고 게다가 서로 다른 유형의 SAH가 부여되어 있다. 환자에게서 병력을 소상히 청취할 수 없는 경우에(가령, 자발성 파열로 의식을 잃으면서 넘어져서 머리를 다치는 경우 등) 두 가지 유형의 SAH 모두를 진단에 붙일 수도 있지만 admit_id가 다른 점에 대해서는 다른 방식의 설명이 필요하다(어쩌면, 자의로 퇴원했던 환자가 다른 병원을 떠돌다가 다시 입원하면서 새 admit_id를 받았는데 처음에는 몰랐던 외상 요소가 불거졌다든가 하는 등). 

이 레코드 때문에 실제 사망자가 56명임에도 57명으로 집계되었다. 이 환자를 첫 번째 진단을 기준으로 집계할지 두 번째 진단을 기준으로 집계하는 것이 좋을지 잘 모르겠으나, 이처럼 작은 규모의 데이터에서는, 아마도 기계적으로 분석한 결과의 뒤에 데이터의 문제점을 상술하여 두는 것이 정답과 가장 가까울 것이다. 



## 사족

여러 학생들이 중첩환자를 찾는 과정에 R 내장함수 `duplicated`를 적용하였다. 중첩된 행을 찾아서 첫 행만 남기고 삭제한 뒤 deaths 테이블을 결합하여 첫 입원부터 사망까지 계산하는 형식을 취했다. 디폴트 기능으로는 첫 행만을 남기는 제한점을 가지기 때문에 이 데이터에 적용할 바람직한 소거 방법이 아닐 것이라는 출제자의 예상과는 달리 학생들처럼 코딩을 하여도 데이터가 말썽을 부리지 않은 점은 다행이다.

한 가지들 더 이야기하자면, `duplicated` 함수의 디폴트 옵션은 첫 행 이후의 중첩 행들에 표시하는 것이지만 옵션 `fromLast = TRUE`로 설정하면 뒤에서부터 작업이 실행된다. 맨 뒤의 행만 남기고 앞부분의 중첩 부분을 일괄적으로 삭제할 수 있다는 점을 알아 두면 편리할 것이다. 이렇게 디폴트대로 첫 입원날짜를 뽑고, 다시 옵션을 바꾸면 마지막 퇴원날짜를 뽑을 수 있었을 것이다.

이 원고의 추천 답안에서는 중첩환자를 제거하기 위해서 `duplicated` 함수의 도움을 받지 않고 고유 id를 나열한 뒤에 필요한 변수를 차곡차곡 붙이는 절차를 행했다. 이렇게 하기 위해서 발휘한 기술은 일견 거창해 보이지만 for 반복구문을 써서, 매 반복주기마다 같은 id를 가진 입원 행들의 subset을 만든 것에 불과했다. R 프로그래머들이 자주 당면하는 제문제들의 현주소도 이와 다르지 않다. 당장의 함수 한 개를 더 알고 있는가 아닌가가  편리 측면에서 절실한 문제임에 틀림이 없으면서도 동시에, 처리할 업무의 "논리"를 꿰는 것을 필요한 모든 함수의 목록을 꿰는 일보다 더 중요시해야 한다. 실용적인 프로그래머라면 마땅히 적재적소에서  함수를 동원할 수 있어야 하면서 동시에, 프로그래밍이라는 것이 본질적으로 컨트롤을  통제하는 일이라는 사실을 인지하는 것이다. 

(끝)







