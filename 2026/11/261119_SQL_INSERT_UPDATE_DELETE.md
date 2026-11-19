## 1. 오늘의 흐름

어제까지는 데이터를 꺼내 보기만 했고, 오늘은 데이터를 넣고 고치고 지웠다. 바꾼 결과는 `RETURNING`으로 바로 확인했다.

1. `INSERT ... RETURNING`, `INSERT ... SELECT`로 행 추가
2. `UPDATE ... SET`으로 값 변경 (서브쿼리로 대상 지정까지)
3. `DELETE`, `NOT EXISTS`, `TRUNCATE`로 행 삭제
4. 참고로 `RETURNING * `은 Postgres 에서만 지원하는 문법이니 주의하자

## 2. 구조

```txt
INSERT    → 행 추가        VALUES로 직접, 또는 SELECT 결과를 통째로
UPDATE    → 값 변경        SET 열 = 새 값, WHERE로 대상 지정
DELETE    → 행 삭제        WHERE로 대상 지정
TRUNCATE  → 모든 행 삭제   표는 남고 내용만 비움

… RETURNING 열  → 위 명령이 바꾼 행을 바로 보여줌
```

| 명령 | 대상 | `WHERE`를 빼면 |
| --- | --- | --- |
| `INSERT` | 새 행 | 해당 없음 |
| `UPDATE` | `WHERE`에 맞는 행 | 모든 행이 바뀐다 |
| `DELETE` | `WHERE`에 맞는 행 | 모든 행이 지워진다 |
| `TRUNCATE` | 표 전체 | `WHERE`를 쓸 수 없다 |

`UPDATE`와 `DELETE`는 같은 `WHERE`로 먼저 `SELECT` 해서 대상을 확인한 뒤 실행한다.

### 문법 사용법

```sql
-- 행 추가 + 추가된 행 바로 보기
INSERT INTO 표 (열1, 열2) VALUES (값1, 값2) RETURNING *;

-- 조회 결과를 통째로 넣기: 넣을 열 목록과 SELECT 열의 개수·순서를 맞춘다
INSERT INTO 표A (열1, 열2, 열3)
SELECT 열x, 열y, '고정값' FROM 표B WHERE 조건;

-- 타입 바꾸기
CAST('2026-11-19' AS DATE)

-- 값 변경: 여러 열은 쉼표, 기존 값을 이용한 계산도 된다
UPDATE 표 SET 열1 = 값, 열2 = 열2 + 5 WHERE 조건 RETURNING 열1, 열2;

-- 서브쿼리 결과를 대상으로
UPDATE 표 SET ... WHERE 키 IN (SELECT 키 FROM ... GROUP BY ... HAVING ...);

-- 행 삭제
DELETE FROM 표 WHERE 조건 RETURNING *;

-- 다른 표에 짝이 없는 행만 고르기
WHERE NOT EXISTS (SELECT 1 FROM 표B b WHERE b.키 = a.키)

-- 표 비우기 (번호까지 1부터 다시 쓰려면 RESTART IDENTITY)
TRUNCATE 표;
TRUNCATE 표 RESTART IDENTITY;
```

## 3. 코드

`Query.sql`. 시드 데이터가 들어 있는 `sportscenter`에서 실행한다.

