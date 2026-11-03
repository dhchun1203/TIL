## 1. 오늘의 흐름

1. **딕셔너리:** 키로 값을 찾는다. 조회, 수정, 추가, 삭제.
2. **딕셔너리 메서드:** `keys()`, `values()`, `items()`, `get()`.
3. **집합(Set):** 중복 없는 값의 모음. 추가, 삭제, 집합 연산.
4. **중복 제거:** 집합과 딕셔너리로 리스트의 중복을 없앤다.
5. **제어문:** `if`로 갈라지고, `for`와 `while`로 반복하고, `break`와 `continue`로 흐름을 끊는다.
6. **종합 실습:** 회원 데이터 필터링, 숫자 맞추기 게임.

## 2. 딕셔너리 (Dictionary)

### 기본 규칙

- `{"키": "값"}` 형태
- 키는 중복 불가
- 값은 중복 가능
- 추가하는 순서를 유지

```python
print({"a":1, "b":1})
```

- 출력 `{'a': 1, 'b': 1}`. 값 `1`이 두 번 들어가도 된다.
- 직접 실행하니 `{"a": 1, "a": 2}`는 `{'a': 2}`가 된다. 같은 키를 다시 쓰면 뒤의 값이 남는다.
- 순서 유지: 파이썬 3.7부터 보장된다. 3.6에서는 CPython 구현상 유지되었지만 의존하면 안 되는 구현 세부사항이었다.

### 조회, 수정, 추가

```python
user_info = {
    "name": "LeeJaeHyun",
    "age": 28,
}

print(user_info)

user_info["age"] # 키로 값을 가져오기
user_info["age"] = 30
print(user_info["age"])
user_info["role"] = "Tutor"
print(user_info)
```

```
{'name': 'LeeJaeHyun', 'age': 28}
30
{'name': 'LeeJaeHyun', 'age': 30, 'role': 'Tutor'}
```

- `user_info["age"]`: 키로 값을 가져온다.
- `user_info["age"] = 30`: 이미 있는 키에 넣으면 값이 바뀐다.
- `user_info["role"] = "Tutor"`: 없던 키에 넣으면 새로 추가된다. 같은 문법이다.
- 네 번째 줄 `user_info["age"]`는 `print`가 없어서 출력되지 않는다.

### 삭제

```python
user_info = {
    "name": "LeeJaeHyun",
    "age": 38,
    "role": "Tutor",
}
print(user_info)

# del 명령어 사용
del user_info["role"]
print(user_info)

# pop 사용 : 꺼내면서 삭제
age = user_info.pop("age")
print(age, user_info)
```

```
{'name': 'LeeJaeHyun', 'age': 38, 'role': 'Tutor'}
{'name': 'LeeJaeHyun', 'age': 38}
38 {'name': 'LeeJaeHyun'}
```

- `del user_info["role"]`: 키를 지정해서 삭제한다.
- `user_info.pop("age")`: 지우면서 그 값을 돌려준다. 변수 `age`에 `38`이 담겼다.

### 주요 메서드: keys(), values(), items()

```python
menu = {"coffee" : 4000, "tea" : 4500}

# Key 목록 조회
print(menu.keys())
# Value 목록 조회
print(menu.values())
# items 조회
print(menu.items())

# 딕셔너리는 인덱서 방식으로 값을 참조할 수 없다. > list 로 변환
print(list(menu.keys())[0])

print(sum(menu.values()))
```

```
dict_keys(['coffee', 'tea'])
dict_values([4000, 4500])
dict_items([('coffee', 4000), ('tea', 4500)])
coffee
8500
```

| 메서드 | 돌려주는 것 |
| --- | --- |
| `keys()` | 키 목록 |
| `values()` | 값 목록 |
| `items()` | (키, 값) 묶음 목록 |
- `list(menu.keys())[0]`: 키 목록을 리스트로 바꾸면 `[0]`처럼 위치로 꺼낼 수 있다. 출력 `coffee`.
- `sum(menu.values())`: 값을 모두 더한다. 4000 + 4500 = `8500`.
- 직접 실행하니 `menu[0]`은 `KeyError: 0`이 난다. 딕셔너리의 `[ ]`는 위치가 아니라 키를 받는다.

