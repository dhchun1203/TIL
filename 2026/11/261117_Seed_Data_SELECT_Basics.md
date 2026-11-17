## 1. 오늘의 흐름

표를 다 만든 뒤, 연습용 데이터를 시드 파일로 한 번에 채우고 `SELECT`로 원하는 행과 열만 꺼냈다. 데이터가 수십 건은 있어야 조회가 의미 있다.

1. `enrollments`(N:M)와 `instructors`를 만들고 `ALTER TABLE`로 강좌에 강사 열 추가
2. 시드 파일을 컨테이너에 복사하고 `\i`로 실행
3. `SELECT`의 별칭·계산·`DISTINCT`와 `WHERE` 조건으로 조회

## 2. 구조

```txt
lockers.member_id      → members.member_id           1:1  (UNIQUE)
enrollments.member_id  → members.member_id           ┐ N:M (회원 ↔ 강좌)
enrollments.course_id  → courses.course_id           ┘
courses.sport_id       → sports.sport_id             1:N
instructors.sport_id   → sports.sport_id             1:N
courses.instructor_id  → instructors.instructor_id   1:N  (강사 없는 강좌는 NULL)
```

| 표 | 시드 후 행 수 | 담는 것 |
| --- | --- | --- |
| `members` | 60 | 회원 |
| `sports` | 4 | 수영, 헬스, 골프연습, 스포츠댄스 |
| `instructors` | 8 | 강사 |
| `courses` | 14 | 개설 강좌 |
| `lockers` | 18 | 사물함 배정 |
| `enrollments` | 200 | 수강 신청 기록 |

시드(seed) 파일은 표를 처음 상태로 다시 만들고 연습용 데이터를 채우는 SQL 파일이다.

## 3. 코드

### 1) 시드 데이터 넣기

```bash
docker compose cp sportscenter_seed.sql db:/tmp/sportscenter_seed.sql   # 내 PC 파일 → 컨테이너 /tmp
docker compose exec db psql -U postgres                                  # postgres DB로 접속
\i /tmp/sportscenter_seed.sql                                            # 파일 실행
```

시드 파일은 `DROP DATABASE sportscenter` → `CREATE DATABASE sportscenter` → `\c sportscenter` → 표 6개 생성 → `INSERT` 순서로 돌아간다.

### 2) `sportscenter_study.sql` (오늘 추가한 부분)

```sql
-- 수강 신청 테이블
CREATE TABLE enrollments
(
    member_id           INT                 NOT NULL REFERENCES members (member_id),
    course_id           INT                 NOT NULL REFERENCES courses (course_id),
    enroll_month        CHAR(7)             NOT NULL,
    status              VARCHAR(10)         NOT NULL,
    pay_amount          NUMERIC(10,2)       NOT NULL CHECK (pay_amount >= 0),
    PRIMARY KEY (member_id, course_id, enroll_month)
);

INSERT INTO enrollments
VALUES (1, 1, '2026-11', '수료', 50000);

INSERT INTO enrollments
VALUES (2, 2, '2026-10', '수료', 70000);

INSERT INTO enrollments
VALUES (2, 2, '2026-10', '수료', 90000);

-- 강사 테이블
DROP TABLE IF EXISTS instructors;
CREATE TABLE instructors
(
    instructor_id           SERIAL          PRIMARY KEY,
    name                    VARCHAR(20)     NOT NULL,
    sports_id               INT             NOT NULL REFERENCES sports (sports_id),
    phone                   VARCHAR(20),
    hire_date               DATE            NOT NULL
);

INSERT INTO instructors (name, sports_id, phone, hire_date)
VALUES 
('홍길동', 1, '010-1234-1234', '2025-01-01'),
('손흥민', 2, '010-1234-8080', '2025-01-10');


-- 테이블 수정 ALTER TABLE

ALTER TABLE courses
ADD COLUMN instructor_id INT REFERENCES instructors (instructor_id);

SELECT course_name AS 강좌명,
fee AS 수강료,
fee * 3 AS "3개월 수강료"
FROM courses
LIMIT 5;

SELECT DISTINCT region FROM members;

SELECT DISTINCT sport_id, level FROM courses;

SELECT * FROM members WHERE member_id <= 7;

SELECT * FROM members WHERE region = '마포구';

SELECT COUNT(*) AS "이메일 미기재" FROM members WHERE email IS NULL;

SELECT * FROM members WHERE birth_year BETWEEN 1995 AND 2000;

SELECT * FROM courses WHERE course_id IN (1, 5, 10);

SELECT * FROM members WHERE name LIKE '김%';
```

