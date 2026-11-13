## 1. 오늘의 흐름

DB를 쓰려면 PostgreSQL 서버가 있어야 한다. 내 컴퓨터에 직접 설치하는 대신 Docker 컨테이너로 띄우면, 설치 과정 없이 명령 한 줄로 켜고 다 쓰면 지울 수 있다.

그런데 컨테이너를 지우면 그 안의 데이터도 같이 사라진다. DB는 데이터를 오래 담아 두려고 쓰는 건데 지울 때마다 사라지면 곤란하다. 그래서 컨테이너를 띄우기 전에 데이터를 보관할 볼륨부터 정하고, 처음부터 볼륨을 붙여서 띄운다.

서버를 띄웠으면 그 안에 DB와 표를 만들어야 한다. 매번 `CREATE TABLE`을 손으로 치는 건 번거롭고 실수도 생기니까, 만드는 과정을 SQL 파일 하나에 적어 두고 한 번에 실행한다. 그러면 언제든 같은 상태를 다시 만들 수 있다.

1. 볼륨: 데이터를 컨테이너 밖에 보관하는 이유와 다루는 명령
2. 볼륨을 붙여 컨테이너 띄우기
3. SQL 파일로 DB, 표, 데이터 만들기
4. 만든 DB에 골라 들어가 확인하기
5. (오후) 긴 `docker run`을 `compose.yaml`로 옮기기 (7번)

## 2. 구조

```txt
내 폴더 (pg_lab)                       컨테이너 my-pg (postgres:18)
kant_shop.sql ── docker cp ──▶ /tmp/kant_shop.sql ── psql -f ──▶ PostgreSQL 서버
kant_shop.sql ── psql < 파일 (그대로 흘려 보냄) ─────────────▶ PostgreSQL 서버
                                                                  └ DB kant_shop
                                                                     └ 표 product
                                       /var/lib/postgresql ◀── 볼륨 pg-data (컨테이너 밖 보관함)
```

SQL 파일 하나가 만드는 표는 이렇게 생겼다.

| 열 | 타입 | 제약 | 뜻 |
| --- | --- | --- | --- |
| `id` | `SERIAL` | `PRIMARY KEY` | 자동으로 1, 2, 3... 붙는 번호 |
| `name` | `VARCHAR(100)` | `NOT NULL` | 상품명 (100자까지) |
| `price` | `INT` | `NOT NULL` | 가격 |
| `stock` | `INT` | `NOT NULL` | 재고 |

## 3. 코드

### 1) 볼륨

```bash
docker volume ls                # 볼륨 목록
docker volume create [볼륨명]   # 볼륨 만들기
docker volume inspect [볼륨명]  # 볼륨 정보 보기

# 볼륨 지우기
docker volume rm [볼륨명]       # 이름으로 하나(또는 여러 개) 지우기
docker volume prune             # 안 쓰는 익명 볼륨 모두 지우기 (y/N 확인)
docker volume prune -a          # 안 쓰는 볼륨을 이름 있는 것까지 모두 지우기
```

- 볼륨은 컨테이너 밖에 있는 보관함이다. 컨테이너를 지워도 볼륨은 남는다.
- `docker run`에서 `-v 볼륨명:경로`로 붙이면, 볼륨이 없을 때는 그 이름으로 새로 만들어진다. 그래서 `docker volume create`를 미리 하지 않아도 된다.

### 2) 볼륨을 붙여 컨테이너 띄우기

```bash
docker run -d --name my-pg \
  -e POSTGRES_PASSWORD=admin \
  -p 5432:5432 \
  -v pg-data:/var/lib/postgresql \
  postgres:18
```