```sql
INSERT INTO
    sports (sport_name)
VALUES
    ('필라테스') RETURNING *;

SELECT
    *
FROM
    sports;

CREATE TABLE
    contact_lists (
        request_id SERIAL PRIMARY KEY,
        member_id INT NOT NULL,
        name VARCHAR(20) NOT NULL,
        reason VARCHAR(50) NOT NULL,
        request_date DATE NOT NULL
    );

SELECT
    *
FROM
    contact_lists;

TRUNCATE contact_lists;

INSERT INTO
    contact_lists (name, member_id, reason, request_date)
SELECT
    name,
    member_id,
    '이메일 미등록' AS reason,
    CAST('2026-11-19' AS DATE) AS request_date
FROM
    members
WHERE
    email IS NULL
UNION
SELECT
    name,
    member_id,
    '휴대폰 미등록',
    CAST('2026-11-19' AS DATE)
FROM
    members
WHERE
    phone IS NULL RETURNING *;

SELECT
    *
FROM
    members
WHERE
    member_id = 2;

UPDATE members
SET
    email = 'ljh@abc.com'
WHERE
    member_id = 2 RETURNING *;

SELECT
    *
FROM
    courses;

UPDATE courses
SET
    class_days = '월수금',
    start_time = '20:00:00'
WHERE
    course_id = 9 RETURNING course_id,
    course_name,
    class_days,
    start_time;

SELECT
    *
FROM
    courses
WHERE
    sport_id = 1;

UPDATE courses
SET
    fee = fee * 1.05
WHERE
    sport_id = 1 RETURNING fee;

-- 서브쿼리 활용한 UPDATE
-- 2026-11 수강신청인원이 5명 이상인 경우 수강신청인원수 +5
UPDATE courses
SET
    capacity = capacity + 5
WHERE
    course_id in (
        SELECT
            course_id
        FROM
            enrollments
        WHERE
            enroll_month = '2026-11'
        group by
            course_id
        having
            COUNT(*) >= 5
    ) RETURNING course_id,
    course_name,
    capacity;

-- DELETE
SELECT
    *
FROM
    lockers;

DELETE FROM lockers
WHERE
    member_id = 38 RETURNING *;

-- DELETE FROM members
-- WHERE 
SELECT
    m.member_id,
    m.name,
    m.join_date
FROM
    members m
WHERE
    NOT EXISTS (
        SELECT
            1
        FROM
            enrollments e
        WHERE
            e.member_id = m.member_id
    );

DELETE FROM members m
WHERE
    NOT EXISTS (
        SELECT
            1
        FROM
            enrollments e
        WHERE
            e.member_id = m.member_id
    ) RETURNING m.member_id,
    m.name;

-- DELETE
-- TRUNCATE
-- DROP TABLE
```

## 4. 핵심 정리

**Q. `RETURNING`은 왜 쓰지?**

- 바꾼 행을 `SELECT`를 다시 치지 않고 바로 본다.
- `INSERT`에 붙이면 자동으로 매겨진 `SERIAL` 번호를 알 수 있다. 필라테스는 `sport_id` 5를 받았다.
- `*` 대신 열 이름을 적으면 필요한 열만 나온다.

**Q. `INSERT ... SELECT`는?**

- `VALUES` 대신 조회 결과를 통째로 넣는다. 넣을 열 목록과 **`SELECT` 열의 개수·순서가 맞아야 한다.**
- `'이메일 미등록'`처럼 고정값도 열 하나로 넣을 수 있다. 날짜는 `CAST('2026-11-19' AS DATE)`로 타입을 맞춘다.
- `UNION`으로 두 조건의 결과를 합쳐 넣었다. 이메일 미등록 15명 + 휴대폰 미등록 3명 = 18행.
- `request_id`는 `SERIAL`이라 열 목록에서 뺀다.

**Q. `UPDATE`는?**

- `SET 열 = 값`으로 바꾼다. 여러 열은 쉼표로 잇는다.
- `fee = fee * 1.05`, `capacity = capacity + 5`처럼 기존 값으로 계산할 수 있다.
- `WHERE`가 바꿀 행을 정한다. 빼면 표의 모든 행이 바뀐다.

**Q. 서브쿼리로 대상을 고르려면?**

- `WHERE course_id IN (서브쿼리)`: 서브쿼리가 돌려준 번호 목록이 대상이 된다. 11월 신청이 5건 이상인 강좌 6개의 정원이 늘었다.
- `NOT EXISTS (서브쿼리)`: 바깥 행마다 안쪽에 짝이 있는지 본다. 안쪽이 한 행도 없으면 참이다.
- `SELECT 1`은 값이 아니라 "있느냐"만 보니 아무 값이나 쓴다.
- 수강 기록이 없는 회원 6명을 `SELECT`로 먼저 확인하고, 같은 조건으로 `DELETE` 했다. 수강 기록이 있는 회원은 `enrollments`가 참조하고 있어서 지울 수 없다.

**Q. `DELETE`와 `TRUNCATE`는?**

- `DELETE`는 `WHERE`로 골라 지운다. `TRUNCATE`는 표의 모든 행을 비운다. 둘 다 표 자체는 남는다.
- `TRUNCATE`를 해도 `SERIAL` 번호는 되돌아가지 않는다. 직접 해보니 18행을 넣고 비운 뒤 다시 넣으면 19번이 나왔고, **`TRUNCATE ... RESTART IDENTITY`로 비우면 1번부터 다시 나왔다**.

## 5. 실행 예시

```sql
INSERT INTO sports (sport_name) VALUES ('필라테스') RETURNING *;
```

```txt
 sport_id | sport_name 
----------+------------
        5 | 필라테스
(1 row)

INSERT 0 1
```