### in으로 확인하기

```python
print("coffee" in menu)
print(4000 in menu)
print(4000 in menu.values())
```

```
True
False
True
```

- 딕셔너리에 `in`을 쓰면 **키**를 찾는다. `4000`은 값이라 `4000 in menu`는 `False`.
- 값에서 찾으려면 `menu.values()`에 쓴다.

### get(): 없는 키를 안전하게 조회

```python
# menu["cake"] keyError 발생(없는 키 참조)
price = menu.get("cake", 5000)
print(price)

# default 값이 없는 경우 None
print(menu.get("cake"))
print(menu.get("coffee", 0))
print(menu)
```

```
5000
None
4000
{'coffee': 4000, 'tea': 4500}
```

- `menu["cake"]`는 없는 키라서 `KeyError`가 난다. 직접 실행해서 확인했다.
- `menu.get("cake", 5000)`: 없으면 두 번째 값 `5000`을 돌려준다.
- `menu.get("cake")`: 기본값을 안 주면 `None`.
- `menu.get("coffee", 0)`: 키가 있으면 기본값은 쓰이지 않고 `4000`.
- `get`을 써도 `menu`에 `cake`가 추가되지는 않는다. 마지막 출력에 `cake`가 없다.

## 3. 집합 (Set)

### 개념과 만들기


- 중괄호 사용
- `set()` 함수로 생성

```python
grade = set([1,2,2,3,3,3])

print(grade)
```

- 출력 `{1, 2, 3}`. 리스트의 중복이 사라진다.

```python
print(type({})) # {} 빈 딕셔너리
print(type(set()))
print({1,2,2})
```

```
<class 'dict'>
<class 'set'>
{1, 2}
```

- 빈 `{}`는 딕셔너리다. 빈 집합은 `set()`으로 만든다.
- `{1,2,2}`는 `{1, 2}`.

### 추가와 삭제

```python
s = {1, 2}

s.add(2)
s.add(5)

print(s)
s.remove(1) # 값이 없는 것을 삭제하려 하면 KeyError 가 발생한다.
print(s)

# discard : 값이 없는 것을 삭제하려고 해도 오류가 발생하지 않는다.
s.discard(20)
print(s)
```

```
{1, 2, 5}
{2, 5}
{2, 5}
```

- `add(2)`: 이미 있는 값이라 변화가 없다. `add(5)`는 추가된다.
- `remove(1)`: 지운다. 없는 값을 지우면 `KeyError`가 난다. 직접 실행해서 `remove(20)`이 `KeyError: 20`을 내는 것을 확인했다.
- `discard(20)`: 없는 값을 지워도 에러가 없다.

### 집합 연산

```python
set_a = {1,2,3}
set_b = {3,4,5}

print(set_a & set_b) # 교집합
print(set_a | set_b) # 합집합
print(set_a - set_b) # 차집합
print(3 in set_a) # 포함 여부
```

| 식 | 의미 | 결과 |
| --- | --- | --- |
| `set_a & set_b` | 교집합 | `{3}` |
| `set_a | set_b` | 합집합 | `{1, 2, 3, 4, 5}` |
| `set_a - set_b` | 차집합 | `{1, 2}` |
| `3 in set_a` | 포함 여부 | `True` |

## 4. 중복 제거

```python
raw_emails = ["a@sparta.com", "b@sparta.com", "a@sparta.com"]

# 집합으로 중복 제거
unique_set = set(raw_emails)
print(list(unique_set))

# 중복제거 및 순서 유지
sort_emails = list(dict.fromkeys(raw_emails))
print(sort_emails)

# sorted
print(sorted(set(["c", "d", "a", "f", "a"])))
```