- `-v pg-data:/var/lib/postgresql`은 컨테이너 안에서 데이터가 저장되는 폴더를 볼륨 `pg-data`에 연결한다. `postgres:18`부터는 이 자리가 `/var/lib/postgresql`이다.
- 줄 끝의 `\`는 "명령이 다음 줄로 이어진다"는 표시다. 옵션이 길어지면 이렇게 나눠 써야 읽기 쉽다.
- 맨 끝의 `postgres:18`은 어떤 이미지로 만들지 정하는 자리라 빠지면 안 된다. 옵션을 다 쓰고 마지막에 온다.
- 켜졌는지는 `docker ps`, 꺼진 것까지 보려면 `docker ps -a`로 확인한다.

### 3) SQL 파일 (`kant_shop.sql`)

```sql
-- 1. 데이터베이스 생성 
DROP DATABASE IF EXISTS kant_shop;

CREATE DATABASE kant_shop;

-- 2. 새로 만든 데이터베이스에 접속(특정 DB를 사용하겠다.)
\c kant_shop

-- 3. 상품 테이블 생성
CREATE TABLE product 
(
    id SERIAL PRIMARY KEY, 
    name VARCHAR(100) NOT NULL,
    price INT NOT NULL,
    stock INT NOT NULL
);

-- 4. 상품 초기 데이터 입력
INSERT INTO product (name, price, stock) VALUES
('무선 이어폰', 35000, 100),
('게이밍 마우스', 28000, 85),
('27인치 모니터', 259000, 15),
('USB-C 허브', 19900, 200),
('노트북 거치대', 24000, 7);
```

- 맨 위에서 `DROP DATABASE IF EXISTS`로 지우고 시작하는 이유는, 이 파일을 몇 번을 실행해도 늘 같은 깨끗한 상태로 만들기 위해서다. DB가 없으면 지울 것도 없으니 건너뛴다.
- `\c`는 SQL이 아니라 psql 프로그램의 명령이다. 그래서 끝에 `;`가 없고, 이 파일은 psql로 실행해야 한다.
- `id`를 `SERIAL`로 만들었기 때문에 `INSERT`할 때 `id`는 적지 않는다. 번호는 DB가 알아서 붙인다.

### 4) SQL 파일을 컨테이너에 넣어 실행하기

방법은 두 가지다.

```bash
# 방법 1: 파일을 컨테이너 안으로 복사해 두고 실행
docker cp kant_shop.sql my-pg:/tmp/kant_shop.sql
docker exec -it my-pg psql -U postgres -f //tmp/kant_shop.sql

