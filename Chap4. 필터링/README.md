# Chap 4. 필터링
아래와 같은 상황에서는 테이블의 모든 행에서 작업할 수 있다.
- 새 열이 추가된 후, 테이블의 모든 행 수정
- 메시지 큐 테이블에서 모든 행 검색

이런 상황에서는 제외할 행을 고려하지 않아도 되기에 `WHERE`절이 필요없다. 그러나 대부분의 경우에는 대상을 테이블의 부분 집합으로 좁히려 한다.

따라서 SQL 쿼리에서는 쿼리가 수행될 행 대상을 제한하기 위해 하나 이상의 **필터 조건**을 포함하는 `WHERE` 절이 선택적으로 포함된다.

또한, `SELECT` 문에는 그룹화된 데이터의 필터조건을 포함하는 `HAVING` 절이 포함된다.

### 문법
``` sql
SELECT select_list
FROM table_name
WHERE search_condition;
```

## 4.1 조건 평가
`WHERE` 절은 `AND` 혹은 `OR` 연산자로 구분된 하나 이상의 **조건**을 포함할 수 있다.

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

### 4.1.1 괄호 사용
`WHERE` 절에 `AND` 혹은 `OR` 연산자를 사용하여 조건이 세 개 이상 포함될 경우, 코드 가독성을 위해 괄호를 사용해야 한다.
``` sql
WHERE (fname = 'STEVEN' OR lname = 'YOUNG')
    AND create_date > '2006-01-01' 
```

### 4.1.2 `NOT` 연산자 사용
#### 연산자
| Equal | Not Equal |
|---|---|
| = | <> or != |

#### 예시
``` sql
SELECT lastname, firstname, jobtitle 
FROM employees
WHERE jobtitle <> 'Sales Rep';
```

## 4.2 조건 작성
아래 내용에서는 다양한 방법으로 단일 조건을 구성하는 방법에 대해 다룰 것이다.

조건은 하나 이상의 **연산자**와 결합된 하나 이상의 **표현식**으로 구성된다.

표현식은 다음 중 하나일 수 있다.
- 숫자
- 테이블 또는 뷰의 열
- 'Maple Street' 와 같은 문자열
- `CONCAT('Learning', ' ', 'SQL')`과 같은 내장 함수
- 서브쿼리
- `('Seoul', 'Incheon', 'Busan')` 과 같은 표현식 목록

연산자는 다음과 같다.
- `=`, `!=`, `<`, `>`, `<>`, `LIKE`, `IN`, `BETWEEN` 과 같은 비교 연산자

## 4.3 조건 유형
### 4.3.1 동등조건
### 4.3.2 범위조건
### 4.3.3 멤버십조건
### 4.3.3 일치조건

## 4.4 Null

## 4.5 학습 점검
