# 터미널과 Git 기초

터미널에 익숙해져라. 멋지잖아.

# 1. 터미널 Git 실습

> 🎯 앞으로 Git은 GUI 대신 **터미널**에서 사용한다. 목적은 터미널에 익숙해지는 것 → 추후 리눅스 환경 적응.

## 0. Git 개념 정리 (2026-10-26)

### Git과 GitHub

**Git**

- 리눅스 창시자 토발즈가 2005년에 개발함
- 분산형: 모든 개발자가 전체 이력을 가진 로컬 저장소를 갖는다.
  - 서버 연결 없이도 커밋 이력 조회, 브랜치 작업이 가능

**GitHub**

- Git 원격 저장소를 호스팅하는 웹 서비스
- 2018년 마이크로소프트에서 인수함
- Pull Request, Issues, Actions 등 협업 기능 제공
- 개발자 포트폴리오로도 활용

> 💡 Git은 내 컴퓨터에서 동작하는 도구, GitHub는 Git 저장소를 올려 두고 함께 쓰는 서비스이다. GitHub 없이도 Git은 쓸 수 있다.

### 필수 개념

#### 리포지토리 (Repository)

- 로컬 / 원격 리포지토리로 나뉜다.
- 저장소의 실체는 프로젝트 폴더 안의 `.git` 폴더이다. `.git`을 지우면 이력이 모두 사라진다.

#### Git의 3가지 영역

| 영역 | 설명 |
| --- | --- |
| Working Directory (작업 폴더) | 실제로 파일을 편집하는 곳 |
| Staging Area (인덱스) | 다음 커밋에 담을 변경을 골라 두는 영역. `.git/index` 파일로 관리되는 논리적 영역 |
| Repository | 커밋(스냅샷 - 수정된 내용만 저장된다는 의미) 이력이 저장되는 영역 |

#### 핵심 명령어 흐름

```txt
add → commit → push / pull
```

- `add`: 스테이징 영역에 추가함
- `commit`: 로컬 리포지토리에 이력을 남김
- `push`: 원격 리포지토리에 이력을 업로드
- `pull`: 원격 리포지토리에서 이력을 다운로드

#### 원격 저장소가 왜 필요한가

1. 서로 다른 곳에 있는 개발자가 하나의 원격 저장소를 통해 작업을 주고받는다.
2. 한 사람이 사무실과 집에서 같은 프로젝트를 이어서 작업한다.

### git add

파일(변경사항) 중 이력에 남길 것만 골라 Staging Area에 올린다.

### git commit

Staging Area의 변경 내역을 하나의 커밋으로 저장소에 저장한다.

#### 커밋은 누적된다

각 커밋은 고유한 해시(ID)를 가지며, 원하는 커밋으로 되돌아갈 수 있다.

### 되돌리기: git reset vs git revert

| 명령어 | 사용 상황 |
| --- | --- |
| `git reset` | 변경 이력을 **가져간 팀원이 없을 때** |
| `git revert` | 이미 **가져간 팀원이 있을 때** |

**git reset**

- 현재 브랜치(HEAD)를 지정한 커밋으로 이동시킨다. 옵션에 따라 Staging, 작업 폴더까지 되돌린다.
- 옵션 정리
  - `--soft`: 커밋만 취소 (변경 내용은 Staging에 유지)
  - `--mixed` (기본값): 커밋과 Staging을 취소 (작업 폴더의 변경은 유지)
  - `--hard`: 커밋, Staging, 작업 폴더까지 모두 되돌림 (변경 내용 사라짐, 주의)

**git revert**

- 지정한 커밋의 변경을 반대로 적용한 새 커밋을 만든다. 이력은 그대로 남는다.

### git branch

브랜치는 특정 커밋을 가리키는 움직이는 포인터이다. 본 작업과 분리된 독립적인 작업 줄기도 생성 가능하다.

```bash
git switch -c feature/jump
git checkout -b feature/jump   # 더 주로 사용함
```

### GitFlow 방법론

Git으로 협업하는 브랜치 전략이다.

- 절대적인 방법은 아니며 팀에 맞게 수정해서 쓴다.

**main 브랜치**

