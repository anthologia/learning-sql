# Chap 3. 쿼리 입문
## 3.1 쿼리 역학
사용자가 Database connection을 얻으면, DB 서버(이하 서버)는 쿼리 실행 준비를 한다.

이후, 서버는 쿼리를 실행하기 전 아래 사항을 확인한다.
- 이 구문을 실행할 권한이 있는가?
- 원하는 데이터에 액세스할 수 있는 권한이 있는가?
- 구문의 문법이 정확한가?

이 세 가지 단계를 통과하면, 쿼리는 **Query optimizer**로 전달된다.
- Query optimizer가 하는 일은 무엇일까 ❓
  1. FROM 절에 명명된 테이블에 조인할 순서 및 사용 가능한 인덱스를 확인한다.
  2. 서버가 쿼리 실행에 필요한 **실행 계획** execution plan을 선택한다.

서버가 쿼리 실행을 마치면 결과셋을 반환한다.

## 3.2 쿼리 절
`SELECT` 문은 여러 개의 구성요소 component 및 절 clause로 구성된다.

쿼리 절 예시
- `SELECT`
  - 쿼리 결과에 포함될 열을 결정한다
- `FROM`
  - 데이터를 검색할 테이블과 테이블을 조인하는 방법을 식별한다
- `WHERE`
  - 불필요한 데이터를 걸러낸다
- `GROUP BY`
  - 공통 열 값을 기준으로 행을 그룹화한다
- `HAVING`
  - 불필요한 그룹을 걸러낸다
- `ORDER BY`
  - 하나 이상의 열을 기준으로 최종 결과의 행을 정렬한다

위의 모든 절은 ANSI specification에 포함된다.
- ANSI specification❓ ANSI SQL❓
  - 다양한 DBMS에서 서로 다른 SQL 문법을 가지므로, 미국 표준 협회(ANSI)에서 이를 정립시킨 표준 SQL문을 의미한다.

## 3.3 SELECT 절
`SELECT` 절은 `SELECT` 문의 첫 번째 절이지만, DB 서버가 판단하는 마지막 절 중 하나이다.

최종 결과셋에 포함될 항목을 결정하기 위해서는 최종 결과셋에 포함될 모든 열을 먼저 알아야 하기 때문이다.

쿼리 결과에 포함될 열을 결정하는 것 외에도 아래와 같은 항목을 `SELECT` 절에 추가할 수 있다.
- 숫자 또는 문자열과 같은 리터럴
- transaction.amount * - 1 과 같은 표현식
- ROUND(transaction.amount, 2)와 같은 내장 함수(built-in function) 호출
- 사용자 정의 함수(user-defined function) 호출

예시
``` sql
SELECT language_id,
       'COMMON' language_usage,
       language_id * 3.1415927 lang_pi_value,
       upper(name) language_name
FROM language; 
```

- 내장함수만 실행하거나 간단한 표현식을 사용하는 경우에는 FROM 절을 건너뛰는 것도 가능하다.

### 3.3.1 열의 별칭 Column alias
결과셋의 열에 새 레이블을 할당하고 싶거나 이름이 모호한 경우, 열 별칭을 추가할 수 있다.

열 별칭을 두드러지게 하려면 `AS`를 사용하면 된다.

#### 문법
``` sql
SELECT 표현식 AS 열_별명;
SELECT 표현식 열_별명;
```

#### 예시
``` sql
SELECT CONCAT('John', 'Doe') AS name;
```

``` sql
SELECT CONCAT_WS(', ', lastname, firstname) 'Full name'
FROM employees
ORDER BY 'Full name';
```

``` sql
SELECT orderNumber 'Order no.', SUM(priceEach * quantityOrdered) total
FROM orderDetails
GROUP BY 'Order no.'
HAVING total > 60000;
```

### 3.3.2 중복 제거 Distinct
경우에 따라 쿼리가 중복된 데이터 행을 반환할 수 있다.

만약 고유한 데이터 행만을 보고 싶다면, `DISTINCT` 키워드를 사용하여 확인할 수 있다.

#### 문법
``` sql
SELECT DISTINCT select_list
FROM table_name
WHERE search_condition
ORDER BY sort_expression;
```

#### 예시
``` sql
SELECT DISTINCT state, city
FROM customer
WHERE state IS NOT NULL
ORDER BY state, city;
```

만약 서버가 중복 데이터를 제거하는 것을 원치 않거나, 결과에 중복이 없는 게 확실할 때는 `ALL` 키워드를 지정할 수 있다.

그러나 `ALL`은 기본 값이므로 명시적으로 포함할 필요가 없다.

- 성능 향상❗ :️ `DISTINCT` 결과를 생성하려면 데이터를 정렬해야 하므로 결과셋의 용량이 클 때는 시간이 오래 걸릴 수 있다. 따라서 무분별한 `DISTINCT` 사용 보다는 작업 중인 데이터를 먼저 이해하고 중복 여부를 판별하는 것이 좋다.

