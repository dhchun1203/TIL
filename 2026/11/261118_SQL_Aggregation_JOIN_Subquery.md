## 1. 오늘의 흐름

표 하나에서 행을 고르는 데서 나아가, 행을 묶어 세고 여러 표를 이어서 조회했다. 데이터에서 "월별 매출", "강사 없는 강좌" 같은 답을 꺼내는 단계다.

1. `AND`/`OR`, `CASE`, `ORDER BY`로 행을 고르고 꾸미고 정렬
2. 집계 함수와 `GROUP BY`·`HAVING`으로 묶어서 세기
3. `UNION`, `JOIN`, 서브쿼리로 여러 결과와 표를 합치기

## 2. 구조

```txt
SELECT 문의 실행 순서

FROM / JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
 (표 준비)   (행 거름)  (묶음)     (그룹 거름) (열 고름)  (정렬)     (개수 자름)
```

| 하고 싶은 것 | 문법 |
| --- | --- |
| 행 거르기 | `WHERE` (`AND`, `OR`, `IS NOT NULL`, `<>`) |
| 값 새로 만들기 | `CASE WHEN ... THEN ... END` |
| 정렬 | `ORDER BY 열1 DESC, 열2` |
| 여러 행을 한 값으로(집계) | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` |
| 그룹별로 집계 / 그룹 거르기 | `GROUP BY` / `HAVING` |
| 결과를 위아래로 쌓기 | `UNION`, `UNION ALL` |
| 표를 옆으로 잇기 | `LEFT` / `RIGHT` / `FULL OUTER JOIN` |
| 쿼리 안의 쿼리 | 서브쿼리 (`WHERE`, `FROM`, `SELECT` 자리) |

`WHERE`가 `SELECT`보다 먼저라서 별칭을 못 쓰고, `ORDER BY`는 `SELECT` 다음이라 별칭(`누적신청`)을 쓸 수 있다.

### 문법 사용법

```sql
-- 행 거르기: AND가 OR보다 먼저 → 묶을 땐 괄호
SELECT 열 FROM 표 WHERE 조건1 AND (조건2 OR 조건3);
-- 다르다: 열 <> 값      비어 있지 않다: 열 IS NOT NULL

-- 값 새로 만들기: 위에서부터 처음 맞는 조건의 값
CASE WHEN 조건1 THEN 값1
     WHEN 조건2 THEN 값2
     ELSE 값3
END AS 새이름

-- 정렬: 기본은 오름차순, DESC는 내림차순. 앞 열이 같으면 다음 열로
ORDER BY 열1 DESC, 열2

-- 집계: 여러 행 → 값 하나
SELECT COUNT(*), COUNT(열), COUNT(DISTINCT 열), SUM(열), AVG(열), MIN(열), MAX(열) FROM 표;

-- 그룹별 집계 + 그룹 거르기
SELECT 묶을열, COUNT(*)
FROM 표
WHERE 행조건              -- 묶기 전
GROUP BY 묶을열
HAVING COUNT(*) >= 10;    -- 묶은 뒤

-- 위아래로 쌓기: 위아래 열 개수를 맞춘다
SELECT 열1, 열2 FROM 표A
UNION ALL                 -- UNION이면 중복 제거
SELECT 열1, 열2 FROM 표B;

-- 옆으로 잇기: RIGHT JOIN, FULL OUTER JOIN도 같은 모양
SELECT a.열, b.열
FROM 표A a
LEFT JOIN 표B b ON a.키 = b.키;

-- 서브쿼리 세 자리
WHERE 열 > (SELECT AVG(열) FROM 표)                -- WHERE 자리: 값과 비교
FROM (SELECT ... GROUP BY ...) AS 이름             -- FROM 자리: 결과를 표처럼
SELECT 열, (SELECT AVG(열) FROM 표) AS 전체평균     -- SELECT 자리: 모든 행에 같은 값
```

## 3. 코드

`sportscenter_study.sql` (오늘 추가한 부분). 시드 데이터가 들어 있는 `sportscenter`에서 실행한다.

```sql
SELECT course_name, level, start_time
FROM courses
WHERE sport_id = 1
AND (level = '초급' OR start_time < '09:00');

SELECT name, birth_year, 
        CASE WHEN birth_year <= 1960 THEN '할인'
             WHEN birth_year <= 2000 THEN '청소년'
             ELSE '일반'
        END AS 요금구분
FROM members
ORDER BY birth_year, name
LIMIT 10;

-- 수강료가 비싼 순서대로 정렬
SELECT course_name, fee
FROM courses
WHERE fee >= 80000
ORDER BY  fee DESC, course_name
LIMIT 5;