- 항상 출시 가능한 완성본
- 접근 및 병합은 팀장이 관리

## 1. Git 설치 (winget)

winget은 Windows 패키지 매니저이다.

```bash
winget install -i Git.Git
```

- `-i` (`--interactive`): 설치 옵션 선택 화면을 활성화
- 설치 중 확인할 옵션: 기본 에디터(VS Code 등), 기본 브랜치 이름(`main`), PATH 설정(Git from the command line...)
- 설치 후 확인

```bash
git --version
```

### winget search (패키지 검색)

설치하기 전에 패키지 이름과 **ID**를 확인할 때 쓴다.

```bash
winget search git                     # 이름/ID/태그에 git이 포함된 패키지 검색
winget search --id Git.Git            # ID로 검색
winget search git --source winget     # 검색 대상 저장소 지정 (winget / msstore)
winget search git --count 5           # 결과 개수 제한
```

검색 결과 열 구성

| 열 | 의미 |
| --- | --- |
| 이름 (Name) | 패키지 표시 이름 |
| ID | 패키지 고유 식별자 (예: `Git.Git`), 설치할 때 사용 |
| 버전 (Version) | 최신 버전 |
| 일치 (Match) | 검색어와 일치한 부분 |
| 원본 (Source) | 패키지 출처 (winget, msstore) |

- 이름이 비슷한 패키지가 많으므로 설치는 **ID 기준**으로 하는 것이 안전하다.
- 정확히 일치하는 항목만 대상으로 하려면 `-e` (`--exact`) 옵션을 사용한다.

```bash
winget install --id Git.Git -e
```

### winget 자주 쓰는 명령어

| 명령어 | 설명 |
| --- | --- |
| `winget search <검색어>` | 패키지 검색 |
| `winget show <ID>` | 패키지 상세 정보 (버전, 설명, 설치 옵션) |
| `winget install <ID>` | 설치 |
| `winget list` | 설치된 프로그램 목록 |
| `winget upgrade` | 업그레이드 가능한 프로그램 목록 |
| `winget upgrade <ID>` | 특정 프로그램 업그레이드 |
| `winget uninstall <ID>` | 삭제 |

## 2. 터미널 기본 명령어

### 2-1. 파일 목록 보기

```bash
ls          # 기본 목록
ls -a       # 숨김 파일 포함 (.git, .gitignore 등)
ls -l       # 상세 정보
ls -al      # 숨김 파일 + 상세 정보 (자주 쓴다)
```

`ls -al` 출력 해석 예시

```text
drwxr-xr-x  2 user group 4096 Oct 26 16:00 src
-rw-r--r--  1 user group   12 Oct 26 16:01 README.md
```

| 맨 앞 글자 | 의미 |
| --- | --- |
| `d` | 폴더 (directory) |
| `l` | 링크 (바로가기, symbolic link) |
| `-` | 일반 파일 |

뒤따르는 9글자는 `rwx`가 3번 반복되며 **접근 권한**을 나타낸다.

- `r` Read(읽기), `w` Write(쓰기), `x` Execute(실행), 없으면 `-`
- 순서: 소유자(user) → 그룹(group) → 그 외(others)
- 예: `rwxr-xr--` = 소유자는 읽기/쓰기/실행, 그룹은 읽기/실행, 그 외는 읽기만
- 권한 변경은 `chmod` (예: `chmod +x script.sh`)

### 2-2. 이동과 경로

```bash
pwd         # 현재 위치 출력
cd <폴더>    # 폴더 이동
cd ..       # 상위 폴더
cd ~        # 홈 폴더
cd -        # 이전에 있던 폴더로 돌아가기
```

### 2-3. 파일/폴더 조작

```bash
mkdir <폴더>        # 폴더 생성
touch <파일>        # 빈 파일 생성
cat <파일명>        # 파일 내용 조회 (예: cat README.md)
cp <원본> <대상>     # 복사
mv <원본> <대상>     # 이동 / 이름 변경
rm <파일>           # 파일 삭제 (휴지통 없이 바로 삭제되니 주의)
rm -r <폴더>        # 폴더 삭제
```

### 2-4. 현재 shell 확인과 출력