```
['b@sparta.com', 'a@sparta.com']
['a@sparta.com', 'b@sparta.com']
['a', 'c', 'd', 'f']
```

| 방법 | 중복 | 순서 |
| --- | --- | --- |
| `list(set(리스트))` | 제거 | 저장된 출력에서 원래 순서와 달라짐 (`b`가 먼저) |
| `list(dict.fromkeys(리스트))` | 제거 | 처음 나온 순서 유지 (`a`, `b`) |
| `sorted(set(리스트))` | 제거 | 정렬됨 (`a`, `c`, `d`, `f`) |
- `dict.fromkeys(리스트)`: 리스트의 값을 키로 하는 딕셔너리를 만든다. 키는 중복될 수 없고 넣은 순서가 유지되어서 중복 제거와 순서 유지가 함께 된다.
- `sorted(...)`: 정렬된 리스트를 돌려준다.
- 직접 실행하니 `len(set(raw_emails))`는 `2`, `len(raw_emails)`는 `3`.

## 5. 제어문 (chapter05)

### if / elif / else


```python
if 조건식:
    실행할 코드 #1
elif 조건식2:
    실행할 코드 #2
else:
    실행할 코드 #3
```

```python
temp = 30

if temp >= 28:
    print("에어컨을 켭니다.")
    print("1시간 후에 자동 종료됩니다.")
print("프로그램 종료")
```

- 들여쓰기한 두 줄은 조건이 참일 때만 실행된다. 마지막 `print`는 들여쓰기가 없어서 조건과 상관없이 실행된다.
- 직접 실행하니 `에어컨을 켭니다.`, `1시간 후에 자동 종료됩니다.`, `프로그램 종료` 순서로 세 줄이 나온다.

```python
cart = ["카메라", "삼각대"]
if cart :
    print("결제를 진행합니다.")
else:
    print("장바구니가 비었습니다.")
```

- 조건식 자리에 리스트를 그대로 쓴다. `cart`에 값이 들어 있어서 `결제를 진행합니다.`가 출력된다.

```python
if True:
    pass
    # TODO: 쇼핑 카트 로직 추가 예정
```

- `pass`: 블록 안에 코드가 아직 없을 때 자리를 채운다. 출력은 없다.

```python
age = 20
message = "성인" if age >= 20 else "미성년자"
print(message)
```

- 한 줄 표기: `값1 if 조건 else 값2`. 출력 `성인`.

### 조건 순서

```python
score = 95

if score >= 80:
    print("B")
elif score >= 90:
    print("A")
    
if score >= 90:
    print("A")
elif score >= 80:
    print("B")
```

```
B
A
```

- 첫 번째 묶음은 `score >= 80`이 먼저라서 95점인데 `B`가 나온다. `elif score >= 90`은 검사되지 않는다.
- 두 번째 묶음은 `90`을 먼저 검사해서 `A`가 나온다.
- 범위 조건은 좁은 쪽(큰 값)부터 쓴다.

```python
is_login = True
has_permission = True
if is_login and has_permission:
    print("관리자 페이지 이동")

x = 10
if x == 10:
    print(f"x는 {x} 입니다.")
```

```
관리자 페이지 이동
x는 10 입니다.
```

- `and`: 양쪽이 모두 참일 때 참.

### for 반복문

```python
for 변수 in 반복할_데이터:
    반복수행할 코드 #1
```

```python
fruits = ["apple", "banana", "체리"]
for fruit in fruits:
    print(fruit)

for ch in "스파르타":
    print(ch)

menu = {"아아" : 4000, "홍차" : 4500}
for key in menu.keys():
    print(key)
    
for name, price in menu.items():
    print(f"{name} : {price}")
```

```
apple
banana
체리
스
파
르
타
아아
홍차
아아 : 4000
홍차 : 4500
```