SELECT name, email 
FROM members
WHERE email IS NOT NULL
ORDER BY email DESC, name
LIMIT 10;

-- 집계함수
SELECT 
    COUNT(*)            AS 강좌수,
    MIN(fee)            AS 최저수강료,
    MAX(fee)            AS 최고수강료,
    SUM(capacity)       AS 전체정원,
    ROUND(AVG(fee), 2)  AS 평균수강료
FROM courses;

-- COUNT(*) VS COUNT(컬럼)

SELECT 
    COUNT(*)                AS 회원수,
    COUNT(email)            AS 이메일있음,
    COUNT(phone)            AS 전화번호있음,
    COUNT(DISTINCT region)  AS 거주지
FROM members;

-- 최고 매출 SUM

SELECT 
    COUNT(*)                AS 신청건수,
    SUM(pay_amount)         AS 매출,
    MAX(pay_amount)         AS 최고매출
FROM   enrollments
WHERE enroll_month = '2026-11';

SELECT
    enroll_month            AS 신청월,
    COUNT(*)                AS 신청건수,
    SUM(pay_amount)         AS 매출
FROM enrollments
GROUP BY enroll_month
ORDER BY enroll_month;


SELECT 
    enroll_month, status, COUNT(*) AS 건수
FROM enrollments
GROUP BY enroll_month, status
ORDER BY enroll_month, status;

-- HAVING

SELECT 
    course_id, COUNT(*) AS 누적신청
FROM enrollments
WHERE status <> '취소'
GROUP BY course_id
HAVING COUNT(*) >= 18
ORDER BY 누적신청 DESC, course_id;

-- 거주지, 해당거주지회원수
-- 조건 : 해당 거주지 회원수 10명 이상인 경우

SELECT region, COUNT(*) AS 회원수
FROM members
GROUP BY region
HAVING COUNT(*) >= 10
ORDER BY 회원수 DESC;

-- UNION(중복제거) / UNION ALL(그대로 쌓아 올린다)

SELECT member_id FROM enrollments WHERE course_id = 4 AND enroll_month = '2026-09'
UNION ALL
SELECT member_id FROM enrollments WHERE course_id = 4 AND enroll_month = '2026-09'
UNION ALL
SELECT member_id FROM enrollments WHERE course_id = 4 AND enroll_month = '2026-09'
ORDER BY member_id;

SELECT '강사' AS 구분, name, phone FROM instructors WHERE sport_id = 4
UNION ALL
SELECT '회원' AS 구분, name, phone FROM members WHERE region = '영등포구';


-- LEFT JOIN: returns all rows from the left table (table1), and only the matched rows from the right table (table2)

SELECT 
    e.member_id,
    c.course_name,
    m.name,
    e.enroll_month,
    e.status
FROM enrollments e
LEFT JOIN members m ON e.member_id = m.member_id
LEFT JOIN courses c ON e.course_id = c.course_id
WHERE e.course_id = 1 AND e.enroll_month = '2026-11'
ORDER BY m.name;

-- 
SELECT 
    c.course_id,
    c.course_name,
    i.name AS 강사
FROM courses c
LEFT JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY c.course_id;

SELECT 
    c.course_id,
    c.course_name,
    i.name AS 강사
FROM courses c
RIGHT JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY c.course_id;

SELECT 
    c.course_id,
    c.course_name,
    i.name AS 강사
FROM courses c
FULL OUTER JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY c.course_id;

-- 서브 쿼리 IN()

SELECT AVG(fee) FROM courses;

SELECT course_name, fee
FROM courses
WHERE fee > (SELECT AVG(fee) FROM courses)
ORDER BY fee DESC, course_name;


-- FROM Sub Query
-- Inline View
SELECT paid.member_id, m.name, paid.total_payment
FROM (
    SELECT member_id, SUM(pay_amount) AS total_payment
    FROM enrollments
    GROUP BY member_id
) AS paid
LEFT JOIN members m ON paid.member_id = m.member_id
WHERE paid.total_payment >= 400000
ORDER BY paid.total_payment DESC, paid.member_id;

SELECT 
    course_name,
    fee,
    (SELECT ROUND(AVG(fee),2) FROM courses) AS 전체평균,
    fee - ROUND((SELECT AVG(fee) FROM courses), 2) AS 평균과차이