## 3.4 FROM 절
여기서는 `FROM` 절을 단순히 하나 이상의 테이블 목록으로 정의하는 것이 아닌 아래의 정의로 확장할 것이다.

> `FROM` 절은 쿼리에 사용되는 테이블을 명시할 뿐만 아니라, 테이블을 서로 연결하는 수단도 함께 정의한다.

### 3.4.1 테이블 유형
**_테이블_** 이라는 용어를 생각하면, 대부분 DB에 저장된 일련의 관련 행 집합을 떠올린다. 즉, 이는 한 가지 유형의 테이블이다.

여기서는 데이터가 어떻게 저장되는지에 대한 개념 대신, **관련 행들의 집합**에 집중하여 테이블이라는 용어를 더 일반적인 방식으로 사용할 것이다.

| 영구(permanent) 테이블     | 파생(derived) 테이블         | 임시(temporary) 테이블 | 가상(virtual) 테이블  |
|-----------------------|-------------------------|------------------|------------------|
| `CREATE TABLE` 문으로 생성 | 하위 쿼리에서 반환하고 메모리에 보관된 행 | 메모리에 저장된 휘발성 데이터 | `CREATE VIEW` 문으로 생성 |


FROM에 영구 테이블을 포함하는 것은 익숙하므로 나머지 테이블에 대해 간략하게 다뤄보자. 

#### 파생 테이블 Derived table
서브 쿼리(sub query)는 다른 쿼리에 포함된 쿼리이다. 괄호로 묶여 있으며, `SELECT` 문의 여러 부분에서 찾을 수 있다.

그러나 FROM 절 내에서의 서브 쿼리는 FROM 절에 명시된 다른 테이블과 상호작용할 수 있는 파생 테이블을 생성하는 역할을 한다.

``` sql
SELECT CONCAT(cust.lname, ', ', cust.fname) full_name
FROM (
  SELECT fname, lname, email
  FROM customer
  WHERE fname = 'JESSIE'
) cust;
```

서브 쿼리는 별칭을 통해 참조되는데 위 구문의 경우에는 cust라고 지정하였다.  cust의 데이터는 쿼리 기간동안 메모리에 보관된 후 삭제된다.

#### 임시 테이블 Temporary table
모든 관계형 DB는 휘발성의 임시 테이블을 정의할 수 있다.

이런 테이블은 영구 테이블처럼 보이지만, 보통 트랙잭션이 끝날 때 혹은 데이터베이스 세션이 닫힐 때 사라진다.

``` sql
CREATE TEMPORARY TABLE actors_j (
  actor_id SMALLINT(5),
  fname VARCHAR(45),
  lname VARCHAR(45),
);
```

#### 가상 테이블(뷰) Virtual table(View)
뷰는 데이터 딕셔너리에 저장된 쿼리이다.

마치 테이블처럼 동작하지만 뷰에 저장된 데이터가 존재하지는 않는다. 이 때문에 가상 테이블이라고 불린다.

뷰에 대한 쿼리를 실행하면, 쿼리가 뷰 정의와 합쳐져 실행할 최종 쿼리를 만든다.

``` sql
CREATE VIEW cust_vw AS
  SELECT customer_id, first_name, last_name, active
  FROM customer;

SELECT first_name, last_name
FROM cust_vw
WHERE active = 0;
```

뷰를 작성하더라도 데이터가 추가 생성되거나 저장되지는 않는다.

사용자로부터 열을 숨기고 복잡한 데이터베이스 설계를 단순화하는 등의 이유로 뷰가 만들어진다.

### 3.4.2 테이블 연결
`FROM` 절에 둘 이상의 테이블이 있으면, 그 테이블을 연결하는데 필요한 조건도 포함해야 한다.

``` sql
SELECT customer.fname, customer.lname, time(rental.rental_date) retal_time
FROM customer
  INNER JOIN rental ON customer.customer_id = rental.customer_id
WHERE date(rental.rental_date) = '2005-06-14';
```

`JOIN` 과 관련된 자세한 내용은 5장에서 다룬다.

### 3.4.3 테이블 별칭 정의
단일 쿼리에서 여러 테이블을 조인할 경우, 참조 테이블을 식별할 방법이 필요하다.

- employee.emp_id 와 같이 전체 테이블 이름을 사용
- 각 테이블의 별칭을 할당하고 쿼리 전체에서 해당 별칭을 사용

아래의 쿼리는 테이블 별칭을 사용한 쿼리이다. 열 별칭과 마찬가지로 `AS` 키워드를 사용하거나 생략할 수 있다.

``` sql
SELECT c.fname, c.lname, time(r.rental_date) retal_time
FROM customer c
  INNER JOIN rental r ON c.customer_id = r.customer_id
WHERE date(r.rental_date) = '2005-06-14';
```