# 방법 2: 내 폴더의 파일을 입력으로 흘려 보내기
docker exec -i my-pg psql -U postgres < kant_shop.sql
```

- 방법 1의 `-f` 뒤 경로는 내 컴퓨터가 아니라 컨테이너 안의 경로다. 그래서 `docker cp`로 먼저 넣어 둬야 한다.
- 방법 2는 복사가 필요 없고 경로 문제도 없어서 더 간단하다.

### 5) 만든 DB에 들어가 확인하기

```bash
docker exec -it my-pg psql -U postgres -d kant_shop
docker exec -it my-pg psql -U postgres -d kant_shop -c "SELECT * FROM product"
```

- `-d kant_shop`을 붙이면 기본 DB(`postgres`)가 아니라 `kant_shop`으로 바로 들어간다.
- `-c`는 SQL 한 문장만 실행하고 바로 나온다. 확인만 할 때 편하다.

### 6) Git Bash 경로 변환 끄기 (`~/.bashrc`에 한 번만)

```bash
sed -i '/PATHCONV\|CONV_EXCL/d' ~/.bashrc
echo 'export MSYS_NO_PATHCONV=1' >> ~/.bashrc
echo 'export MSYS2_ARG_CONV_EXCL="*"' >> ~/.bashrc
source ~/.bashrc
```

## 4. 핵심 정리

**Q. 볼륨은 왜 붙이지?**

컨테이너를 지우면 그 안의 `/var/lib/postgresql`(데이터가 실제로 저장되는 폴더)도 같이 지워진다. 이 폴더를 컨테이너 밖의 볼륨 `pg-data`에 연결해 두면, 컨테이너를 지우고 같은 `-v`로 다시 만들어도 표와 데이터가 그대로 남는다.

**Q. `docker volume ls`에 긴 ID 이름의 볼륨은 왜 생겼지?**

`-v` 없이 DB 컨테이너를 켜면 Docker가 이름 없는 볼륨(익명 볼륨)을 자동으로 만든다. 컨테이너를 지워도 남기 때문에 여러 번 만들고 지운 만큼 쌓인다. 남길 데이터라면 처음부터 이름을 붙이는 게 좋다.

**Q. 볼륨을 붙였는데 비밀번호를 바꿔서 다시 만들면?**

바뀌지 않는다. `-e POSTGRES_PASSWORD`는 데이터가 비어 있을 때 처음 한 번만 읽는다. 이미 데이터가 든 볼륨을 붙이면 처음 정한 비밀번호가 그대로다.

**Q. 볼륨은 어떻게 지우지?**

순서가 중요하다. 볼륨을 쓰는 컨테이너가 남아 있으면(꺼져 있어도) 볼륨을 지울 수 없다. 그래서 컨테이너를 먼저 지우고 볼륨을 지운다.

```bash
docker rm -f my-pg          # 1. 볼륨을 쓰는 컨테이너부터 지우기
docker volume rm pg-data    # 2. 그다음 볼륨 지우기
docker volume ls            # 3. 지워졌는지 확인
```

| 명령 | 지우는 것 |
| --- | --- |
| `docker volume rm 이름` | 이름을 적은 볼륨 |
| `docker volume prune` | 아무 컨테이너도 안 쓰는 **익명** 볼륨만 (긴 ID 이름들) |
| `docker volume prune -a` | 아무 컨테이너도 안 쓰는 볼륨 전부 (이름 있는 것 포함) |
| `docker compose down -v` | compose 파일에 선언한 볼륨과 익명 볼륨. `external`로 선언한 볼륨은 지우지 않음 |

`prune`은 실행하면 "정말 계속할까요? [y/N]"를 물어본다. 오전에 쌓인 긴 ID 이름의 익명 볼륨은 `docker volume prune` 한 번으로 치울 수 있다. 지운 볼륨 안의 데이터는 되돌릴 수 없다.

**Q. 파일을 `<`로 넘길 때는 왜 `-it`가 아니라 `-i`만 쓰지?**

`-i`는 내 쪽 입력을 컨테이너로 보내고, `-t`는 터미널 화면을 붙인다. 사람이 직접 타이핑할 때는 둘 다 필요하다. 하지만 파일을 흘려 보낼 때는 입력이 키보드가 아니라 파일이라서, 화면을 붙이라는 `-t`와 맞지 않아 오류가 난다. 그래서 `-i`만 쓴다.

**Q. `/tmp/kant_shop.sql`이라고 썼는데 왜 `C:/Users/.../Temp/...`를 못 찾는다고 하지?**

Git Bash(UCRT64)는 `/`로 시작하는 인자를 Windows 경로로 바꿔서 넘긴다. 그래서 컨테이너 안의 `/tmp`가 Windows 임시 폴더 경로로 바뀌어 버렸다. 슬래시를 두 번(`//tmp`) 쓰면 바꾸지 않고, `~/.bashrc`에 변환을 끄는 변수를 넣어 두면 매번 신경 쓰지 않아도 된다. Git Bash는 `MSYS_NO_PATHCONV`, MSYS2는 `MSYS2_ARG_CONV_EXCL`을 쓴다.

**Q. `NOT NULL`인 열을 비우면 어떻게 되지?**

거절된다. 직접 실행하니 `stock`을 빼고 넣었을 때 `null value in column "stock" ... violates not-null constraint` 오류가 났다. 꼭 있어야 하는 값을 DB가 대신 지켜 주는 장치다.

