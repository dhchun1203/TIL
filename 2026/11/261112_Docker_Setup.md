## 1. 오늘의 흐름

1. **Docker 1:** 이미지를 받아 컨테이너를 켜고, 끄고, 지운다. `-p`로 포트를 잇는다.
2. **Docker 2:** 컨테이너 안에 들어가고(`exec`), 볼륨으로 데이터를 남기고, Dockerfile로 장바구니를 이미지로 만들고, `compose.yaml`로 짧게 켜고 끈다.
3. **nginx:** 파일을 내주는 웹 서버, 앱 여러 개를 80번 포트 하나로 묶는 리버스 프록시.
4. **데이터베이스 기초:** 표, 행, 열, 타입, 기본 키와 SQL 네 문장.
5. **Docker 3, 4:** MySQL과 PostgreSQL을 컨테이너로 켜서 표를 만들고, 볼륨으로 지킨다.

## 2. 구조

| 이름 | 비유 | 만드는 명령 | 지우는 명령 |
| --- | --- | --- | --- |
| 이미지 | 붕어빵 틀 (바뀌지 않음) | `docker pull`, `docker build -t 이름 .` | `docker rmi` |
| 컨테이너 | 붕어빵 (틀로 여러 개) | `docker run` | `docker rm` |
| 볼륨 | 붕어빵 밖 보관함 | `-v 이름:경로` | `docker volume rm` |

```txt
Docker Hub ── docker pull ──▶ 내 컴퓨터의 이미지 ── docker run ──▶ 컨테이너
                                                                    │ -v
                                                                 볼륨 (지워도 남음)
```

```txt
브라우저 ── localhost/menu/ ──▶ nginx (80번) ── proxy_pass ──▶ menu 앱
                                    └─ 파일은 바로 줌 (site/index.html)
```

```txt
DB 컨테이너 ─ 데이터베이스 서버(DBMS) ─ 데이터베이스 cafe ─ 표 orders (id, menu, qty)
```

## 3. 명령

컨테이너 켜고 끄기

```bash
docker pull nginx
docker run -d --name web -p 8080:80 nginx   # 뒤에서, 이름 web, 내 8080 → 컨테이너 80
docker ps            # 켜진 것
docker ps -a         # 꺼진 것까지
docker stop web
docker start web
docker logs web
docker rm web        # 꺼진 뒤에
docker rmi nginx     # 쓰는 컨테이너가 없어야 함
docker run -it --name my-server ubuntu:24.04 bash   # 들어가기, exit하면 꺼짐
```

들어가기, 볼륨, 내 이미지

```bash
docker exec -it web bash                                    # 켜 둔 컨테이너에 들어가기, exit해도 계속 돈다
docker run -d --name web -p 8080:80 -v cafe-html:/usr/share/nginx/html nginx   # 볼륨
docker run -d --name web -p 8080:80 -v ./site:/usr/share/nginx/html nginx      # 내 폴더
docker build -t cart .
docker run --rm cart                                        # 끝나면 컨테이너 자동 삭제
docker compose up -d
docker compose down
```

```dockerfile
FROM astral/uv:python3.14-bookworm-slim
WORKDIR /app
COPY . .
RUN uv sync
CMD ["uv", "run", "main.py"]
```

nginx 리버스 프록시 (`conf/default.conf`)

```nginx
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /menu/ {
        proxy_pass http://menu/;
    }
}
```

```bash
docker exec cafe-web-1 nginx -t           # 설정 검사
docker exec cafe-web-1 nginx -s reload    # 끄지 않고 다시 읽기
```

SQL 네 문장

```sql
CREATE DATABASE cafe;
CREATE TABLE orders (id INT PRIMARY KEY, menu VARCHAR(20), qty INT);
INSERT INTO orders VALUES (1, 'latte', 2);
SELECT * FROM orders WHERE id = 1;
```

MySQL / PostgreSQL 컨테이너

```bash
docker run -d --name db -e MYSQL_ROOT_PASSWORD=cafe1234 -e MYSQL_DATABASE=cafe -p 3306:3306 -v cafe-data:/var/lib/mysql mysql:8.4
docker exec -it db mysql -u root -p cafe

docker run -d --name pg -e POSTGRES_PASSWORD=cafe1234 -p 5432:5432 -v pg-data:/var/lib/postgresql postgres:18
docker exec -it pg psql -U postgres
```

## 4. 핵심 정리