## 4. 핵심 정리

**Q. 시드 파일은 어떻게 넣지?**

- psql은 컨테이너 안에서 돈다. 내 PC 파일은 못 보니 `cp`로 컨테이너 `/tmp`에 넣고 `\i`로 실행한다.
- 시드는 `sportscenter`를 통째로 지웠다 다시 만든다. 그래서 `postgres` DB에 접속해서 실행한다.
- `-d sportscenter`로 들어가면 `cannot drop the currently open database`로 막힌다.

**Q. 회원과 강좌는 왜 중간 테이블로 잇지?**

- 회원 ↔ 강좌는 N:M이다. 한쪽에 FK를 두면 한 칸에 값이 여러 개 들어가야 한다.
- `enrollments` 한 행 = 누가, 어떤 강좌를, 몇 월에.
- 복합 PK는 세 값의 조합이 겹칠 때만 막는다.

**Q. `NUMERIC(10,2)`와 `CHECK`는?**

- `NUMERIC(10,2)`: 전체 10자리 중 소수 2자리. 정확해야 하는 금액에 쓴다.
- `CHECK (pay_amount >= 0)`: 조건에 안 맞는 값을 거절한다. 직접 넣어 보니 `-1000`은 막혔다.

**Q. 이미 있는 표는 어떻게 고치지?**

- `ALTER TABLE ... ADD COLUMN`: 데이터는 두고 열만 더한다. 기존 행의 새 열은 NULL이다.
- `DROP TABLE IF EXISTS`: 표가 없으면 오류 없이 넘어간다. 다른 표가 참조하는 표는 그래도 못 지운다.

**Q. 원하는 것만 꺼내려면?**

- `SELECT`는 열을, `WHERE`는 행을 고른다.
- `AS`로 결과 열 이름을 바꾼다. 띄어쓰기가 있으면 `"3개월 수강료"`처럼 큰따옴표.
- `DISTINCT`는 중복을 뺀다. 열이 여러 개면 조합 기준이다.

| 조건 | 뜻 | 예 |
| --- | --- | --- |
| `=`, `<=` | 같다, 이하 | `region = '마포구'` |
| `IS NULL` | 비어 있다 | `email IS NULL` |
| `BETWEEN a AND b` | a 이상 b 이하 (양 끝 포함) | `birth_year BETWEEN 1995 AND 2000` |
| `IN (...)` | 목록 중 하나 | `course_id IN (1, 5, 10)` |
| `LIKE '김%'` | 김으로 시작 | `name LIKE '김%'` |

NULL은 `=`로 비교하지 못한다. 직접 실행하니 `email = NULL`은 0건, `email IS NULL`은 15건이었다.

## 5. 오늘 만난 오류

| 메시지 | 원인 |
| --- | --- |
| `cannot drop the currently open database` | `sportscenter`에 접속한 채로 시드 실행 |
| `cannot drop table sports because other objects depend on it` | 다른 표가 참조하는 표를 삭제 |
| `duplicate key value violates unique constraint "enrollments_pkey"` | 같은 회원·강좌·월 조합을 두 번 입력 (복합 PK) |
| `new row for relation "enrollments" violates check constraint` | 음수 금액 (`CHECK`) |

## 6. 실행 예시

```sql
SELECT course_name AS 강좌명,
fee AS 수강료,
fee * 3 AS "3개월 수강료"
FROM courses
LIMIT 5;
```

```txt
      강좌명      | 수강료 | 3개월 수강료 
------------------+--------+--------------
 수영 초급 새벽반 |  70000 |       210000
 헬스 자유이용    |  50000 |       150000
 수영 중급 저녁반 |  75000 |       225000
 수영 상급 저녁반 |  80000 |       240000
 아쿠아로빅       |  65000 |       195000
(5 rows)
```

```sql
SELECT DISTINCT region FROM members;
```

```txt
  region  
----------
 은평구
 영등포구
 용산구
 마포구
 서대문구
(5 rows)
```

```sql
SELECT DISTINCT sport_id, level FROM courses;
```

```txt
 sport_id | level 
----------+-------
        3 | 초급
        1 | 상급
        2 | 자유
        2 | 중급
        1 | 초급
        3 | 자유
        4 | 초급
        3 | 중급
        2 | 초급
        4 | 중급
        1 | 중급
(11 rows)
```

```sql
SELECT * FROM members WHERE member_id <= 7;
```