**Q. `SERIAL` 번호는 항상 빈틈없이 1씩 늘까?**

아니다. 직접 실행하니 실패한 `INSERT`도 번호를 하나 써 버려서, 5번 다음에 넣은 행이 6이 아니라 7번을 받았다. 번호는 "겹치지 않는 것"만 보장되고 "빈틈 없음"은 보장되지 않는다.

## 5. 오늘 만난 오류

에러는 맨 앞 단어부터 읽는다. `bash:`면 Docker까지 가기 전에 터미널에서 실패한 것이고, `psql:`이면 컨테이너 안의 psql이, `docker:`면 Docker가 낸 것이다.

| 메시지 | 원인 | 해결 |
| --- | --- | --- |
| `psql: error: C:/Users/.../Temp/kant_shop.sql: No such file or directory` | Git Bash가 `/tmp`를 Windows 경로로 바꿈 | `//tmp/...` 또는 경로 변환 끄기 |
| `cannot attach stdin to a TTY-enabled container because stdin is not a terminal` | 파일을 `<`로 넘기면서 `-t`를 붙임 | `-it` 대신 `-i` |
| `bash: kant.shop.sql: No such file or directory` | 파일 이름 오타 (`_`를 `.`로 침) | `ls *.sql`로 이름 확인, Tab 자동 완성 |
| `Conflict. The container name "/my-pg" is already in use` | 같은 이름의 컨테이너가 남아 있음 | `docker start my-pg`로 켜서 들어가거나 `docker rm -f my-pg` 뒤 새로 |
| `bash: ‘export: command not found` | 복사한 명령에 둥근 따옴표(`‘`)가 들어감 | 일반 따옴표(`'`)로 직접 타이핑 |

## 6. 실행 예시

SQL 파일을 처음 실행하면 문장마다 결과가 한 줄씩 나온다. 첫 줄의 `NOTICE`는 오류가 아니라 "지울 DB가 없어서 건너뛴다"는 안내다. `INSERT 0 5`는 5행이 들어갔다는 뜻이다.

```txt
psql:kant_shop.sql:2: NOTICE:  database "kant_shop" does not exist, skipping
DROP DATABASE
CREATE DATABASE
You are now connected to database "kant_shop" as user "postgres".
CREATE TABLE
INSERT 0 5
```

`-d kant_shop -c "SELECT * FROM product"`로 확인하면 `id`가 1부터 자동으로 붙어 있다.

```txt
 id |     name      | price  | stock 
----+---------------+--------+-------
  1 | 무선 이어폰   |  35000 |   100
  2 | 게이밍 마우스 |  28000 |    85
  3 | 27인치 모니터 | 259000 |    15
  4 | USB-C 허브    |  19900 |   200
  5 | 노트북 거치대 |  24000 |     7
(5 rows)
```

## 7. 오후: Docker Compose로 옮기기

오전에 친 `docker run` 명령은 옵션이 다섯 개나 붙어서 길다. 매번 이걸 치면 번거롭고, 하나만 빠뜨려도 다르게 만들어진다. 그래서 같은 내용을 `compose.yaml` 파일에 적어 두고, `docker compose up -d` 한 줄로 켜고 `down`으로 끈다. 명령을 외우는 대신 파일을 읽으면 어떻게 띄웠는지 바로 보인다.

### 구조

`docker run`의 옵션 하나하나가 `compose.yaml`의 한 자리로 옮겨진다.

| `docker run` | `compose.yaml` | 이번 값 |
| --- | --- | --- |
| 이미지 (맨 끝) | `image:` | `postgres:18` |
| `--name` | `container_name:` | `kant-pg` |
| `-e` | `environment:` | `POSTGRES_PASSWORD: admin` |
| `-p` | `ports:` | `"5433:5432"` |
| `-v` | `volumes:` (서비스 안) | `pg-data:/var/lib/postgresql` |
| (없음) | `volumes:` (맨 바깥) | 쓸 볼륨을 선언. `external: true` |