FROM courses;
```

## 4. 핵심 정리

**Q. 행을 고르고, 꾸미고, 정렬하려면?**

- `WHERE`는 조건에 맞는 행만 남긴다.
- 조건은 `AND`(모두 만족), `OR`(하나라도 만족)로 잇는다. 둘을 섞을 때는 `AND`가 먼저 계산되니 괄호로 묶는다.
- `<>`는 "다르다", `IS NOT NULL`은 "비어 있지 않다"다.
- `CASE WHEN 조건 THEN 값 ... ELSE 값 END`: 위에서부터 처음 맞는 조건의 값을 준다.
- `ORDER BY fee DESC, course_name`: 수강료 내림차순, 같으면 강좌명 오름차순.

**Q. 집계 함수와 `COUNT(*)` vs `COUNT(열)`은?**

- 집계 함수는 여러 행을 값 하나로 줄인다.
- `COUNT(*)`는 행 수, `COUNT(열)`은 NULL을 뺀 수, `COUNT(DISTINCT 열)`은 중복을 뺀 수다.
- 회원 60명 중 이메일이 있는 사람은 45명, 거주지는 5곳이었다.
- `AVG`는 소수가 길게 나오니 `ROUND(값, 2)`로 자른다.

**Q. `GROUP BY`와 `HAVING`은?**

- `GROUP BY`는 같은 값끼리 묶어 그룹별로 집계한다.
- `SELECT`에는 `GROUP BY`한 열이나 집계 함수만 올 수 있다.
- `WHERE`는 묶기 전의 행을, `HAVING`은 묶은 뒤의 그룹을 거른다. 그래서 집계 조건(`COUNT(*) >= 10`)은 `HAVING`에 쓴다.

**Q. 여러 결과나 표를 합치려면?**

- `UNION`은 결과를 위아래로 쌓는다. 위아래 열 개수가 같아야 한다.
- `UNION ALL`은 그대로, `UNION`은 중복을 빼고 쌓는다. 같은 쿼리 3번을 쌓으니 `UNION ALL`은 12행, `UNION`은 4행이었다.
- `JOIN`은 표를 옆으로 잇는다. `ON`에 어느 열끼리 이을지 적고, `courses c`처럼 표에 짧은 별칭을 붙여 `c.course_name`으로 쓴다.

| JOIN (`courses` ←→ `instructors`) | 남는 행 | 결과 |
| --- | --- | --- |
| `JOIN` | 양쪽에 짝이 있는 행만 | 12행 |
| `LEFT JOIN` | 왼쪽(강좌) 전부, 강사 없는 강좌는 NULL | 14행 |
| `RIGHT JOIN` | 오른쪽(강사) 전부, 강좌 없는 강사는 NULL | 13행 |
| `FULL OUTER JOIN` | 양쪽 전부 | 15행 |

**Q. 서브쿼리는 어디에 쓰지?**

- 쿼리 안의 쿼리다. 안쪽이 먼저 실행돼서 그 결과를 바깥이 쓴다.
- `WHERE` 자리: "평균보다 비싼"처럼 미리 계산한 값과 비교한다.
- `FROM` 자리(인라인 뷰): 집계한 결과를 표처럼 쓴다. `AS paid`처럼 이름을 붙이고 JOIN할 수 있다.
- `SELECT` 자리: 모든 행에 같은 값(전체 평균)을 붙인다.

## 5. 오늘 만난 오류

| 메시지 | 원인 |
| --- | --- |
| `aggregate functions are not allowed in WHERE` | 집계 조건을 `WHERE`에 씀 → `HAVING`으로 |
| `column "members.name" must appear in the GROUP BY clause or be used in an aggregate function` | 묶지 않은 열을 `SELECT`에 넣음 |

## 6. 실행 예시

```sql
SELECT course_name, level, start_time
FROM courses
WHERE sport_id = 1
AND (level = '초급' OR start_time < '09:00');
```

```txt
   course_name    | level | start_time 
------------------+-------+------------
 수영 초급 새벽반 | 초급  | 06:00:00
 아쿠아로빅       | 초급  | 10:00:00
 수영 마스터즈반  | 상급  | 07:00:00
(3 rows)
```

```sql
SELECT name, birth_year, 
        CASE WHEN birth_year <= 1960 THEN '할인'
             WHEN birth_year <= 2000 THEN '청소년'
             ELSE '일반'
        END AS 요금구분
FROM members
ORDER BY birth_year, name
LIMIT 10;
```

```txt
  name  | birth_year | 요금구분 
--------+------------+----------
 권서현 |       1955 | 할인
 황다은 |       1955 | 할인
 권준서 |       1958 | 할인
...
 정유진 |       1960 | 할인
 신지아 |       1962 | 청소년
 황은지 |       1962 | 청소년
(10 rows)
```

```sql
SELECT course_name, fee
FROM courses
WHERE fee >= 80000
ORDER BY  fee DESC, course_name
LIMIT 5;
```

```txt
    course_name     |  fee   
