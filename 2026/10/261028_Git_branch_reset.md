## 1. 기본

### 명령어 연속 실행 (&&)

```bash
git add . && git commit -m "docs: README.md 추가"
```

- `&&`: 앞 명령이 **성공하면** 뒤 명령 실행. 실패하면 중단
- `;`: 성공 여부와 관계없이 이어서 실행
- `&` 하나만 쓰면 백그라운드 실행이라 의미가 다르다

### .gitignore

- Git이 추적하지 않을 파일/폴더를 지정하는 파일. 프로젝트 최상위에 둔다.
- 사용하는 이유
    - 비밀 정보 보호: `.env`, API 키
    - 불필요한 용량 제외: `node_modules/`, 빌드 결과물
    - 개인 환경 파일 제외: 로그, `.DS_Store`, IDE 설정
- 작성 예시

```text
# 주석
Logs/
node_modules/
.env
*.log
!important.log   # ! 는 예외 처리
```

- 관리
    - 프로젝트 시작 시 먼저 만들어 두고, 새로 제외할 대상이 생기면 한 줄씩 추가한다.
    - `.gitignore` 파일 자체는 커밋해서 팀과 공유한다.
- 주의
    - 이미 추적 중인 파일은 `.gitignore`에 추가해도 계속 추적된다. 추적을 끊으려면 아래처럼 한다.

```bash
git rm --cached <파일>
```

- 이미 커밋된 비밀키는 파일을 지워도 이력에 남는다. 처음부터 ignore 처리한다.

### 커밋 메시지 컨벤션

형식: `타입: 제목`

| 타입 | 의미 |
| --- | --- |
| `feat` | 새 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서 수정 |
| `style` | 코드 포맷, 세미콜론 등 (동작 변화 없음) |
| `refactor` | 동작은 같고 코드 구조만 개선 |
| `test` | 테스트 코드 추가/수정 |
| `chore` | 빌드, 설정 등 기타 작업 |
| `rename` | 파일/폴더 이름 변경 |
| `remove` | 파일 삭제 |

- 예: `docs: README.md 추가`, `feat: .gitignore 파일 생성`
- 팀마다 타입 구성이 다르니 프로젝트 규칙을 먼저 확인한다. (따로 한번 볼 것)

## 2. 원격 저장소

### git clone

```bash
git clone https://github.com/dhchun1203/kant-study.git
git clone https://github.com/dhchun1203/kant-study.git my-folder   # 폴더명 지정
```

- 원격 저장소를 이력까지 통째로 복제한다.
- 저장소 이름의 폴더가 생기고, `origin`이 자동으로 연결된다.

### git log 읽기

```text
60302dc (HEAD -> main) docs: Logs 폴더 제외
77bb06b (origin/main) feat: .gitignore 파일 생성
0d56a1e docs: README.md 추가
```

- 맨 앞 7자리는 커밋 해시(ID)의 축약형
- `HEAD -> main`: 현재 위치. 로컬 main 브랜치의 최신 커밋
- `origin/main`: 원격 main이 가리키는 커밋
- 로컬 main이 원격보다 1커밋 앞서 있다 → `git push` 필요
- 색상: 초록은 로컬(main), 빨강은 원격(origin/main) (터미널 기본 색상 기준)

## 3. 되돌리기

### git reset

```bash
git reset --soft 7ac7366
```

| 옵션 | HEAD(커밋) | Staging | 작업 폴더 |
| --- | --- | --- | --- |
| `--soft` | 이동 | 유지 | 유지 |
| `--mixed` (기본값) | 이동 | 초기화 | 유지 |
| `--hard` | 이동 | 초기화 | 초기화 (변경 내용 사라짐) |

- `--soft`: 커밋만 취소하고 다시 커밋하고 싶을 때
- `--hard`: 작업 내용까지 버릴 때. 주의해서 사용
- 이미 push해서 팀원이 가져간 커밋은 `reset` 대신 `revert`

### git reflog

```bash
git reflog
```