### 코드

`compose.yaml`

```yaml
services:                 # 실행할 컨테이너(서비스) 목록 — 필수
  db:              # 예: db, web (직접 정하는 이름)
    image: postgres:18      # 사용할 이미지
    container_name: kant-pg   # 컨테이너 이름 (생략 시 자동 생성)
    environment:      
        POSTGRES_PASSWORD: admin     # 환경 변수 (-e)
    ports:                 # 포트 연결 (-p) — 문자열로 감싸는 것을 권장
      - "5433:5432"
    volumes:               # 볼륨 연결 (-v)
      - pg-data:/var/lib/postgresql

volumes:                  # 사용할 볼륨 선언 (최상위)
  pg-data:
   external: true
```

```bash
docker compose up -d     # compose.yaml대로 켜기 (뒤에서)
docker compose ps        # 이 폴더의 compose 컨테이너 보기
docker compose down      # 끄고 컨테이너 지우기

docker exec -it kant-pg psql -U postgres -d kant_shop -c "SELECT * FROM product"
```

### 핵심 정리

**Q. 서비스 이름 `db`와 `container_name`의 `kant-pg`는 뭐가 다르지?**

`db`는 compose 파일 안에서 이 서비스를 부르는 이름이고, `kant-pg`는 `docker ps`에 보이는 실제 컨테이너 이름이다. `container_name`을 안 쓰면 compose가 `폴더이름-서비스이름-번호`로 이름을 붙인다(어제 본 `cafe-web-1`). 이름을 정해 뒀기 때문에 `docker exec -it kant-pg ...`처럼 바로 부를 수 있다.

**Q. 포트를 왜 `5432:5432`가 아니라 `5433:5432`로 했지?**

`:` 앞은 내 컴퓨터의 문, 뒤는 컨테이너 안의 문이다. 내 컴퓨터의 5432번은 오전에 띄운 `my-pg`가 이미 쓰고 있어서, 같은 번호로 열면 `port is already allocated`로 겹친다. 그래서 바깥 문만 5433으로 바꿨다. 컨테이너 안의 PostgreSQL은 그대로 5432에서 기다린다. `docker exec`로 안에서 들어갈 때는 바깥 포트와 상관없다.

**Q. 맨 아래 `external: true`는 뭘 하지?**

"이 볼륨은 이미 밖에 있으니 그걸 그대로 써라"는 뜻이다. Docker 문서에 따르면 이때 compose는 볼륨을 새로 만들지 않고, 없으면 에러를 낸다. `external`이 없으면 compose는 `프로젝트이름_pg-data`처럼 앞에 이름을 붙인 새 볼륨을 만든다. 그러면 오전에 `docker run -v pg-data:...`로 만든 볼륨과 다른 보관함이 된다. `external: true`로 오전의 `pg-data`를 그대로 붙였기 때문에, `kant-pg`에서도 오전에 만든 `kant_shop`의 `product` 표를 조회할 수 있다.

**Q. `docker compose down`을 하면 데이터도 지워질까?**

`down`은 컨테이너를 끄고 지우지만 볼륨은 남긴다. 데이터는 볼륨에 있으니 다시 `up -d`하면 그대로다. 볼륨까지 지우는 건 `down -v`다. 단, Docker 문서에 따르면 `external`로 선언한 볼륨은 `down -v`로도 지워지지 않는다. 이번 `pg-data`는 `external: true`라서 지우려면 `docker volume rm pg-data`로 직접 지워야 한다.

**Q. YAML은 뭘 조심해야 하지?**

들여쓰기가 곧 구조다. 같은 단계의 키는 같은 칸에 맞추고, 한 단계 들어갈 때마다 더 들여 쓴다. `-`로 시작하는 줄은 목록의 한 항목이다. 포트는 `"5433:5432"`처럼 따옴표로 감싸 문자열로 쓰는 것을 권장한다.