- 리스트는 요소를 하나씩, 문자열은 글자를 하나씩 꺼낸다.
- 딕셔너리에 `keys()`를 쓰면 키를, `items()`를 쓰면 (키, 값)을 `name, price`로 풀어서 받는다.

```python
for i in range(5):
    print(f"{i+1} 번째 로직 수행")

for i in range(1, 6, 3):
    print(i)

print(range(5))
print(list(range(5))) # 리스트로 변환 후 출력
```

```
1 번째 로직 수행
2 번째 로직 수행
3 번째 로직 수행
4 번째 로직 수행
5 번째 로직 수행
1
4
range(0, 5)
[0, 1, 2, 3, 4]
```

| 식 | 만들어지는 값 |
| --- | --- |
| `range(5)` | 0, 1, 2, 3, 4 |
| `range(1, 6, 3)` | 1, 4 (시작, 끝 직전까지, 간격) |
- `print(range(5))`는 `range(0, 5)`로 나온다. 값을 보려면 `list(range(5))`로 바꾼다.

### while 반복문

```python
while 조건식:
    반복할 로직 #1
```

```python
num = 1
while num <= 3:
    print(f"현재 숫자는 {num}")
    num += 1

count = 3
while count > 0:
    print(count)
    count -= 1
```

```
현재 숫자는 1
현재 숫자는 2
현재 숫자는 3
3
2
1
```

- 조건이 참인 동안 반복한다. 반복 안에서 `num += 1`, `count -= 1`로 조건을 바꿔야 끝난다.

### break와 continue

```python
for num in range(1,6):
    if num == 3:
        print("즉시 종료")
        break
    print(f"현재 num={num}")
```

```
현재 num=1
현재 num=2
즉시 종료
```

```python
for num in range(1, 6):
    if num % 2 == 0: # 짝수
        continue
    print(f"홀수 {num}")
```

```
홀수 1
홀수 3
홀수 5
```

| 키워드 | 동작 |
| --- | --- |
| `break` | 반복문을 끝낸다 |
| `continue` | 이번 차례만 건너뛰고 다음 차례로 간다 |

### 중첩 반복: 구구단

```python
for x in range(2, 10):
    if x == 4:
        break
    print(f"---{x} 단---")
    for y in range(1, 10):
        if y == 6:
            break
        print(f"{x} X {y} = {x * y}")
```

```
---2 단---
2 X 1 = 2 ... 2 X 5 = 10
---3 단---
3 X 1 = 3 ... 3 X 5 = 15
```

- 바깥 `for`가 단, 안쪽 `for`가 곱하는 수.
- 안쪽 `break`(`y == 6`)는 안쪽 반복만 끝낸다. 각 단은 5까지만 나온다.
- 바깥 `break`(`x == 4`)가 바깥 반복을 끝내서 2단과 3단만 나온다.
- 이 셀은 마지막 커밋(`5cfb625`)에서 `if y == 6: break` 두 줄이 추가되었고, 출력도 그에 맞게 `5`까지로 바뀌었다. 이전 커밋에서는 `9`까지 나왔다.

## 6. 종합 실습 (chapter06)

### 회원 데이터 필터링

딕셔너리를 담은 리스트를 `for`로 돌면서 `if`로 거른다.

```python
user_database = [
    {"name" : "손흥민", "level" : "VIP", "active" : True, "point" : 15000},
    {"name" : "김민재", "level" : "Gold", "active" : True, "point" : 9000},
    {"name" : "조규성", "level" : "VIP", "active" : True, "point" : 12000},
    {"name" : "이강인", "level" : "Gold", "active" : True, "point" : 8000},
    {"name" : "홍명보", "level" : "Bronze", "active" : False, "point" : 1000},
]

total_vip_points = 0
active_user_count = 0

print("=== 마케팅 대상 활성 VIP 및 Gold 회원 결과")

for user in user_database:
    if user['active'] == True:
        active_user_count += 1

        if user['level'] == "VIP":
            total_vip_points += user['point']
            print(f'[VIP 등급] {user["name"]} 회원님에게 VIP 쿠폰이 발행되었습니다.')
        elif user['level'] == "Gold":
            print(f'[Gold 등급] {user["name"]} 회원님에게 Gold 쿠폰이 발행되었습니다.')
print(f"전체 활성 회원 수    : {active_user_count}")
print(f"활성 VIP 포인트 누적 : {total_vip_points:,}")
```