```txt
 member_id |  name  | gender | birth_year |     phone     |       email        |  region  | join_date  
-----------+--------+--------+------------+---------------+--------------------+----------+------------
         1 | 김하늘 | F      |       1995 | 010-1234-5678 | haneul@example.com | 마포구   | 2026-09-01
         2 | 이준호 | M      |       1988 | 010-2345-6789 |                    | 서대문구 | 2026-09-03
         3 | 박서윤 | F      |       1972 | 010-3456-7890 | seoyun@example.com | 마포구   | 2025-03-15
         4 | 최민준 | M      |       2001 | 010-4567-8901 | minjun@example.com | 은평구   | 2026-08-20
         5 | 정유진 | F      |       1960 | 010-5678-9012 |                    | 서대문구 | 2024-11-02
         6 | 강도윤 | M      |       1983 |               | doyun@example.com  | 마포구   | 2025-06-10
         7 | 조수아 | F      |       1999 | 010-7890-1234 | sua@example.com    | 용산구   | 2026-10-05
(7 rows)
```

```sql
SELECT * FROM members WHERE region = '마포구';
```

```txt
 member_id |  name  | gender | birth_year |     phone     |        email         | region | join_date  
-----------+--------+--------+------------+---------------+----------------------+--------+------------
         1 | 김하늘 | F      |       1995 | 010-1234-5678 | haneul@example.com   | 마포구 | 2026-09-01
         3 | 박서윤 | F      |       1972 | 010-3456-7890 | seoyun@example.com   | 마포구 | 2025-03-15
         6 | 강도윤 | M      |       1983 |               | doyun@example.com    | 마포구 | 2025-06-10
        18 | 최준서 | M      |       1966 | 010-5544-2914 |                      | 마포구 | 2025-09-07
...
        60 | 오혜원 | F      |       1985 |               | member60@example.com | 마포구 | 2026-07-10
(17 rows)
```

```sql
SELECT COUNT(*) AS "이메일 미기재" FROM members WHERE email IS NULL;
```

```txt
 이메일 미기재 
---------------
            15
(1 row)
```

```sql
SELECT * FROM members WHERE birth_year BETWEEN 1995 AND 2000;
```

```txt
 member_id |  name  | gender | birth_year |     phone     |        email         |  region  | join_date  
-----------+--------+--------+------------+---------------+----------------------+----------+------------
         1 | 김하늘 | F      |       1995 | 010-1234-5678 | haneul@example.com   | 마포구   | 2026-09-01
         7 | 조수아 | F      |       1999 | 010-7890-1234 | sua@example.com      | 용산구   | 2026-10-05
        16 | 권유나 | F      |       1998 | 010-7429-1462 | member16@example.com | 은평구   | 2026-07-11
        35 | 서가은 | F      |       1997 | 010-5440-4706 |                      | 서대문구 | 2025-02-12
        43 | 조하은 | F      |       1997 | 010-8718-5847 | member43@example.com | 서대문구 | 2024-08-17
        45 | 오서현 | F      |       1997 | 010-5585-6746 | member45@example.com | 서대문구 | 2026-01-07
        53 | 권은지 | F      |       1998 | 010-4621-9593 | member53@example.com | 은평구   | 2024-12-05
(7 rows)
```

```sql
SELECT * FROM courses WHERE course_id IN (1, 5, 10);
```

```txt
 course_id | sport_id |    course_name     | level | class_days | start_time | capacity |  fee  | instructor_id 
-----------+----------+--------------------+-------+------------+------------+----------+-------+---------------
         1 |        1 | 수영 초급 새벽반   | 초급  | 월수금     | 06:00:00   |       20 | 70000 |             1
         5 |        1 | 아쿠아로빅         | 초급  | 화목       | 10:00:00   |       25 | 65000 |             1
        10 |        3 | 골프 타석 자유이용 | 자유  | 매일       | 07:00:00   |       30 | 90000 |              
(3 rows)
```

```sql
SELECT * FROM members WHERE name LIKE '김%';
```

```txt
 member_id |  name  | gender | birth_year |     phone     |        email         | region | join_date  
-----------+--------+--------+------------+---------------+----------------------+--------+------------
         1 | 김하늘 | F      |       1995 | 010-1234-5678 | haneul@example.com   | 마포구 | 2026-09-01
         9 | 김정우 | M      |       1981 | 010-7353-9848 | member09@example.com | 은평구 | 2025-04-15
(2 rows)
```