```sql
SELECT * FROM sports;
```

```txt
 sport_id | sport_name 
----------+------------
        1 | 수영
        2 | 헬스
        3 | 골프연습
        4 | 스포츠댄스
        5 | 필라테스
(5 rows)
```

```sql
CREATE TABLE contact_lists (...);
SELECT * FROM contact_lists;
TRUNCATE contact_lists;
```

```txt
CREATE TABLE
 request_id | member_id | name | reason | request_date 
------------+-----------+------+--------+--------------
(0 rows)

TRUNCATE TABLE
```

```sql
INSERT INTO contact_lists (name, member_id, reason, request_date)
SELECT name, member_id, '이메일 미등록' AS reason, CAST('2026-11-19' AS DATE) AS request_date
FROM members WHERE email IS NULL
UNION
SELECT name, member_id, '휴대폰 미등록', CAST('2026-11-19' AS DATE)
FROM members WHERE phone IS NULL
RETURNING *;
```

```txt
 request_id | member_id |  name  |    reason     | request_date 
------------+-----------+--------+---------------+--------------
          1 |        42 | 강나연 | 이메일 미등록 | 2026-11-19
          2 |         6 | 강도윤 | 휴대폰 미등록 | 2026-11-19
          3 |        27 | 권우진 | 이메일 미등록 | 2026-11-19
...
         17 |        15 | 황은지 | 이메일 미등록 | 2026-11-19
         18 |        58 | 황은찬 | 이메일 미등록 | 2026-11-19
(18 rows)

INSERT 0 18
```

```sql
SELECT * FROM members WHERE member_id = 2;
```

```txt
 member_id |  name  | gender | birth_year |     phone     | email |  region  | join_date  
-----------+--------+--------+------------+---------------+-------+----------+------------
         2 | 이준호 | M      |       1988 | 010-2345-6789 |       | 서대문구 | 2026-09-03
(1 row)
```

```sql
UPDATE members SET email = 'ljh@abc.com' WHERE member_id = 2 RETURNING *;
```

```txt
 member_id |  name  | gender | birth_year |     phone     |    email    |  region  | join_date  
-----------+--------+--------+------------+---------------+-------------+----------+------------
         2 | 이준호 | M      |       1988 | 010-2345-6789 | ljh@abc.com | 서대문구 | 2026-09-03
(1 row)

UPDATE 1
```

```sql
SELECT * FROM courses;
```

```txt
 course_id | sport_id |    course_name     | level | class_days | start_time | capacity |  fee   | instructor_id 
-----------+----------+--------------------+-------+------------+------------+----------+--------+---------------
         1 |        1 | 수영 초급 새벽반   | 초급  | 월수금     | 06:00:00   |       20 |  70000 |             1
...
         9 |        3 | 골프 중급반        | 중급  | 화목       | 19:00:00   |       10 | 130000 |             6
...
        14 |        1 | 수영 마스터즈반    | 상급  | 토         | 07:00:00   |       10 |  90000 |             1
(14 rows)
```

```sql
UPDATE courses
SET class_days = '월수금', start_time = '20:00:00'
WHERE course_id = 9
RETURNING course_id, course_name, class_days, start_time;
```

```txt
 course_id | course_name | class_days | start_time 
-----------+-------------+------------+------------
         9 | 골프 중급반 | 월수금     | 20:00:00
(1 row)

UPDATE 1
```

```sql
SELECT * FROM courses WHERE sport_id = 1;
```

```txt
 course_id | sport_id |   course_name    | level | class_days | start_time | capacity |  fee  | instructor_id 
-----------+----------+------------------+-------+------------+------------+----------+-------+---------------
         1 |        1 | 수영 초급 새벽반 | 초급  | 월수금     | 06:00:00   |       20 | 70000 |             1
         3 |        1 | 수영 중급 저녁반 | 중급  | 월수금     | 19:00:00   |       20 | 75000 |             3
         4 |        1 | 수영 상급 저녁반 | 상급  | 화목       | 20:00:00   |       15 | 80000 |             3
         5 |        1 | 아쿠아로빅       | 초급  | 화목       | 10:00:00   |       25 | 65000 |             1
        14 |        1 | 수영 마스터즈반  | 상급  | 토         | 07:00:00   |       10 | 90000 |             1
(5 rows)
```

```sql
UPDATE courses SET fee = fee * 1.05 WHERE sport_id = 1 RETURNING fee;
```