--------------------+--------
 헬스 PT 그룹반     | 150000
 골프 중급반        | 130000
 골프 입문반        | 120000
 골프 타석 자유이용 |  90000
 수영 마스터즈반    |  90000
(5 rows)
```

```sql
SELECT name, email 
FROM members
WHERE email IS NOT NULL
ORDER BY email DESC, name
LIMIT 10;
```

```txt
  name  |        email         
--------+----------------------
 조수아 | sua@example.com
 박서윤 | seoyun@example.com
 최민준 | minjun@example.com
 오혜원 | member60@example.com
...
 권은지 | member53@example.com
(10 rows)
```

```sql
SELECT 
    COUNT(*)            AS 강좌수,
    MIN(fee)            AS 최저수강료,
    MAX(fee)            AS 최고수강료,
    SUM(capacity)       AS 전체정원,
    ROUND(AVG(fee), 2)  AS 평균수강료
FROM courses;
```

```txt
 강좌수 | 최저수강료 | 최고수강료 | 전체정원 | 평균수강료 
--------+------------+------------+----------+------------
     14 |      40000 |     150000 |      266 |   81785.71
(1 row)
```

```sql
SELECT 
    COUNT(*)                AS 회원수,
    COUNT(email)            AS 이메일있음,
    COUNT(phone)            AS 전화번호있음,
    COUNT(DISTINCT region)  AS 거주지
FROM members;
```

```txt
 회원수 | 이메일있음 | 전화번호있음 | 거주지 
--------+------------+--------------+--------
     60 |         45 |           57 |      5
(1 row)
```

```sql
SELECT 
    COUNT(*)                AS 신청건수,
    SUM(pay_amount)         AS 매출,
    MAX(pay_amount)         AS 최고매출
FROM   enrollments
WHERE enroll_month = '2026-11';
```

```txt
 신청건수 |    매출    | 최고매출  
----------+------------+-----------
       56 | 4286500.00 | 150000.00
(1 row)
```

```sql
SELECT
    enroll_month            AS 신청월,
    COUNT(*)                AS 신청건수,
    SUM(pay_amount)         AS 매출
FROM enrollments
GROUP BY enroll_month
ORDER BY enroll_month;
```

```txt
 신청월  | 신청건수 |    매출    
---------+----------+------------
 2026-08 |       46 | 3123500.00
 2026-09 |       48 | 3117000.00
 2026-10 |       50 | 3376500.00
 2026-11 |       56 | 4286500.00
(4 rows)
```

```sql
SELECT 
    enroll_month, status, COUNT(*) AS 건수
FROM enrollments
GROUP BY enroll_month, status
ORDER BY enroll_month, status;
```

```txt
 enroll_month | status | 건수 
--------------+--------+------
 2026-08      | 수료   |   42
 2026-08      | 취소   |    4
 2026-09      | 수료   |   43
 2026-09      | 취소   |    5
 2026-10      | 수료   |   47
 2026-10      | 취소   |    3
 2026-11      | 수강중 |   56
(7 rows)
```

```sql
SELECT 
    course_id, COUNT(*) AS 누적신청
FROM enrollments
WHERE status <> '취소'
GROUP BY course_id
HAVING COUNT(*) >= 18
ORDER BY 누적신청 DESC, course_id;
```

```txt
 course_id | 누적신청 
-----------+----------
        13 |       23
         8 |       21
        11 |       18
(3 rows)
```

```sql
SELECT region, COUNT(*) AS 회원수
FROM members
GROUP BY region
HAVING COUNT(*) >= 10
ORDER BY 회원수 DESC;
```

```txt
  region  | 회원수 
----------+--------
 서대문구 |     22
 마포구   |     17
 은평구   |     10
(3 rows)
```

```sql
SELECT member_id FROM enrollments WHERE course_id = 4 AND enroll_month = '2026-09'
UNION ALL
SELECT member_id FROM enrollments WHERE course_id = 4 AND enroll_month = '2026-09'
UNION ALL
SELECT member_id FROM enrollments WHERE course_id = 4 AND enroll_month = '2026-09'
ORDER BY member_id;
```

```txt
 member_id 
-----------
        10
        10
        10
        11
...
        46
(12 rows)
```

```sql
SELECT '강사' AS 구분, name, phone FROM instructors WHERE sport_id = 4
UNION ALL
SELECT '회원' AS 구분, name, phone FROM members WHERE region = '영등포구';
```

```txt
 구분 |  name  |     phone     
