## 1. 오늘의 흐름

생활체육센터 회원관리 DB를 만들었다. 잘못된 데이터가 애초에 들어오지 못하게 하는 건 프로그램이 아니라 DB의 제약 조건이다.

1. compose로 PostgreSQL 18을 띄우고 `sportscenter` DB를 만들어 접속
2. `members`, `sports`, `courses`를 만들며 PK와 FK 확인
3. `lockers`로 1:1 관계와 `ON DELETE CASCADE` 확인

## 2. 구조

```txt
lockers.member_id   → members.member_id   1:1  (UNIQUE + ON DELETE CASCADE)
courses.sports_id   → sports.sports_id    1:N
```

| 표 | 기본 키 | 가리키는 열 | 비워도 되는 열 |
| --- | --- | --- | --- |
| `members` | `member_id` (`SERIAL`) | 없음 | `phone`, `email`, `region` |
| `sports` | `sports_id` (`SERIAL`) | 없음 | 없음 |
| `courses` | `course_id` (`SERIAL`) | `sports_id` → `sports` | 없음 |
| `lockers` | `locker_no` (직접 입력) | `member_id` → `members` | 없음 |

표 구조를 정하는 `CREATE`는 DDL, 행을 다루는 `INSERT`·`SELECT`·`DELETE`는 DML이다.

## 3. 코드

### 1) DB 서버 띄우고 들어가기

`docker-compose.yaml`

```yaml
# 생활체육 회원관리 실습용 PostgreSQL
name: sportscenter

services:
  db:
    container_name: sportscenter-db
    image: postgres:18
    environment:
      POSTGRES_PASSWORD: admin
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql

volumes:
  pgdata:
```

```bash
docker compose up -d
docker compose exec db psql -U postgres                  # 1. 기본 DB로 들어가 CREATE DATABASE sportscenter;
docker compose exec db psql -U postgres -d sportscenter  # 2. 다음부터는 바로 sportscenter로
```

psql 명령: `\l` DB 목록, `\c 이름` DB 전환, `\dt` 테이블 목록, `\d+ 테이블` 열과 규칙, `\q` 종료.

### 2) `sportscenter_study.sql`

```sql
-- 데이터베이스 생성 (Data Definition Language)
CREATE DATABASE sportscenter;

CREATE TABLE members
(
    member_id       SERIAL          PRIMARY KEY,
    name            VARCHAR(30)     NOT NULL,
    gender          CHAR(1)         NOT NULL,
    birth_year      SMALLINT        NOT NULL,
    phone           VARCHAR(20),
    email           VARCHAR(50),
    region          VARCHAR(10),
    join_date       DATE            NOT NULL

);

-- 회원 등록 
-- INSERT INTO members (컬럼1, 컬럼2, 컬럼3) VALUES(컬럼1값, 컬럼2값, 컬럼3값);

-- INSERT INTO members VALUES (모든컬럼 ,,,,,);
-- INSERT INTO members SELECT * FROM WHERE ~~~

-- INSERT INTO members (컬럼1, 컬럼2, 컬럼3) 
-- VALUES
-- (컬럼1값, 컬럼2값, 컬럼3값),
-- (컬럼1값, 컬럼2값, 컬럼3값),
-- (컬럼1값, 컬럼2값, 컬럼3값),
-- (컬럼1값, 컬럼2값, 컬럼3값)


-- Data Manipulate Language
INSERT INTO members 
(
    name, 
    gender, 
    birth_year, 
    phone, 
    email, 
    region, 
    join_date
)
VALUES
(
    '김하늘', 
    'F', 
    1995, 
    '010-1234-5678', 
    'haneul@example.com', 
    '마포구', 
    '2026-09-01'
);

INSERT INTO members 
(
    name, 
    gender, 
    birth_year, 
    phone, 
    email, 
    region, 
    join_date
)
VALUES 
(
    '이준호', 
    'M', 
    1988, 
    '010-2345-6789', 
    NULL, 
    '서대문구', 
    '2026-09-03'
);

-- 종목 개설강좌 목록
CREATE TABLE sports
(
    sports_id           SERIAL          PRIMARY KEY,
    sports_name         VARCHAR(20)     NOT NULL
);

INSERT INTO sports (sports_name) VALUES ('수영');
INSERT INTO sports (sports_name) VALUES ('헬스');

-- 개설 강좌
CREATE TABLE courses
(
    course_id           SERIAL          PRIMARY KEY,
    sports_id           INT             NOT NULL REFERENCES sports (sports_id), 
    course_name         VARCHAR(30)     NOT NULL,
    level               VARCHAR(10)     NOT NULL,
    class_days          VARCHAR(10)     NOT NULL,
    start_time          TIME            NOT NULL,
    capacity            SMALLINT        NOT NULL,
    fee                 INT             NOT NULL 
);

INSERT INTO courses (sports_id, course_name, level, class_days, start_time, capacity, fee)
VALUES (1, '수영 초급 새벽반', '초급', '월수금', '06:00', 20, 70000);
INSERT INTO courses (sports_id, course_name, level, class_days, start_time, capacity, fee)
VALUES (2, '헬스 자유이용', '자유', '매일', '06:00', 50, 50000);

-- INSERT INTO courses (sports_id, course_name, level, class_days, start_time, capacity, fee)
-- VALUES (30, '헬스 자유이용', '자유', '매일', '06:00', 50, 50000);


CREATE TABLE lockers
(
    locker_no           INT         PRIMARY KEY,
    member_id           INT         NOT NULL UNIQUE REFERENCES members (member_id) ON DELETE CASCADE,
    -- ON DELETE CASCADE -> 참조하던 키가 지워지면 같이 지우겠다.
    asign_date          DATE        NOT NULL
);

INSERT INTO lockers VALUES (101, 1, '2026-11-01');
INSERT INTO lockers VALUES (102, 1, '2026-11-01');
INSERT INTO members 
(
    name, 
    gender, 
    birth_year, 
    join_date
)
VALUES 
(
    '임시회원', 
    'M', 
    2000, 
    '2026-11-01'
);

SELECT member_id, name FROM members;
INSERT INTO lockers VALUES (102, 3, '2026-11-01');
```

