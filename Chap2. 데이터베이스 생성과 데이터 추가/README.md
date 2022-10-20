# Chap 2. 데이터베이스 생성과 데이터 추가
## 2.1 MySQL 데이터베이스 생성
[Mac 기준]
1) MySQL Community Server 다운로드 후, 아래의 명령어를 터미널에 입력하여 mysql 명령줄 도구를 호출한다.
``` bash
$ cd /usr/local/mysql/bin
$ ./mysql -uroot -p
```

2) 아래 링크에서 sakila 예제 데이터베이스를 다운로드하여 로컬 폴더에 저장한다.
[https://dev.mysql.com/doc/index-other.html](https://dev.mysql.com/doc/index-other.html)


3) 아래 코드와 같이 sakila 예제 데이터베이스가 저장된 로컬 디렉토리를 대입하여 실행한다.
``` sql
mysql> source /Users/jooyoung/sakila-db/sakila-schema.sql;
mysql> source /Users/jooyoung/sakila-db/sakila-data.sql;
```

---

## 2.2 mysql 명령줄 도구 사용 방법
아래 코드와 같이 작업할 데이터베이스를 sakila로 변경할 수 있다.
``` bash
mysql> show databases;
mysql> use sakila;
```

아래와 같이 mysql 명령줄 도구를 호출할 때, 사용자 이름과 데이터베이스를 같이 저장할 수도 있다.
``` bash
$ ./mysql -uroot -p sakila;
```

- now() 함수는 현재 날짜와 시간을 리턴하는 내장 MySQL 함수이다.
- 아래와 같이 FROM절 없이 쿼리 실행이 가능한 것은 MySQL의 특징이다.
- 다른 DBMS에서는 이런 경우, 단일 데이터 행을 포함하는 dummy 라는 단일 열로 구성된 dual이라는 테이블을 제공한다. 따라서, 타 DBMS와 호환되는 쿼리를 작성해야 하는 경우에는 VER 2로 작성해야 한다.

``` sql
mysql> SELECT now(); // VER 1
mysql> SELECT now() FROM dual; // VER 2
```

---

## 2.3 MySQL 자료형
일반적으로 데이터베이스 서버는 문자열, 날짜 및 숫자 데이터를 저장할 수 있다.

그 외에는 XML, JSON 혹은 공간 데이터(spatial data)와 같은 특수 자료형을 다룰 수도 있다.


### 2.3.1 문자 데이터
#### 문자 데이터(character data)
- 고정 길이 문자열(fixed-length)
  - 공백으로 오른쪽이 채워지고, 항상 동일한 수의 바이트를 사용한다.
  - char 열의 최대 길이는 현재 255 바이트이다.
  - 열에 저장할 문자열이 약어처럼 길이가 동일할 때 사용된다.
    ``` sql
    char(20)
    ```

- 가변 길이 문자열(variable-length)
    - 공백으로 오른쪽이 채워지지 않고, 항상 동일한 수의 바이트를 사용하지 않는다.
    - varchar 열의 최대 길이는 최대 65,535바이트(64KB)까지 사용할 수 있다.
    - 열에 저장할 문자열
      ``` sql
      varchar(20)
      ```

#### 캐릭터셋(character set : 문자 집합)
라틴 알파벳을 사용하는 언어는 하나의 문자를 저장할 때, 1바이트만 필요하다. 반면, 다른 언어들은 많은 수의 문자를 포함하므로 여러 바이트의 저장 공간이 필요하다. 이러한 문자 집합, 즉 캐릭터셋을 **멀티 바이트 캐릭터셋(multibyte character set)** 이라고 한다.

- MySQL에서는 다양한 캐릭터셋을 모두 지원한다. 아래의 명령어로 지원되는 캐릭터셋을 확인할 수 있다.
  ``` sql
  mysql> SHOW CHARACTER SET;
  ```
- MySQL 8에서는 기본 캐릭터셋으로 utf8mb4을 사용한다.
- 열을 정의할 때 기본 캐릭터셋이 아닌 다른 캐릭터셋을 선택하려면 이래와 같이 지정할 수 있다.
  ``` sql
  varchar(20) character set latin1
  ```