```txt
  fee  
-------
 73500
 78750
 84000
 68250
 94500
(5 rows)

UPDATE 5
```

```sql
UPDATE courses
SET capacity = capacity + 5
WHERE course_id IN (
    SELECT course_id FROM enrollments
    WHERE enroll_month = '2026-11'
    GROUP BY course_id
    HAVING COUNT(*) >= 5
)
RETURNING course_id, course_name, capacity;
```

```txt
 course_id |   course_name    | capacity 
-----------+------------------+----------
         2 | 헬스 자유이용    |       55
         3 | 수영 중급 저녁반 |       25
         4 | 수영 상급 저녁반 |       20
         8 | 골프 입문반      |       15
        11 | 라틴댄스 초급    |       25
        13 | 실버 스포츠댄스  |       25
(6 rows)

UPDATE 6
```

```sql
SELECT * FROM lockers;
```

```txt
 locker_no | member_id | assign_date 
-----------+-----------+-------------
       101 |         1 | 2026-09-01
       102 |         3 | 2025-03-15
...
       110 |        38 | 2025-08-01
...
       118 |        60 | 2026-07-10
(18 rows)
```

```sql
DELETE FROM lockers WHERE member_id = 38 RETURNING *;
```

```txt
 locker_no | member_id | assign_date 
-----------+-----------+-------------
       110 |        38 | 2025-08-01
(1 row)

DELETE 1
```

```sql
SELECT m.member_id, m.name, m.join_date
FROM members m
WHERE NOT EXISTS (
    SELECT 1 FROM enrollments e WHERE e.member_id = m.member_id
);
```

```txt
 member_id |  name  | join_date  
-----------+--------+------------
        47 | 박민재 | 2026-07-05
        13 | 서승현 | 2026-07-12
        55 | 서성민 | 2026-02-27
        48 | 홍정우 | 2026-08-10
        23 | 류민재 | 2024-03-19
        58 | 황은찬 | 2024-04-26
(6 rows)
```

```sql
DELETE FROM members m
WHERE NOT EXISTS (
    SELECT 1 FROM enrollments e WHERE e.member_id = m.member_id
)
RETURNING m.member_id, m.name;
```

```txt
 member_id |  name  
-----------+--------
        47 | 박민재
        13 | 서승현
        55 | 서성민
        48 | 홍정우
        23 | 류민재
        58 | 황은찬
(6 rows)

DELETE 6
```

## 6. 챌린지: Python으로 수강신청

`psycopg`로 `sportscenter` DB에 접속해서 터미널에서 수강신청을 받는 프로그램이다. 회원 이름으로 회원 ID를 찾고, 강좌명으로 검색해 고른 뒤, `enrollments`에 신청을 넣는다.

```txt
main() → list_courses(신청 가능 목록) → ask_member(이름 → member_id)
       → ask_course(LIKE 검색 → course_id) → enroll(INSERT + 결과 확인)
```

### 핵심

- **접속**: `psycopg.connect(DSN, row_factory=dict_row)`. `dict_row`로 받으면 행을 `row['member_id']`처럼 열 이름으로 꺼낸다.
- **값 넣기**: SQL 안에 `%s` 자리표시자를 두고 값은 튜플로 따로 넘긴다(`(m_name,)`). 문자열로 직접 이어 붙이지 않는다. 따옴표 처리를 드라이버가 맡아서 안전하다.
- **와일드카드**: `%`는 SQL이 아니라 값 쪽에 붙인다. `like %s` + `f"%{c_name}%"`.
- **결과 받기**: `fetchone()`은 한 행 또는 `None`, `fetchall()`은 행 목록이다.
- **수강료**: `INSERT`의 `VALUES` 안에 서브쿼리 `(SELECT fee FROM courses WHERE course_id = %s)`를 넣어, 수강료 조회와 신청을 한 문장으로 끝냈다.
- **저장**: `with psycopg.connect(...) as conn:` 블록이 오류 없이 끝나면 변경이 commit된다.

### 제출한 `main.py`