## 4. 핵심 정리

**Q. 왜 DB를 직접 만들고 `-d`로 들어가야 하지?**

`POSTGRES_DB`를 안 주었으니 `sportscenter`는 없다. 표는 지금 접속한 DB에 만들어지니, 먼저 `CREATE DATABASE`로 만든 뒤 `-d sportscenter`로 들어가야 한다. compose에서는 컨테이너 이름이 아니라 서비스 이름(`db`)을 쓴다.

**Q. 열 타입과 NULL 허용은 어떻게 정하지?**

담길 값의 모양에 맞춘다. 성별은 딱 1글자라 `CHAR(1)`, 출생연도는 작은 정수라 `SMALLINT`다. 전화번호는 계산할 값이 아니고 앞의 0이 사라지면 안 되니 문자열이다. 없을 수도 있는 `phone`, `email`, `region`만 `NOT NULL`을 뺐다.

**Q. PK와 `SERIAL`은?**

PK는 행 하나를 콕 집어야 하니 중복과 NULL을 막는다. `SERIAL`은 정수 열에 "자동 번호표에서 다음 번호를 꺼내라"는 기본값을 붙인 것이라 번호가 되돌아가지 않는다. 임시회원(3번)을 지워도 다음 회원은 4번이다.

**Q. `REFERENCES`(외래 키)는 뭘 막지?**

"가리키는 값이 저 표에 실제로 있어야 한다"는 약속이다. 그래서 없는 종목 번호로 강좌를 넣는 것도, 강좌가 가리키는 종목을 지우는 것도 막는다.

**Q. 1:1과 `ON DELETE CASCADE`는?**

FK에 `UNIQUE`를 더하면 같은 회원 번호가 한 번만 나올 수 있으니 1:1이 된다. `UNIQUE`가 없으면 1:N이다. `ON DELETE CASCADE`를 붙이면 회원이 지워질 때 그 회원의 사물함 행도 같이 지워진다. 같은 FK라도 "지울 때 막을지, 같이 지울지"를 표마다 정한다.

## 5. 오늘 만난 오류

| 메시지 | 원인 |
| --- | --- |
| `duplicate key value violates unique constraint "lockers_member_id_key"` | 회원 1에게 사물함을 두 번 배정 (`UNIQUE`) |
| `insert or update on table "courses" violates foreign key constraint` | 없는 종목 번호(30)로 강좌 등록 (`REFERENCES`) |
| `update or delete on table "sports" violates foreign key constraint` | 강좌가 있는 종목을 삭제 (`REFERENCES`) |
| `null value in column "join_date" ... violates not-null constraint` | 필수 열을 빼고 넣음 (`NOT NULL`) |
| `value too long for type character(1)` | `CHAR(1)`에 두 글자 |
| `column "total" does not exist` | `SELECT` 별칭을 `WHERE`에서 씀 (`WHERE`가 먼저 실행된다) |

