# 개인과제


## SAH (거미막밑출혈) 자료 가져오기

거미막하출혈은(subarachnoid hemorrhage; SAH) 뇌에 공급되는 주요동맥에 발생한 동맥류(꽈리)가 파열되면서 혈액이 거미막 밑으로 파급하면서 발생하는 중증질환이다. 일단 파열되어 출혈된 혈액의 양, 이차적으로  높아진 뇌의 압력, 동맥류의 위치, 2차 파열 여부, 불특정한 뇌혈관의 수축등으로 인한 뇌경색 등이 거미막하출혈 환자들의 예후를 결정하는 주요 요인들로 알려져 있지만, 이 자료를 모은 임상의사는 파열의 직접 원인이 외상성(traumatic)인지 또는 자발적(spontaneous)인지에 따라 이환된 환자들의 사망률이 다른지 궁금했다.


자료는 nycflights 자료 때처럼 세 개의 데이터프레임으로 분리되어 저장되어 있고 다음의 github 주소로부터 R로 가져올 수 있다.

### patients 테이블

개인들의 기초 정보가 든 patients 테이블은 세 개의 컬럼을 가진 1,000행의 자료철이다. id에 환자식별번호가, gender에 성별, birth_year에 출생년도가 정수로 들어 있다.


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

### admits 테이블

admits 테이블은 입퇴원 날짜, 진단명이 든 간단한 입원 요약 테이블이다. 한 행은 하나의 입원으로서 환자식별번호(id)로 patients 테이블, deaths 테이블과 연계된다. patients 테이블에는 생일이 없이 생년만 존재하므로 admits 테이블을 불러 올 때 입원날짜로부터 연도만 뽑아 미리 변수 admit_year를 만들어 두면 편리하다.


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
파열 원인에 따른 진단명은 type 변수에 들어 있다. 집계는 다음과 같다.


```r
> table(admits$type)
```

```
## 
## spontaneous SAH   traumatic SAH 
##             324             238
```
이 테이블은 SAH 환자들의 입원 테이블로서 총 526행(spontaneous 324명, traumatic 238명)이지만 이 안에 든 환자는 526명이 아니라 500명임을 다음 명령으로 알 수 있다.


```r
> length( unique( admits$id ) )
```

```
## [1] 500
```
환자 id의 수와 입원 admit_id의 수에 불일치가 있는 일은 실제로 아주 흔하다.  이 경우는 한 환자가 같은 진단명으로 여러 번 입원했기 때문이다.


### deaths 테이블

사망한 환자의 기록으로서, 조사기간 동안 사망한 57명의 사망일자를 담고 있다. 이 테을에서도 변수 death_year를 먼저 만들어 두면 편리하다.


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

이 테이블에 등장하지 않은 환자는 조사기간 동안 생존했다고 가정한다.


## 과제

### 1. 유형(군)에 따른 환자들의 나이를 평균, 표준편차로 비교하고 성별을 도수를 비교한다.



```r
> #
> #
```


### 2. 유형(군)에 따른 사망/생존을 비교한다.

만들어진 분철 t는 왼쪽 테이블을 중심으로 결합했으므로 deaths 테이블에는 없는 자료(즉, 조사 시점에 생존한 환자)는 NA로 입력되었다. 따라서 t$death_year가 NA일 때 0을 강제로 대입해서 표로 만들 때 0이면 생존, 0이 아니면 사망으로 보면 된다.


```r
> #
> #
```


### 3. 중첩 환자 제외

위에서 처리한 방식으로 테이블의 결합하면 두 번 이상 입원했던 환자가 여러 행으로 나오면서 나이, 성별 비교 때 또는 사망 여부 비교 때 여러 번 집계가 된다는 문제가 있다. 따라서 한 환자 당 한 행을 골라내는 것이 필요한데, 나이와 성별을 비교할 때는 첫 입원 때 행을 쓰는 것이 타당해 보이고 사망 때는 마지막 입원 때 행을 쓰는 것이 타당해 보인다.




```r
> #
> #
```

 여기에서  SAH의 유형별 연령의 평균과 표준편차, 성별을 비교했다.


```r
> #
> #
```

같은 방식으로, 하지만, 마지막 입원의 행만 추출해서 사망 여부를 비교한다.



```r
> #
> #
```

