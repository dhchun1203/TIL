# 04. Git Merge Conflict

git에서 충돌을 예방하고 해결하는 방법

## 1. 3-way merge

- merge는 두 브랜치의 끝(`current`, `incoming`)과 **베이스(`base`)** 3곳을 비교해서 합친다.
- `current`: 현재 브랜치(HEAD) / `incoming`: 합쳐 들어오는 브랜치

```text
      C - D (feature = incoming)
     /       \
A - B - E ---- M (main = current)
    ↑
    base (베이스)
```

비교 규칙: base와 달라진 쪽을 채택한다.

| base | current | incoming | 결과 |
| --- | --- | --- | --- |
| X | X | Y | Y 채택 (자동) |
| X | Y | X | Y 채택 (자동) |
| X | Y | Y | Y 채택 (양쪽이 같은 변경, 자동) |
| X | Y | Z | **충돌** (어느 쪽이 맞는지 판단 불가) |

## 2. 충돌이 나는 경우

- 같은 파일의 **같은 부분**을 서로 다르게 수정
- 한쪽은 파일을 수정하고 다른 쪽은 삭제
- 같은 파일이라도 **다른 줄**을 수정했다면 충돌 없이 자동으로 합쳐진다.

## 3. 직접 충돌 만들어 보기 (실습)

임시 폴더에서 연습한다.

```bash
mkdir conflict-test && cd conflict-test
git init
echo "hello" > a.txt
git add .
git commit -m "feat: a.txt 생성"

git checkout -b feature
echo "feature 수정" > a.txt
git add .
git commit -m "feat: feature에서 수정"

git checkout main             # 원래 브랜치로 돌아가기
echo "main 수정" > a.txt
git add .
git commit -m "feat: main에서 수정"

git merge feature             # 충돌 발생
```

- 충돌이 나면 4번의 순서대로 해결해본다.
- 해결 후 `git log --oneline --graph`로 merge 커밋을 확인한다.

## 4. 충돌 해결 과정

### 4-1. 충돌 발생

```bash
git merge feature
```

```text
Auto-merging a.txt
CONFLICT (content): Merge conflict in a.txt
Automatic merge failed; fix conflicts and then commit the result.
```

- merge가 중간에 멈춘 상태(merge 진행 중)가 된다.

### 4-2. 상태 확인

```bash
git status
```

- `Unmerged paths` 아래에 `both modified: a.txt`처럼 충돌 파일이 표시된다.

### 4-3. 충돌 표시 읽기

```text
<<<<<<< HEAD
main의 내용 (current)
=======
feature의 내용 (incoming)
>>>>>>> feature
```

- `<<<<<<< HEAD` ~ `=======`: 현재 브랜치(current)의 내용
- `=======` ~ `>>>>>>> feature`: 합쳐 들어오는 브랜치(incoming)의 내용

### 4-4. 직접 수정

- 선택지: current만 / incoming만 / 둘 다 / 새로 작성
- 최종 내용만 남기고 `<<<<<<<`, `=======`, `>>>>>>>` 표시를 **모두 지운다.**
- VS Code에서는 충돌 영역 위에 버튼이 표시된다: Accept Current Change, Accept Incoming Change, Accept Both Changes, Compare Changes
- 파일 전체를 한쪽으로 정할 때 (명령어 옵션 이름은 `--ours`=current, `--theirs`=incoming)

```bash
git checkout --ours a.txt     # current 버전으로
git checkout --theirs a.txt   # incoming 버전으로
```

### 4-5. 남은 표시 확인

```bash
git diff --check
```

- 충돌 표시가 남아 있으면 경고한다.

### 4-6. add 후 commit

```bash
git add a.txt        # 해결 완료 표시
git status           # 모든 충돌이 해결되었는지 확인
git commit           # merge 커밋 생성 (메시지는 자동 생성)
```

### 4-7. 중단하고 싶을 때

```bash
git merge --abort    # merge 이전 상태로 되돌림
```

### 전체 흐름

```text
merge 실행 → 충돌 발생 → status로 파일 확인 → 파일 수정(표시 제거) → add → commit
                                  └─ 포기: merge --abort
```

## 5. 충돌 예방

- 작업을 시작하기 전에 최신 상태로 맞춘다. (`git pull`)
- 브랜치를 오래 끌지 않는다. 오래 분기될수록 충돌이 커진다.
- 기준 브랜치(develop 등)의 변경을 자주 feature 브랜치로 가져와 맞춘다.
- 커밋과 PR을 작게 나눈다.
- 같은 파일을 동시에 수정하지 않도록 팀원과 작업 범위를 나눈다.
- 전체 포맷 변경 같은 대규모 수정은 따로, 미리 공유하고 진행한다.
- PR에서도 충돌이 있으면 표시된다. 가능하면 PR 전에 로컬에서 먼저 해결한다.