- 아래의 명령을 통해 전체 데이터베이스에 대한 기본 캐릭터셋을 설정할 수 있다.
  ``` sql
  create database european_sales character set latin1;
  ```

#### 텍스트 데이터(text data)
varchar 열에 64KB 제한을 초과하는 데이터를 저장하기 위해서는 텍스트 자료형을 사용해야 한다.
- MySQL 텍스트 자료형

  | tinytext   | text          | mediumtext        | longtext             |
  |------------|---------------|-------------------|----------------------|
  | 255 byte   | 65,535 byte   | 16,777,215 byte   | 4,294,967,295 byte   |

- 텍스트 자료형을 선택할 때의 고려사항
  - 텍스트 열에 로드되는 데이터가 해당 유형의 최대 크기를 초과하면 데이터가 잘린다.
  - 데이터를 열에 로드하면, 후행 공백이 제거되지 않는다.
  - 정렬 또는 그룹화에 text 열을 사용할 경우, 필요하다면 한도를 늘릴 수 있지만, 처음에는 1,024byte만 사용된다.
  - text를 제외한 텍스트 자료형은 MySQL 고유의 자료형이다. DB2나 Oracle에서는 큰 문자 오브젝트에 clob 자료형을 사용한다.
  - MySQL 8은 varchar 열에 최대 65,535 바이트를 허용하므로, tinytext나 text 자료형을 사용할 필요가 없다.


### 2.3.2 숫자 데이터
#### 정수 자료형
- MySQL 정수 자료형

  | tinyint     | smallint         | mediumint              | int                            | bigint           |
  |-------------|------------------|------------------------|--------------------------------|------------------|
  | -128 ~ 127  | -32,768 ~ 32,767 | -8,388,608 ~ 8,388,607 | -2,147,483,648 ~ 2,147,483,647 | -2^63 ~ 2^63 - 1 |

  - 모든 정수 자료형은 `unsigned` 키워드를 앞에 붙여 적용시키는 것이 가능하다.


#### 부동소수점 자료형
- MySQL 부동소수점 자료형

  | float(p, s)           | double(p, s)           |
  |-----------------------|------------------------|
  | -3.40E+38 ~ -1.17E-38 | -1.79E+308 ~ 2.22E-308 |

        
  - 부동소수점 자료형을 사용할 때는 정밀도(precision)와 척도(scale)를 지정할 수 있지만, 필수는 아니다.
    - 정밀도 p : 소수점 왼쪽과 오른쪽 모두에 허용되는 자릿수
    - 척도 s : 소수점 오른쪽의 허용 자릿수

### 2.3.3 시간 데이터
#### MySQL 시간 자료형

| date | datetime | timestamp                    | year | time |
|------|----------|------------------------------|------|------|
|YYYY-MM-DD|YYYY-MM-DD HH:MI:SS| YYYY-MM-DD HH:MI:SS|YYYY|HH:MI:SS|


- datetime, timestamp, time 자료형에서는 SS에 대해 소수점 6자리까지 사용할 수 있다.
- datetime은 `1000-01-01 00:00:00 ~ 9999-12-31- 23:59:59` 의 범위를 지원하고, timestamp는 `1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07 UTC`를 지원한다.
- 각 데이터베이스 서버는 시간 자료형에 대해 서로 다른 날짜 범위를 허용한다. 현재 혹은 미래 날짜를 다룰 때에는 차이가 없겠지만, 300년 이상의 과거 날짜를 저장할 경우에는 반드시 주의해야 한다.

---

## 2.4 테이블 생성
### 2.4.1 단계 1 : 설계
테이블을 설계하기 전에는 어떤 종류의 정보를 포함할지 브레임 스토밍해보자.

예시로 아래의 항목을 정하여 person 테이블을 만들어보았다.

| 열              | 자료형          | 허용값        |
|----------------|--------------|------------|
|  name          | varchar(40)  |            |
| eye_color      | char(2)      | BL, BR, GR |
| birth_date     | date         |            |
| address        | varchar(100) |            |
| favorite_foods | varchar(200) |            |