- HEAD가 움직인 모든 기록(commit, reset, checkout 등)을 보여준다. 로컬에만 존재한다.
- 위급할 때 사용: `reset --hard`로 날린 커밋도 reflog에서 해시를 찾아 복구할 수 있다.

```bash
git reflog                    # 되돌아갈 지점 확인
git reset --hard HEAD@{1}     # 한 단계 전 상태로 복구
```

- 커밋하지 않은 변경은 reflog로도 복구되지 않는다.

## 4. 브랜치

### 브랜치란

- 특정 커밋을 가리키는 포인터. 독립적인 작업 줄기를 만든다.
- 따로 빼서 작업하는 이유
    - main을 항상 안정적인 상태로 유지
    - 기능별로 병렬 작업 가능
    - 기능 단위로 리뷰(PR)하고 합치기 쉽다
    - 실험하다 실패해도 버리면 그만이다

### 생성과 이동

```bash
git branch feature/login            # 생성만 (이동 안 함)
git checkout -b feature/login       # 생성 + 이동
git branch                          # 목록
```

### git stash

- 아직 커밋하지 않은 변경을 임시 보관하고 작업 폴더를 깨끗하게 만든다.
- 언제 쓰나: 작업 중에 급하게 다른 브랜치로 이동해야 하는데 커밋하기엔 애매할 때

```bash
git stash                  # 임시 보관
git stash list             # 보관 목록
git stash pop              # 꺼내서 적용하고 목록에서 삭제
git stash apply            # 꺼내서 적용하되 목록에 유지
git stash -u               # untracked 파일까지 포함
```

### git merge

```bash
git checkout main
git merge feature/login    # 합치려는 대상 브랜치(main)로 이동해서 실행
```

**Fast-forward**: main에 새 커밋이 없을 때. 포인터만 앞으로 이동하고 merge 커밋이 생기지 않는다.

```text
전: A - B (main)
          \ C - D (feature)
후: A - B - C - D (main, feature)
```

**3-way merge**: 양쪽 브랜치에 각각 새 커밋이 있을 때. 공통 조상과 두 브랜치의 끝, 3곳을 비교해 merge 커밋을 만든다.

```text
      C - D (feature)
     /       \
A - B - E ---- M (main)
```

- Fast-forward여도 merge 커밋을 남기고 싶으면 `--no-ff` 옵션

### 충돌 (Conflict)

**발생 시나리오**

- 같은 파일의 같은 부분을 두 브랜치에서 서로 다르게 수정한 뒤 merge할 때
- Git이 어느 쪽을 택할지 판단하지 못해 멈춘다. (`git status`에 `both modified`로 표시)

파일에 이렇게 표시된다.

```text
<<<<<<< HEAD
main의 내용
=======
feature/login의 내용
>>>>>>> feature/login
```

**해결 방법**

1. 충돌 파일을 열어 최종 내용으로 직접 수정하고 `<<<<<<<`, `=======`, `>>>>>>>` 표시를 지운다.
2. `git add <파일>`
3. `git commit` (merge 중이면 메시지가 자동 생성된다)

- 중간에 포기하려면 `git merge --abort`

**GitFlow와 연계**

- GitFlow: `main`(출시) · `develop`(개발 통합) · `feature/*`(기능) · `release/*`(출시 준비) · `hotfix/*`(긴급 수정). 팀에 맞게 조정한다.
- feature 브랜치는 develop에서 분기하고, 완료되면 PR로 develop에 merge한다.
- 충돌은 주로 이 merge 시점에 나온다. 같은 파일을 여러 feature에서 건드렸을 때 발생한다.
- 예방: feature가 오래 살수록 충돌이 커지므로 develop의 변경을 자주 가져와 맞춘다.
- 충돌은 보통 PR을 올리기 전에 feature 쪽에서 해결하고 리뷰를 받는다.

### 브랜치 삭제 (merge 후 정리)

```bash
git branch --delete feature/login   # 병합이 끝난 깔끔한 상태일 때만 삭제 (-d 와 동일)
git branch -D feature/login         # 병합 여부와 관계없이 강제 삭제
```

- 현재 체크아웃한 브랜치는 삭제할 수 없다.