메시지의 제약 이름을 보면 어느 표의 어느 열에서 막혔는지 알 수 있고, `DETAIL:` 줄에 어떤 값 때문인지 나온다.

## 6. 실행 예시

파일을 끝까지 실행하면 사물함 중복 배정 한 줄만 실패한다.

```txt
ERROR:  duplicate key value violates unique constraint "lockers_member_id_key"
DETAIL:  Key (member_id)=(1) already exists.
INSERT 0 1
 member_id |   name   
-----------+----------
         1 | 김하늘
         2 | 이준호
         3 | 임시회원
(3 rows)

INSERT 0 1
```

임시회원을 지우면 102번 사물함도 같이 사라진다.

```sql
DELETE FROM members WHERE member_id = 3;
SELECT * FROM lockers;
```

```txt
 locker_no | member_id | asign_date 
-----------+-----------+------------
       101 |         1 | 2026-11-01
(1 row)
```

## 7. 과제: 상품 DB 설계

상품, 카테고리, 장바구니 표 세 개를 설계하고 PK, FK, 제약 조건을 설정하는 과제다.

### 제출한 SQL

```sql
CREATE DATABASE product;

CREATE TABLE products
(
    product_id              SERIAL              PRIMARY KEY,
    product_name            VARCHAR(30)         NOT NULL,
    product_price           INT                 NOT NULL,
    product_stock           INT                 NOT NULL,
    product_category        VARCHAR(30)         NOT NULL,
    product_descrption      VARCHAR(100)
);

CREATE TABLE categories
(
    category_id             SERIAL              PRIMARY KEY,
    category_name           VARCHAR(30)         NOT NULL,
    category_description    VARCHAR(100)        NOT NULL
);

CREATE TABLE carts
(
    cart_id                 SERIAL              PRIMARY KEY,
    cart_cust_id            INT                 NOT NULL UNIQUE,
    product_id              INT                 NOT NULL REFERENCES products (product_id) ON DELETE CASCADE,
    cart_qty                INT                 NOT NULL
);
```

### 직접 실행해 본 결과

| 넣어 본 것 | 결과 |
| --- | --- |
| 7번 사용자가 사과를 담은 뒤 바나나를 또 담기 | `carts_cart_cust_id_key` 위반으로 거절 |
| 카테고리가 `없는카테고리`인 상품 등록 | 그대로 들어감 |

### 고친 SQL

- `carts`: 한 행은 "장바구니에 담긴 상품 하나"다. `cart_cust_id`에만 `UNIQUE`를 걸면 사용자 한 명이 상품을 하나밖에 못 담는다. 막아야 할 것은 같은 사용자가 같은 상품을 두 줄로 담는 것이라, 두 열의 조합에 `UNIQUE`를 건다.
- `products`: 카테고리를 글자로 적으면 없는 이름도 들어간다. 종목과 강좌처럼 카테고리 번호(정수)를 적고 `REFERENCES`로 `categories`를 가리킨다.
- 순서: SQL은 위에서부터 차례로 실행된다. 가리킬 표가 먼저 있어야 하니 `categories`를 `products`보다 먼저 만든다. 거꾸로면 `relation "categories" does not exist`가 난다.

```sql
CREATE DATABASE product;

CREATE TABLE categories
(
    category_id             SERIAL              PRIMARY KEY,
    category_name           VARCHAR(30)         NOT NULL,
    category_description    VARCHAR(100)        NOT NULL
);

CREATE TABLE products
(
    product_id              SERIAL              PRIMARY KEY,
    product_name            VARCHAR(30)         NOT NULL,
    product_price           INT                 NOT NULL,
    product_stock           INT                 NOT NULL,
    product_category        INT                 NOT NULL REFERENCES categories (category_id),
    product_descrption      VARCHAR(100)
);

CREATE TABLE carts
(
    cart_id                 SERIAL              PRIMARY KEY,
    cart_cust_id            INT                 NOT NULL,
    product_id              INT                 NOT NULL REFERENCES products (product_id) ON DELETE CASCADE,
    cart_qty                INT                 NOT NULL,
    UNIQUE(cart_cust_id, product_id)
);
```

| 넣어 본 것 | 결과 |
| --- | --- |
| 7번 사과, 7번 바나나, 8번 사과 | 모두 들어감 |
| 7번이 사과를 또 담기 | `carts_cart_cust_id_product_id_key` 위반으로 거절 |
| 없는 카테고리 9번으로 상품 등록 | `products_product_category_fkey` 위반으로 거절 |