## 3.5 WHERE 절
> `WHERE` 절은 결과셋에 출력되기를 원하지 않는 행을 필터링하는 메커니즘이다.
### 문법
``` sql
SELECT 조회_목록
FROM 테이블_이름
WHERE 검색_조건;
```

### 예시
``` sql
SELECT lastname, firstname, jobtitle, officeCode 
FROM employees
WHERE jobtitle = 'Sales Rep' AND officeCode = 1;
```

``` sql
SELECT lastname, firstname, jobtitle, officeCode 
FROM employees
WHERE jobtitle = 'Sales Rep' OR officeCode = 1
ORDER BY officeCode, jobTitle;
```

``` sql
SELECT lastname, firstname, jobtitle, officeCode 
FROM employees
WHERE officeCode BETWEEN 1 AND 3
ORDER BY officeCode;
```

``` sql
SELECT lastname, firstname, officeCode 
FROM employees
WHERE officeCode IN (1, 2, 3)
ORDER BY officeCode;
```

``` sql
SELECT lastname, firstname, officeCode 
FROM employees
WHERE officeCode IS NULL;
```

## 3.6 GROUP BY와 HAVING 절
때로는 데이터에서 결과를 검색하기 전에 DB 서버가 데이터를 정제하는 흐름을 찾아볼 수 있다.

이 메커니즘 중 하나는 데이터를 열 값 별로 그룹화하는 `GROUP BY` 절이다.

``` sql
SELECT c.fname, c.lname, count(*)
FROM customer c
  INNER JOIN rental r ON c.customer_id = r.customer_id
GROUP BY c.fname, c.lname HAVING count(*) >= 40;
```

잘 이해가 가지 않아도 괜찮다. 8장에서 자세히 다루기 때문이다.

## 3.7 ORDER BY 절
> `ORDER BY` 절은 원시 열 데이터 또는 열 데이터를 기반으로 표현식을 사용하여 결과셋을 장렬하는 메커니즘이다.

### 3.7.1 내림차순 및 오름차순 정렬
### 문법
``` sql
SELECT 조회_목록
FROM 테이블_이름
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC];
```

### 예시
``` sql
SELECT contactLastName, contactFirstName
FROM customer
ORDER BY contactLastName;
```

``` sql
SELECT orderNumber, orderLineNumber, quantityOrdered * priceEach 
FROM orderdetials
ORDER BY quantityOrdered * priceEach DESC;
```

``` sql
SELECT orderNumber, orderLineNumber, quantityOrdered * priceEach AS subtotal 
FROM orderdetials
ORDER BY subtotal DECS;
```

### 3.7.2 순서를 통한 정렬
### 문법
``` sql
SELECT 조회_목록
FROM 테이블_이름
ORDER BY 열_번호 [ASC|DESC];
```

### 예시
``` sql
SELECT orderNumber, orderLineNumber, quantityOrdered * priceEach 
FROM orderdetials
ORDER BY 3 DESC;
```

이 방법은 `ORDER BY` 절의 숫자를 변경하지 않고, `SELECT` 절에 열을 추가하면 예기치 않은 결과가 발생할 수 있다.

그러므로 순서를 통한 정렬의 사용은 자제하자.

## 3.8 학습 점검
### 3.8.1 실습 3-1
모든 배우의 배우 ID, 이름 및 성을 검색하라. 성 기준으로 정렬한 다음 이름 기준으로 정렬한다.
``` sql
SELECT actor_id, first_name, last_name
FROM actor
ORDER BY first_name, last_name;
```


### 3.8.1 실습 3-2
성이 `WILIAMS` 또는 `DAVIS`인 모든 배우의 배우 ID, 이름 및 성을 검색한다.
``` sql
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name = 'WILIAMS' OR last_name = 'DAVIS';
```

### 3.8.1 실습 3-3
rental 테이블에서 2005년 7월 5일 영화를 대여한 고객의 ID를 반환하는 쿼리를 작성하라. (rental.rental_date 열을 사용하고, date() 함수로 시간 요소를 무시할 수 있음.) 각 고객 ID는 하나의 행을 포함한다.
``` sql
SELECT customer_id
FROM rental
WHERE date(rental_date) = '2005-07-05';

-- 고급 버전
SELECT c.customer_id
FROM customer AS c
INNER JOIN rental AS r USING(customer_id)
WHERE date(r.rental_date) = '2005-07-05';
```

### 3.8.1 실습 3-4
다중 테이블 쿼리의 <#> 부분을 채우세요.

``` sql
SELECT c.email, r.return_date
FROM customer AS c
INNER JOIN rental AS r USING(customer_id)
WHERE date(r.rental_date) = '2005-06-14'
ORDER BY r.return_date DESC;
```