```
=== 마케팅 대상 활성 VIP 및 Gold 회원 결과
[VIP 등급] 손흥민 회원님에게 VIP 쿠폰이 발행되었습니다.
[Gold 등급] 김민재 회원님에게 Gold 쿠폰이 발행되었습니다.
[VIP 등급] 조규성 회원님에게 VIP 쿠폰이 발행되었습니다.
[Gold 등급] 이강인 회원님에게 Gold 쿠폰이 발행되었습니다.
전체 활성 회원 수    : 4
활성 VIP 포인트 누적 : 27,000
```

- `user['active'] == True`로 활성 회원만 통과시킨다. `홍명보`는 `False`라서 빠진다.
- `if` 안에 `if`를 넣어(중첩) 활성 회원 중에서 등급을 나눈다. Bronze는 어느 쪽에도 해당하지 않아 쿠폰 출력이 없다.
- 누적 변수 패턴: 반복 전에 `0`으로 만들고, 반복 안에서 `+=`로 더한다. VIP 15000 + 12000 = `27000`.
- `{total_vip_points:,}`: 천 단위 쉼표를 붙인다. `27,000`.
- 직접 실행한 결과가 저장된 출력과 같다.

### 숫자 맞추기 게임

```python
import random # 난수 발생을 위한 모듈 임포트

number = random.randint(1,100)
comput_number = number
try_count = 0

print("="*39)
print("컴퓨터가 1부터 100사이의 숫자를 생성했습니다.")
print("="*39)

while True:
    user_input = input("컴퓨터에 숫자를 입력해보세요. :" )

    if user_input == "q":
        print(f"게임 종료합니다. 정답은 {number} 입니다.")
        break

    if not user_input.isdigit() and int(user_input) > 0:
        print("숫자로 입력해주세요.")
        continue

    if int(user_input) <= 1 or int(user_input) >= 100:
        print(f"입력값 : {user_input} => 1 에서 100 사이의 값을 입력해주세요.")
        continue

    user_number = int(user_input)
    try_count += 1

    if user_number == comput_number:
        print(f"{user_input} : 정답입니다. {try_count} 회 만에 성공했습니다.")
        break
    elif user_number >= comput_number:
        print(f"{user_number} : 더 작은 수 입니다.")
    else:
        print(f"{user_number} : 더 큰 수 입니다." )
```

흐름:

1. `random.randint(1,100)`으로 정답을 정한다.
2. `while True`로 계속 입력을 받는다.
3. `q`면 정답을 알려주고 `break`.
4. 범위 밖이면 안내하고 `continue`. 시도 횟수는 올라가지 않는다.
5. 범위 안이면 `try_count`를 올리고 정답과 비교한다. 맞으면 `break`, 크면 `더 작은 수`, 작으면 `더 큰 수`.


- `1`과 `100`은 범위 안내가 나오고 거부된다. 조건이 `<= 1`, `>= 100`이라서다. 안내문은 `1 에서 100 사이`라고 되어 있다.
- `-10`은 `isdigit()`이 `False`이지만 `int(user_input) > 0`이 `False`라서 숫자 안내 조건을 지나, 범위 안내가 나온다.
- `abc`나 `1.5`를 입력하면 `int(user_input)`에서 `ValueError: invalid literal for int() with base 10`이 난다. `숫자로 입력해주세요.` 메시지는 나오지 않는다.
- `2`, `3`, `8`을 순서대로 입력하면 `더 큰 수`가 두 번 나오고 `3 회 만에 성공했습니다.`로 끝난다.