### 2.4.2 단계 2 : 정제
정규화의 관점으로 person 테이블의 열을 다시 살펴보면 아래와 같은 문제점을 찾을 수 있다.
- 정규화란?
  - 데이터베이스 설계에 중복(외부 키 제외) 또는 복합 열이 없음을 확인하는 절차

- 문제
  - `name` 열은 이름과 성으로 구성된 복합 객체이다.
  - 여러 사람이 같은 이름, 눈동자, 생년월일 등을 가질 수 있으므로 고유성을 보장하는 열이 없다.
  - `address`는 거리, 도시, 주, 국가, 우편번호 등으로 구성된 복합 객체이다.
  - `favorite_foods`는 0개 혹은 1개 이상의 독립적인 항목을 포함하는 enumeration이다. 또한, 특정 음식이 어떤 사람에게 귀속되는지 알 수 있도록 별도의 테이블을 작성하여 person 테이블에 대한 FK를 포함하도록 하면 좋다.

해당 문제점을 고쳐 만든 테이블은 아래와 같다.

Person 테이블

| 열           | 자료형                | 허용값        |
|-------------|--------------------|------------|
| person_id   | smallint(unsigned) |            |
| first_name  | varchar(20)        |            |
| last_name   | varchar(20)        |            |
| eye_color   | char(2)            | BL, BR, GR |
| birth_date  | date               |            |
| city        | varchar(20)        |            |
| street      | varchar(20)        |            |
| state       | varchar(20)        |            |
| country     | varchar(20)        |            |
| postal_code | varchar(20)        |            |


Favorite_food 테이블

| 열         | 자료형                |
|-----------|--------------------|
| person_id | smallint(unsigned) |
| food      | varchar(20)        |

`person_id` 및 `food` 열은 favorite_food 테이블의 기본 키를 구성한다. 그러나 `person_id`는 person 테이블에 대한 외래 키이기도 하다.

더 깊게 자세하게 정규화를 거칠 수 있지만, 여기서는 이 정도로 마무리한다.

### 2.4.3 단계 3 : SQL 스키마 문 생성
위에서 만든 테이블 설계도를 통해 데이터베이스에 테이블을 만들자.
``` sql
CREATE TABLE person (
  person_id SMALLINT UNSIGNED,
  fname  VARCHAR(20),
  lname  VARCHAR(20),
  eye_color CHAR(2),
  birth_date  DATE,
  city  VARCHAR(20),
  street  VARCHAR(20),
  state VARCHAR(20),
  country VARCHAR(20),
  postal_code VARCHAR(20),
  CONSTRAINT pk_person PRIMARY KEY (person_id)
);
```

가장 밑의 `CONSTRAINT`는 제약조건으로 **기본 키 제약조건**으로 `person_id`를 지정하였다.

그 외에 **체크 체약조건**이 존재한다. 이는 특정 열에 대해 허용 가능한 값을 표시한다.

이를 사용하여 `eye_color` 허용 범위를 지정할 수 있다.
``` sql
eye_color char(2) CHECK (eye_color IN ('BR', 'BL', 'GR')),
```

혹은

``` sql
eye_color ENUM('BR', 'BL', 'GR'),
```
으로 가능하다.

- MySQL 서버는 체크 제약조건이 강제성을 띄지 않는다. 그러므로 `enum()`을 사용하여 그 외의 값을 지정하는 것을 막을할 수 있다.

``` sql
CREATE TABLE favorite_food (
  person_id SMALLINT UNSIGNED,
  food VARCHAR(20),
  CONSTRAINT pk_favorite_food PRIMARY KEY (person_id, food),
  CONSTRAINT fk_fav_food_person_id FOREIGN KEY (person_id) REFERENCES person (person_id)
);
```

favorite_food 테이블에는 **외래 키 제약조건**을 사용하였다. 그 덕분에 favorite_food 테이블의 `person_id` 열의 값에는 person 테이블에 있는 값만 포함하도록 제한된다.  

---