------+--------+---------------
 강사 | 배수민 | 010-3101-2207
 강사 | 차예린 | 
 회원 | 홍혜원 | 010-4895-6934
 회원 | 정서현 | 010-2479-2442
 회원 | 오예린 | 010-8384-9593
 회원 | 장은지 | 010-7121-3363
(6 rows)
```

```sql
SELECT 
    e.member_id,
    c.course_name,
    m.name,
    e.enroll_month,
    e.status
FROM enrollments e
LEFT JOIN members m ON e.member_id = m.member_id
LEFT JOIN courses c ON e.course_id = c.course_id
WHERE e.course_id = 1 AND e.enroll_month = '2026-11'
ORDER BY m.name;
```

```txt
 member_id |   course_name    |  name  | enroll_month | status 
-----------+------------------+--------+--------------+--------
         1 | 수영 초급 새벽반 | 김하늘 | 2026-11      | 수강중
        34 | 수영 초급 새벽반 | 송우진 | 2026-11      | 수강중
         7 | 수영 초급 새벽반 | 조수아 | 2026-11      | 수강중
(3 rows)
```

```sql
SELECT 
    c.course_id,
    c.course_name,
    i.name AS 강사
FROM courses c
LEFT JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY c.course_id;
```

```txt
 course_id |    course_name     |  강사  
-----------+--------------------+--------
         1 | 수영 초급 새벽반   | 한지수
         2 | 헬스 자유이용      | 
         3 | 수영 중급 저녁반   | 오민재
...
        10 | 골프 타석 자유이용 | 
...
        14 | 수영 마스터즈반    | 한지수
(14 rows)
```

```sql
SELECT 
    c.course_id,
    c.course_name,
    i.name AS 강사
FROM courses c
RIGHT JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY c.course_id;
```

```txt
 course_id |   course_name    |  강사  
-----------+------------------+--------
         1 | 수영 초급 새벽반 | 한지수
         3 | 수영 중급 저녁반 | 오민재
...
        14 | 수영 마스터즈반  | 한지수
           |                  | 차예린
(13 rows)
```

```sql
SELECT 
    c.course_id,
    c.course_name,
    i.name AS 강사
FROM courses c
FULL OUTER JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY c.course_id;
```

```txt
 course_id |    course_name     |  강사  
-----------+--------------------+--------
         1 | 수영 초급 새벽반   | 한지수
         2 | 헬스 자유이용      | 
...
        10 | 골프 타석 자유이용 | 
...
        14 | 수영 마스터즈반    | 한지수
           |                    | 차예린
(15 rows)
```

```sql
SELECT AVG(fee) FROM courses;
```

```txt
        avg         
--------------------
 81785.714285714286
(1 row)
```

```sql
SELECT course_name, fee
FROM courses
WHERE fee > (SELECT AVG(fee) FROM courses)
ORDER BY fee DESC, course_name;
```

```txt
    course_name     |  fee   
--------------------+--------
 헬스 PT 그룹반     | 150000
 골프 중급반        | 130000
 골프 입문반        | 120000
 골프 타석 자유이용 |  90000
 수영 마스터즈반    |  90000
(5 rows)
```

```sql
SELECT paid.member_id, m.name, paid.total_payment
FROM (
    SELECT member_id, SUM(pay_amount) AS total_payment
    FROM enrollments
    GROUP BY member_id
) AS paid
LEFT JOIN members m ON paid.member_id = m.member_id
WHERE paid.total_payment >= 400000
ORDER BY paid.total_payment DESC, paid.member_id;
```

```txt
 member_id |  name  | total_payment 
-----------+--------+---------------
        21 | 신우진 |     670000.00
        42 | 강나연 |     630000.00
         9 | 김정우 |     480000.00
        28 | 조동현 |     480000.00
        54 | 조지아 |     480000.00
        11 | 류은찬 |     470000.00
        50 | 류우진 |     450000.00
         8 | 윤동현 |     445000.00
(8 rows)
```

```sql
SELECT 
    course_name,
    fee,
    (SELECT ROUND(AVG(fee),2) FROM courses) AS 전체평균,
    fee - ROUND((SELECT AVG(fee) FROM courses), 2) AS 평균과차이
FROM courses;
```

```txt
    course_name     |  fee   | 전체평균 | 평균과차이 
--------------------+--------+----------+------------
 수영 초급 새벽반   |  70000 | 81785.71 |  -11785.71
 헬스 자유이용      |  50000 | 81785.71 |  -31785.71
 수영 중급 저녁반   |  75000 | 81785.71 |   -6785.71
...
 수영 마스터즈반    |  90000 | 81785.71 |    8214.29
(14 rows)
```