```bash
echo $SHELL                      # 현재 사용 중인 shell 표시
echo "Hello World"               # 문자열 출력
echo "Hello World" > README.md   # 출력 내용을 README.md 파일에 저장 (덮어쓰기)
echo "Second line" >> README.md  # 파일 끝에 추가
```

- `>` : 덮어쓰기 / `>>` : 이어쓰기 (리다이렉션)
- 변수는 `$`를 붙여 참조한다. `$SHELL`처럼 대문자 환경 변수가 많다.

### 2-5. history (명령어 기록)

```bash
history       # 실행했던 명령어 목록 (번호 포함)
!15           # 15번 명령어 다시 실행
!!            # 직전 명령어 다시 실행
```

## 3. 단축키

| 단축키 | 동작 |
| --- | --- |
| `Ctrl + u` | 입력 줄 전체 지우기 |
| `Ctrl + w` | 단어 단위로 지우기 |
| `Ctrl + a` / `Ctrl + e` | 줄의 맨 앞 / 맨 끝으로 이동 |
| `Ctrl + c` | 실행 중인 명령 중단 |
| `Ctrl + l` | 화면 정리 (`clear`와 동일) |
| `Ctrl + r` | 명령어 기록 검색 |
| `↑` / `↓` | 이전 / 다음 명령어 |
| `Tab` | 파일/폴더 이름 자동완성 |

## 4. Git 기본 설정

처음 한 번만 설정한다.

```bash
git config --global user.name "이름"
git config --global user.email "email@example.com"
git config --global init.defaultBranch main
git config --list      # 설정 확인
```

## 5. Git 기본 흐름 실습

```bash
mkdir git-practice && cd git-practice   # 실습 폴더 생성 후 이동
git init                                # 저장소 초기화 (.git 폴더 생성)
echo "Hello World" > README.md          # 파일 생성
git status                              # 변경 상태 확인
git add README.md                       # 스테이징 (커밋 대상에 올림)
git commit -m "docs: add README"        # 커밋 (스냅샷 저장)
git log --oneline                       # 커밋 기록 확인
```

### 작업 영역 개념

```text
Working Directory → (git add) → Staging Area → (git commit) → Repository
```

- Working Directory: 실제로 파일을 수정하는 곳
- Staging Area: 다음 커밋에 포함할 변경 사항을 모아두는 곳
- Repository: 커밋이 저장되는 곳 (`.git` 폴더)

### 자주 쓰는 명령어

| 명령어 | 설명 |
| --- | --- |
| `git status` | 현재 변경 상태 확인 |
| `git diff` | 수정했지만 아직 add 하지 않은 변경 내용 확인 |
| `git diff --staged` | add 한 변경 내용 확인 |
| `git add .` | 변경된 모든 파일 스테이징 |
| `git commit -m "메시지"` | 커밋 생성 |
| `git log --oneline` | 커밋 기록을 한 줄씩 보기 |
| `git branch` | 브랜치 목록 |
| `git switch -c <이름>` | 새 브랜치 생성 후 이동 |
| `git switch <이름>` | 브랜치 이동 |
| `git merge <이름>` | 브랜치 병합 |
| `git restore <파일>` | 수정 내용 되돌리기 |

## 6. 원격 저장소 (GitHub)

```bash
git clone <URL>                          # 원격 저장소 복제
git remote add origin <URL>              # 원격 저장소 연결
git push -u origin main                  # 원격에 업로드 (최초 1회 -u)
git pull                                 # 원격 변경 사항 가져오기
```

## 7. .gitignore

Git이 추적하지 않을 파일을 지정한다. 저장소 최상위에 `.gitignore` 파일을 만든다.

```text
node_modules/
.env
*.log
.DS_Store
```

- `.env` 같은 비밀 키 파일은 반드시 ignore 처리 (한번 커밋되면 기록에 남는다)

## 8. 주의할 점

- `rm`은 휴지통이 없다. 삭제 전에 `ls`로 대상 확인
- 커밋 메시지는 변경 이유가 드러나게 작성 (예: `feat:`, `fix:`, `docs:` 접두어 활용)
- `git status`를 습관처럼 자주 확인
- 이미 push한 커밋은 `reset --hard`, `push --force`로 함부로 고치지 않는다