## 2.5 테이블 수정
### 2.5.1 데이터 삽입
#### 숫자 키 데이터 생성
person 테이블에 데이터를 삽입하기 전에, 숫자 기본 키의 값 생성 방법을 고려해보자. 
1. 표에서 현재 가장 큰 값을 확인하고 해당 값을 추가
2. 데이터베이스 서버가 값을 제공

첫 번째 방법은 race condition의 문제가 발생할 수 있다. 대신 MySQL의 **자동 증가**(auto-increment) 기능을 사용해보자.

``` sql
ALTER TABLE person MODIFY person_id SMALLINT UNSIGNED AUTO_INCREMENT;
```

#### INSERT 문
``` sql
INSERT INTO person
  (person_id, fname, lname, eye_color, birth_date)
  VALUES (null, 'William', 'Turner', 'BR', '1972-05-27');
```

``` sql
INSERT INTO person
  (person_id, fname, lname, eye_color, birth_date, street, city, state, country, postal_code)
  VALUES (null, 'Susan', 'Smith', 'BL', '1975-11-02', '23 Maple St.', 'Arlington', 'VA', 'USA', '20220');
```

유의사항
- `address` 열에는 값을 제공하지 않았지만, 해당 열은 null을 허용하므로 문제가 없다.
- `birth_date` 열에는 문자열이 제공되었지만, 필수 형식과 포멧이 같으면 문제가 없다.

``` sql
INSERT INTO favorite_food (person_id, food) VALUES (1, 'pizza');
INSERT INTO favorite_food (person_id, food) VALUES (1, 'cookies');
INSERT INTO favorite_food (person_id, food) VALUES (1, 'nachos');
```

### 2.5.2 데이터 수정
``` sql
UPDATE person
  SET street = '1225 Tremont St.',
      city = 'MA',
      country = 'USA',
      postal_code = '02138'
  WHERE person_id = 1;
```

### 2.5.3 데이터 삭제
``` sql
DELETE FROM person WHERE person_id = 2;
```

## 2.6 좋은 구문을 망치는 경우
### 2.6.1 고유하지 않은 기본 키
만약 중복된 값의 기본 키를 삽입한다면 문제가 된다.

그러나 person 테이블의 경우, 기본 키 제약조건과 자동 증가 기능을 켰기 때문에 괜찮다.

- 중복된 PK를 만들지 말아야 하는 이유
  - PK는 데이터의 중복을 방지한다.
  - 빠른 검색이 가능하게 한다.

### 2.6.2 존재하지 않는 외래 키
favorite_food 테이블은 외래 키 제약조건이 있기에 입력된 `person_id`의 모든 값이 person 테이블에 존재함을 보증한다.

그러나 그렇지 않은 경우, 존재하지 않는 외래 키를 삽입하면 에러가 발생한다.

이 경우, favorite_food 테이블은 일부 데이터가 person 테이블에 의존하므로, 하위 child로 간주되고, person 테이블은 상위 parent로 간주된다.  

따라서 상위인 person 테이블에 관련 행이 작성 되어야만 하위 테이블에 데이터를 삽입할 수 있다.

### 2.6.3 열 값 위반
person 테이블의 `eye_color` 열은 `enum()`으로 값을 제한하고 있다.

만약 해당 열에 잘못된 값을 삽입한다면 에러가 발생한다.

### 2.6.4 잘못된 날짜 변환
person 테이블의 `birth_date` 열은 DATE 형식을 사용한다.

만약 해당 열에 `DATE`와 일치하지 않는 날짜 형식을 사용하면 에러가 발생한다.

따라서 기본 형식에 의존하는 것보다는 형식 문자열을 명시적으로 지정하는 것이 좋다.

다음은 `str_to_date()`를 사용한 구문이다.
``` sql
UPDATE person
  SET birth_date = str_to_date('DEC-21-1980', '%b-%d-%Y')
  WHERE person_id = 1;
```
## 2.7 샤키라 데이터베이스
샤키라 샘플 데이터베이스는 DVD 대여점 체인을 설계한 것이다.

우리는 샤키라 스키마의 일부 테이블과 person, favorite_food 테이블을 이용하여 비디오 스트리밍 업체로 변경할 것이다.