- **컨테이너 상태:** 켜짐(`Up`) → 꺼짐(`Exited`, `docker ps -a`에만 보임) → 없어짐(`docker rm`). `hello-world`처럼 할 일을 끝내면 스스로 `Exited (0)`이 된다.
- **포트:** `-p 8080:80`은 내 컴퓨터 8080번 문을 컨테이너 80번 문에 잇는다. 포트는 컨테이너를 만들 때만 정할 수 있어서, 나중에 붙이려면 지우고 새로 만든다.
- **지우면 사라진다:** 컨테이너 안에서 고친 파일은 컨테이너와 함께 지워진다. 남겨야 하면 볼륨(`-v 이름:경로`)이나 내 폴더(`-v ./폴더:경로`)에 둔다. `./`를 빼면 오류 없이 같은 이름의 볼륨이 새로 생긴다.
- **Dockerfile:** `FROM`, `WORKDIR`, `COPY`, `RUN`은 build할 때, `CMD`는 run할 때마다 실행된다. 이미지에는 build한 시점의 코드가 들어 있어서, 코드를 고치면 build부터 다시 한다. `.dockerignore`로 `.venv`, `__pycache__`, `.git`을 뺀다.
- **compose:** 긴 `docker run` 옵션을 `compose.yaml`에 적는다. `ports`는 `-p`, `volumes`는 `-v`, `environment`는 `-e` 자리다. `down`은 볼륨을 남기고 `down -v`는 볼륨까지 지운다.
- **웹 서버와 앱 서버:** nginx는 폴더의 파일을 그대로 주고, 앱 서버는 코드를 실행해 답을 만든다. 리버스 프록시는 nginx가 요청을 대신 받아 뒤의 앱에 넘기고 답을 그대로 돌려주는 것이다. 밖으로 여는 포트는 nginx의 80번 하나로 두고 앱은 경로(`/menu/`, `/order/`)로 고른다.
- **nginx 설정:** `proxy_pass`에는 `localhost`가 아니라 compose 서비스 이름을 적는다(컨테이너 안의 `localhost`는 nginx 자신). 끝의 `/`가 있으면 `/menu/`를 떼고 넘긴다. 설정을 고치면 `nginx -t` 뒤 `nginx -s reload`.
- **데이터베이스:** 프로그램을 꺼도 남고, 여러 프로그램의 요청을 차례로 받아 지켜 주고, 조건으로 바로 찾는다. 기본 키는 행마다 겹치지 않고 비울 수 없는 값이다. SQL은 `;`로 끝나고, 글자 값은 작은따옴표, `WHERE`의 같다는 `=` 하나다.
- **DB 컨테이너:** `-e`로 준 비밀번호와 init 폴더(`/docker-entrypoint-initdb.d`)는 데이터 폴더가 비어 있을 때 처음 한 번만 읽는다. 볼륨이 남아 있으면 새 비밀번호를 줘도 예전 것이 그대로다.

| | MySQL | PostgreSQL |
| --- | --- | --- |
| 이미지, 포트 | `mysql:8.4`, 3306 | `postgres:18`, 5432 |
| 관리자, 비밀번호 `-e` | `root`, `MYSQL_ROOT_PASSWORD` | `postgres`, `POSTGRES_PASSWORD` |
| 들어가기 | `mysql -u root -p` | `psql -U postgres` (컨테이너 안에서는 비밀번호를 안 물음) |
| DB 목록, 옮기기 | `SHOW DATABASES;`, `USE shop;` | `\l`, `\c shop` |
| 표 목록, 표 모양 | `SHOW TABLES;`, `DESCRIBE orders;` | `\dt`, `\d orders` |
| 나가기 | `exit` | `\q` |
| 볼륨 자리 | `/var/lib/mysql` | `/var/lib/postgresql` (18부터) |
| 앱 계정 `-e` | `MYSQL_USER`: 일반 계정을 하나 더 | `POSTGRES_USER`: 관리자 이름이 바뀜 |

## 5. 자주 나는 오류

| 메시지 | 원인 | 해결 |
| --- | --- | --- |
| `failed to connect to the docker API` | Docker Desktop 엔진이 꺼져 있음 | Docker Desktop을 열고 Engine running을 기다린다 |
| `The container name "/web" is already in use` | 같은 이름의 컨테이너가 남아 있음(꺼져 있어도) | `docker ps -a`로 보고 `docker rm` |
| `port is already allocated` | 그 포트를 다른 컨테이너가 씀 | 꺼두거나 `-p 8081:80`처럼 다른 포트 |
| `container is running: stop the container before removing` | 켜진 컨테이너를 `rm` | `docker stop` 뒤 `docker rm` |
| Git Bash에서 경로 앞에 `C:/Program Files/Git`이 붙음 | Git Bash가 `/usr/...` 경로를 바꿈 | `export MSYS_NO_PATHCONV=1` |
| `502 Bad Gateway` | `proxy_pass`에 `localhost`를 적음 | 서비스 이름(`http://menu/`)으로 |
| 새 비밀번호로 켰는데 접속 거부 | 볼륨에 예전 데이터가 남아 `-e`를 안 읽음 | 예전 비밀번호로, 새로 하려면 볼륨 삭제 |
| 프롬프트가 `->` 또는 `-#` | SQL 끝의 `;`가 빠짐 | `;`만 치고 Enter |

## 6. 실행 예시

`docker ps` (포트를 잇고 켠 뒤)

```txt
CONTAINER ID   IMAGE     COMMAND                  CREATED                  STATUS                  PORTS                  NAMES
ba5cd077165e   nginx     "/docker-entrypoint.…"   Less than a second ago   Up Less than a second   0.0.0.0:8080->80/tcp   web
```

`docker logs web` (요청 기록: 보낸 주소, 시각, 요청, 결과 번호. 404는 파일 없음, 200은 성공)

```txt
172.17.0.1 - - [12/Nov/2026:02:02:30 +0000] "GET /event.html HTTP/1.1" 404 153 "-" "curl/8.5.0" "-"
172.17.0.1 - - [12/Nov/2026:02:02:30 +0000] "GET /event.html HTTP/1.1" 200 53 "-" "curl/8.5.0" "-"
```

MySQL에서 `SELECT * FROM orders;`

```txt
+----+-----------+------+
| id | menu      | qty  |
+----+-----------+------+
|  1 | latte     |    2 |
|  2 | americano |    1 |
+----+-----------+------+
2 rows in set (0.00 sec)
```