```py
import psycopg
from psycopg.rows import dict_row
# uv add "psycopg[binary]"

DSN = "host=localhost port=5432 dbname=sportscenter user=postgres password=admin"
MONTH = "2026-11"
LIST_SQL = """
select
    distinct c.course_id,
    s.sport_name,
    c.course_name,
    c.class_days,
    c.fee
from
    courses c
    join sports s ON (s.sport_id = c.sport_id)
    left join enrollments e on c.course_id = e.course_id
    and e.enroll_month = %s
order by
    c.course_id

"""
# 데이터베이스 일일 과제 

# 1. 회원이름으로 회원ID 추출
# 회원 이름을 입력받아서 회원 ID 를 추출해서 MEMBER_ID에 저장하세요.
# 회원 ID를 추출하는 기능은 함수로 작성하세요.

# 2. 신청한 강좌의 수강료 추출
# 신청한 강좌의 수강료를 추출해서 fee 변수에 할당(연결)해주세요.
# 중복신청은 불가능하게 먼저 확인하세요. (도전 과제)

# 3. 강좌 검색 기능
# 강좌명을 입력받아 비슷한 강좌를 검색하는 기능을 구현하세요.
# 와일드카드를 활용하셔야 합니다.


# 수강 신청가능 목록 조회
def list_courses(conn):
    for c in conn.execute(LIST_SQL, (MONTH,)):
        print(f"{c['course_id']:>3} | {c['sport_name']} | {c['course_name']} | {c['class_days']} | {c['fee']:,}원")
    print()


# TODO: 입력 로직 구현
def ask_member(conn):
    try:
        m_name = input("회원이름을 입력해주세요.: ")
        
        m_id = conn.execute(
            "select member_id from members where name = %s"
        , (m_name,)).fetchone()
        
        MEMBER_ID = m_id['member_id']
        return MEMBER_ID
        
    except TypeError:
        print("등록되지 않은 회원입니다.")
        ask_member(conn)
    
def ask_course(conn):
    try:
        c_name = input("강좌 이름을 입력해주세요.: ")
        
        c_ids = conn.execute(
            "select course_id, course_name from courses where course_name like %s order by course_id"
        , (f"%{c_name}%",)).fetchall()
        
        print("강좌검색목록=============================")
        
        no_ids = 1
        for id in c_ids:
            print(f"{no_ids}   {id['course_name']}")
            no_ids += 1
            
        selected_no = int(input("번호를 선택해주세요. : "))
        
        while selected_no not in range(1, len(c_ids) + 1):
            print('목록에 있는 번호를 입력해주세요.')
            selected_no = int(input("번호를 선택해주세요. : "))
            
        COURSE_ID = int(c_ids[selected_no - 1]['course_id'])
        return COURSE_ID
        
    except TypeError:
        print("조회결과가 없습니다.")
        return
    
    
# 수강신청
def enroll(conn, member_id, course_id):
    
    conn.execute(
        "INSERT INTO enrollments (member_id, course_id, enroll_month, status, pay_amount) VALUES (%s, %s, %s, '수강중', (SELECT fee FROM courses WHERE course_id = %s))",
        (member_id, course_id, MONTH, course_id)
    )
    print("수강신청 완료되었습니다.")
    
    print("신청결과 +++++++++++++++++++++++++++++++")
    
    
    
    print("회원이름\t강좌아이디\t등록월\t상태\t지불금액")
    
    result = conn.execute(
                "SELECT member_id, course_id, enroll_month, status, pay_amount FROM enrollments WHERE member_id = %s AND course_id = %s",
                (member_id, course_id)
            ).fetchone()
    
    if result:
        print(f"{result['member_id']}\t\t{result['course_id']}\t\t{result['enroll_month']}\t\t{result['status']}\t\t{result['pay_amount']}")
    else:
        print("작업에 문제가 발생했습니다.")
    

def main():
    with psycopg.connect(DSN, row_factory=dict_row) as conn:
        
        # 수강목록
        list_courses(conn)
        
        MEMBER_ID = ask_member(conn)
        COURSE_ID = ask_course(conn)
        
        # 수강신청
        enroll(conn, MEMBER_ID, COURSE_ID)

if __name__ == "__main__":
    main()

```

### 직접 실행해 본 결과

| 입력 | 결과 |
| --- | --- |
| `김하늘` → `골프` → `1` | 신청 완료: `1 / 8 / 2026-11 / 수강중 / 120000.00` |
| 같은 신청을 한 번 더 | `UniqueViolation: ... "enrollments_pkey"`로 프로그램이 멈춤 |
| 없는 이름 → 맞는 이름 | `NotNullViolation: null value in column "member_id"`로 멈춤 |
| 검색 결과가 없는 강좌명(`요가`) | "목록에 있는 번호를 입력해주세요."를 계속 반복 |
| 번호 대신 글자 입력 | `ValueError: invalid literal for int()`로 멈춤 |
