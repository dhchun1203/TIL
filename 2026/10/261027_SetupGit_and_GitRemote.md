# Git 설정과  Github 원격 저장소 연결

어려워도 필수 툴.. 익숙해지자

## 터미널

- VS Code 터미널 열기: `Ctrl + \``
- 폴더 삭제

```bash
rm -rf kant/
```

- `-r` 폴더 내부까지 삭제, `-f` 확인 없이 강제 삭제
- 휴지통 없이 바로 삭제된다. 특히 `rm -rf .`처럼 현재 폴더를 대상으로 하면 위험하니 `pwd`, `ls`로 확인 후 실행

## Git 사용자 설정

```bash
git config --global user.email "myemail@gmail.com"
git config --global user.name "Chun Dae Hwan"
```

기존 값 지우고 다시 설정

```bash
git config --global --unset-all user.name
git config --global user.name "Chun Dae Hwan"
```

설정 확인

```bash
git config --global --list
```

## .git/hooks

- Git 이벤트(commit, push 등) 전후에 자동 실행되는 스크립트를 두는 폴더
- 기본은 `.sample` 예시 파일. 확장자를 지우고 실행 권한을 주면 동작
- 대표 훅: `pre-commit`(커밋 직전), `commit-msg`(메시지 검사), `pre-push`(push 직전)
- `.git` 안에 있어 저장소에 포함되지 않는다. 팀 공유는 husky 같은 도구 사용

## 파일 상태 표시 (VS Code)

| 표시 | 상태 | 설명 |
| --- | --- | --- |
| `U` | Untracked | 아직 추적하지 않는 새 파일 |
| `A` | Added | index(Staging Area)에 추가됨 |
| `M` | Modified | 추적 중인 파일이 수정됨 |
| `D` | Deleted | 파일이 삭제됨 |

## 커밋 기록 조회

```bash
git log --oneline --all
```

- HEAD 위치와 관계없이 모든 브랜치의 커밋 표시
- 브랜치 구조까지 보려면 `--graph` 추가

```bash
git log --oneline --all --graph
```

## GitHub 신규 리포지토리 연결

GitHub에서 리포지토리를 직접 생성한 뒤 로컬에서 연결한다. 로컬에 이미 코드가 있다면 README, .gitignore 없는 빈 저장소로 만든다.

```bash
git remote add origin https://github.com/dhchun1203/Kant.git
git remote -v
git branch -M main
git push -u origin main
```

| 명령어 | 설명 |
| --- | --- |
| `git remote add origin <URL>` | 원격 저장소를 `origin` 이름으로 연결 |
| `git remote -v` | 연결된 원격 주소 확인 |
| `git branch -M main` | 현재 브랜치 이름을 `main`으로 변경 |
| `git push -u origin main` | 원격에 업로드하고 upstream 설정. 이후 `git push`, `git pull`만으로 사용 |

원격 연결 해제

```bash
git remote remove origin
```

## vim

기본 키 (임시 메모)

| 동작 | 키 |
| --- | --- |
| 입력 모드 진입 | `i` |
| 입력 모드 나가기 | `Esc` |
| 저장만 하기 | `:w` |
| 저장하고 나가기 | `:wq` |
| 나가기 | `:q` |
| 저장 없이 강제 종료 | `:q!` |

## 셀프 체크 질문

파일 수정하고 원격저장소에 올리기까지 루틴 설명할 수 있니?

---

이해도 4/5

