## 1. 오늘의 흐름

1. **함수:** 정의, 기본값, 가변 인수, `return`.
2. **변수 스코프:** 지역 변수, 전역 변수, `global`.
3. **람다식:** `lambda`와 `map`, `filter`, `sorted`, `max`.
4. **미니 프로젝트:** 가위바위보 게임(`main.py`).

## 2. 함수 (Function)

### 기본 규칙

- 반복되는 로직을 합쳐놓은 문법
- 재사용성
- 단독 사용 가능: `print()`, `len()`
- 함수명 컨벤션: snake_case
- 매개변수 (parameter)

```py
def 함수이름(매개변수):
    실행할 로직 #1
    실행할 로직 #2
    실행할 로직 #3
```

### 정의와 기본값

```py
def say_hello(name = "홍길동", age  = 30, point = 1000):
    print(f"{name}님, 안녕하세요. 나이는 {age}입니다.")
    print(f"초기 지급 포인트 {point}점이 지급되었습니다.")

say_hello()
```

```txt
홍길동님, 안녕하세요. 나이는 30입니다.
초기 지급 포인트 1000점이 지급되었습니다.
```

- `name = "홍길동"`처럼 매개변수에 값을 적어 두면 기본값이 된다.
- `say_hello()`는 인수를 하나도 안 넘겼기 때문에 기본값 세 개가 모두 쓰인다.

### 가변 인수: *numbers

```py
def add_all(*numbers): # 매개변수 (Parameter)
    print(f"입력된 튜플: {numbers}")
    total = sum(numbers)
    print(f"합계 결과: {total}")

add_all(1,2,3) # 인수 (Argument)
add_all(1,2,3,4,5)
add_all()
```

```txt
입력된 튜플: (1, 2, 3)
합계 결과: 6
입력된 튜플: (1, 2, 3, 4, 5)
합계 결과: 15
입력된 튜플: ()
합계 결과: 0
```

| 용어 | 위치 | 예 |
| --- | --- | --- |
| 매개변수 (parameter) | 함수를 정의하는 쪽 | `*numbers` |
| 인수 (argument) | 함수를 호출하는 쪽 | `1,2,3` |

- `*numbers`: 넘긴 인수가 몇 개든 하나의 튜플로 묶어서 받는다.
- 인수를 안 넘기면 빈 튜플 `()`이고, `sum(())`은 `0`이다.

### 딕셔너리 가변 인수: **kwargs

```py
def user_info(**data):
    print(f"전달된 딕셔너리 구조: {data}")
    for key, value in data.items():
        print(f"{key}:{value}")

user_info(name = "스파르타", level = 5, active = True)
```

```txt
전달된 딕셔너리 구조: {'name': '스파르타', 'level': 5, 'active': True}
name:스파르타
level:5
active:True
```

- `**data`: `이름 = 값` 형태로 넘긴 인수를 딕셔너리로 묶어서 받는다.
- 받은 딕셔너리는 어제 배운 `items()`로 반복할 수 있다.

### return: 결과값 되돌리기

```py
def check_ord_number(number):
    if number > 0:
        return '양수'
    else:
        return '음수'

print(check_ord_number(10))
```

- 출력 `양수`.
- `return`은 값을 함수 밖으로 돌려주고 함수를 끝낸다.
- 직접 실행하니 `check_ord_number(0)`은 `음수`다. `0 > 0`이 거짓이라 `else`로 간다.

### 여러 개 반환

```py
def calc_value(a, b):
    plus = a + b
    minus = a - b
    return plus, minus

result1, result2 = calc_value(1,2)
print(f"덧셈: {result1}, 뺄셈: {result2}")

result = calc_value(10, 5)
print(f"튜플: {result}")
type(result)
```

```txt
덧셈: 3, 뺄셈: -1
튜플: (15, 5)
tuple
```

- `return plus, minus`: 쉼표로 이어 쓰면 튜플 하나로 돌아온다.
- `result1, result2 = ...`: 튜플을 두 변수에 나누어 받는다.
- `result = ...`: 변수 하나로 받으면 튜플 그대로다. `type(result)`는 `tuple`.

### 평균을 계산해서 반환하는 함수

```py
def get_average(*args):
    if len(args) == 0:
        return 0

    # if not args:
    #     return 0

    num_list = list(args)
    sum_num = sum(num_list)
    len_num = len(num_list)
    avg_num = sum_num / len_num
    return avg_num

result1 = get_average(80, 90, 100)
result2 = get_average(75, 80, 92, 100, 55)
result3 = get_average()

print(result1)
print(result2)
print(result3)
```

```txt
90.0
80.4
0
```

- `len(args) == 0`: 인수가 없으면 `0`을 돌려주고 끝낸다. 0으로 나누는 오류를 막는다.
- 주석 처리된 `if not args:`도 같은 역할이다. 직접 실행하니 두 방식 모두 인수가 없을 때 `0`이다.
- `/` 나눗셈의 결과는 소수다. 그래서 `90.0`이고, 인수가 없을 때만 정수 `0`이다.

### 미니 계산기

```py
def add(a, b):
    return a + b

def minus(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        return "0 으로 나눌 수 없습니다."

    return a / b

a = int(input("첫 번째 숫자: "))
b = int(input("두 번째 숫자: "))

print(f"덧셈 연산 {a} + {b}: {add(a, b)}")
print(f"뺄셈 연산 {a} - {b}: {minus(a, b)}")
print(f"곱셈 연산 {a} * {b}: {multiply(a, b)}")
print(f"나눗셈 연산 {a} / {b}: {divide(a, b)}")
```

```txt
덧셈 연산 4 + 0: 4
뺄셈 연산 4 - 0: 4
곱셈 연산 4 * 0: 0
나눗셈 연산 4 / 0: 0 으로 나눌 수 없습니다.
```

- 사칙 연산을 함수 네 개로 나누고, 마지막에 같은 입력으로 네 번 호출한다.
- `divide`는 `b`가 `0`이면 계산하지 않고 문구를 돌려준다.
- 저장된 출력은 `4`와 `0`을 입력한 결과다. 같은 값으로 실행해 출력이 같은 것을 확인했다.

## 3. 변수 스코프 (유효범위)

- 전역 변수: 함수 밖에서 선언하는 변수
- 지역 변수: 함수 안에서 선언하는 변수, 함수 안에서만 접근 가능

정리:

- 전역 변수는 함수 안에서도 읽을 수 있다.
- 함수 안에서 `변수 = 값`을 쓰면 전역 변수가 바뀌는 게 아니라 같은 이름의 지역 변수가 새로 생긴다.
- 함수 안에서 전역 변수의 값을 바꾸려면 `global`로 알려야 한다.

### 지역 변수

```py
def local():
    local_var = 1
    print(local_var)

local()
# print(local_var)
```

- 함수 안의 `local_var`는 함수 안에서만 쓸 수 있다.
- 직접 실행하니 `local()`은 `1`을 출력한다.
- 주석 처리된 `print(local_var)`를 풀고 실행하면 `NameError: name 'local_var' is not defined`가 난다.

### 같은 이름의 전역 변수와 지역 변수

```py
message = "안녕하세요. Kant 몰에 오신것을 환영합니다."

def greeting():
    message = "장난감 코너는 3층에 있습니다."
    print(message)

greeting()
print(message)
```

```txt
장난감 코너는 3층에 있습니다.
안녕하세요. Kant 몰에 오신것을 환영합니다.
```

- 함수 안의 `message`는 지역 변수라서 함수 안에서는 지역 값이 쓰인다.
- 함수 밖 `message`는 그대로다.

### 중첩 함수

```py
value = "전역 데이터"

def outer_function():
    value = "지역 변수 1"

    def inner_function():
        value = "지역 변수 2"
        print(value)

    inner_function()
    print(value)

outer_function()
print(value)
```

```txt
지역 변수 2
지역 변수 1
전역 데이터
```

| 위치 | `value` | 출력 순서 |
| --- | --- | --- |
| `inner_function` 안 | `"지역 변수 2"` | 1번째 |
| `outer_function` 안 | `"지역 변수 1"` | 2번째 |
| 함수 밖 | `"전역 데이터"` | 3번째 |

- 함수마다 자기 `value`가 따로 있어서 서로 영향을 주지 않는다.

### global 키워드

```py
counter = 0

def inc_counter():
    global counter # 전역 변수를 참조하겠다고 선언
    counter += 1

inc_counter()
inc_counter()
print(counter)
```

- 출력 `2`.
- `global counter`를 쓰면 함수 안의 `counter`가 전역 변수 `counter`를 가리킨다. 그래서 `+= 1`이 전역 값을 바꾼다.
- `global` 없이 함수 안에서 `counter += 1`만 쓰면 `UnboundLocalError`가 난다. 직접 실행해서 확인했다.

## 4. 람다식 (Lambda Expression)

```py
lambda 매개변수 : 표현식
```

### 일반 함수와 비교

```py
def add_five(x):
    return x + 5

print(add_five(10))

result = lambda x: x + 5
print(result(10))
```

```txt
15
15
```

- 이름 없는 한 줄짜리 함수다. `lambda x: x + 5`와 `def add_five(x): return x + 5`는 같은 일을 한다.
- 식 한 줄만 쓸 수 있고, 식의 결과가 그대로 반환값이 된다.

### map과 filter

```py
numbers = [1,2,3,4,5]

result = list(map(lambda x : x + 10, numbers))
print(result)

# 리스트 컴프리헨션
print([x + 10 for x in numbers])

filter_result = list(filter(lambda x : x % 2 == 0, numbers))
print(filter_result)

print([x for x in numbers if x % 2 == 0])
```

```txt
[11, 12, 13, 14, 15]
[11, 12, 13, 14, 15]
[2, 4]
[2, 4]
```

| 하는 일 | 람다 방식 | 리스트 컴프리헨션 |
| --- | --- | --- |
| 모든 요소를 바꾸기 | `list(map(lambda x : x + 10, numbers))` | `[x + 10 for x in numbers]` |
| 조건에 맞는 요소만 | `list(filter(lambda x : x % 2 == 0, numbers))` | `[x for x in numbers if x % 2 == 0]` |

- `map(함수, 데이터)`: 모든 요소에 함수를 적용한다.
- `filter(함수, 데이터)`: 함수의 결과가 참인 요소만 남긴다.
- 둘 다 결과를 `list()`로 감싸야 값을 볼 수 있다.
- 같은 결과를 리스트 컴프리헨션으로도 만들 수 있다. 출력이 서로 같다.

### 정렬: sorted, max

```py
players = [("손흥민", 95),("김민재",60),("이강인",80)]

print(sorted(players, key = lambda p : p[1] )) # 오름차순
print(sorted(players, key = lambda p : p[1], reverse=True )) # 내림차순
print(max(players, key = lambda p: p[1])) # 최고 점수 추출
```

```txt
[('김민재', 60), ('이강인', 80), ('손흥민', 95)]
[('손흥민', 95), ('이강인', 80), ('김민재', 60)]
('손흥민', 95)
```

- `key = lambda p : p[1]`: 각 항목에서 `[1]`번째(점수)를 기준으로 삼는다.
- `reverse=True`: 내림차순.
- `max(..., key = ...)`: 기준 값이 가장 큰 항목을 돌려준다.

### 연습: 거듭제곱 후 20 이상만

```py
numbers = [2,3,4,5,6,7]

# 람다식으로 모든 요소의 거듭제곱
sqr_number = list(map(lambda num: num**2 , numbers))
print(sqr_number)

# 결과값이 20 이상인 값만 추출
sqr_number_over_20 = list(filter(lambda num: num >= 20, sqr_number))
print(sqr_number_over_20)
```

```txt
[4, 9, 16, 25, 36, 49]
[25, 36, 49]
```

- `map`으로 제곱한 리스트를 만들고, 그 결과에 `filter`를 한 번 더 적용한다.

## 5. 미니 프로젝트: 가위바위보 게임

`main.py`에 작성한 3판 2선제 가위바위보. 먼저 2번 이기거나 2번 지면 끝난다.

```py
import random
# 가위 바위 보 게임 미니 프로젝트


print("=" * 30)
print("가위바위보 게임")
print("=" * 30)

# 플레이어 이름 입력
player_name = input("플레이어 이름을 입력하세요 : ")

game_data = {"1":"가위", "2":"바위", "3":"보"}

retry_value = "" # 재도전 여부
win_cnt = 0 # 승리 카운터
winning_streak_cnt = 0 # 연승 카운터
lose_cnt = 0 # 패배 카운터
game_record_list = [] # 경기 기록 리스트


while retry_value == "yes" or retry_value == "":
    target_num = str(random.randint(1,3))

    print("1. 가위 / 2. 바위 / 3. 보")
    input_num = input("번호를 선택하세요. : ")
    while int(input_num) < 1 or int(input_num) > 3:
        print("1 부터 3 까지의 숫자만 입력 가능합니다.")
        input_num = input("번호를 선택하세요. : ")


    print()
    print("[게임 결과]")
    print(f"{player_name}: {game_data[input_num]}  vs  컴퓨터: {game_data[target_num]}")

    game_result = ""

    if game_data[input_num] == "가위": # 내가 가위를 낸 경우의 수
        if game_data[target_num] == "보":
            game_result = "승리"
            win_cnt += 1
            winning_streak_cnt +=1
        elif game_data[target_num] == "바위":
            game_result = "패배"
            lose_cnt += 1
            winning_streak_cnt = 0
        else:
            game_result = "비김"
            winning_streak_cnt = 0
    elif game_data[input_num] == "바위": # 내가 바위를 낸 경우의 수
        if game_data[target_num] == "가위":
            game_result = "승리"
            win_cnt += 1
            winning_streak_cnt += 1
        elif game_data[target_num] == "보":
            game_result = "패배"
            lose_cnt += 1
            winning_streak_cnt = 0
        else:
            game_result = "비김"
            winning_streak_cnt = 0
    else: # 내가 보를 낸 경우의 수
        if game_data[target_num] == "바위":
            game_result = "승리"
            win_cnt += 1
            winning_streak_cnt += 1
        elif game_data[target_num] == "가위":
            game_result = "패배"
            lose_cnt += 1
            winning_streak_cnt = 0
        else:
            game_result = "비김"
            winning_streak_cnt = 0
    print(f'=> {game_result}')

    game_record_list.append(game_result)

    print()

    if win_cnt == 2 or lose_cnt == 2: # 3판 2선제 적용 및 승패 목록 출력
        print(f"최종 스코어\t{player_name} {win_cnt} : {lose_cnt} 컴퓨터")
        print(f"승패 목록 : {game_record_list}")
        break

    print(f"현재 스코어\t{player_name} {win_cnt} : {lose_cnt} 컴퓨터")
    print(f"현재 연승 카운트 : {winning_streak_cnt}")
    print()

    retry_value = input("다시 도전 (yes/no) : ")

    if retry_value == "no":
        print("bye bye")
        print(f"승패 목록 : {game_record_list}")

    while retry_value != "yes" and retry_value != "no":
        print("yes 혹은 no 만 입력해주세요.")
        retry_value = input("다시 도전 (yes/no) : ")
```

### 쓰인 것

| 변수 | 역할 |
| --- | --- |
| `game_data` | 입력 번호(`"1"`, `"2"`, `"3"`)를 `가위`, `바위`, `보`로 바꾸는 딕셔너리 |
| `win_cnt`, `lose_cnt` | 승리, 패배 횟수 |
| `winning_streak_cnt` | 연승 횟수. 이기면 +1, 지거나 비기면 0 |
| `game_record_list` | `승리`, `패배`, `비김`을 순서대로 담는 리스트 |
| `retry_value` | 다시 도전할지 입력받은 값 |

### 흐름

1. 이름을 입력받는다.
2. `random.randint(1,3)`으로 컴퓨터의 번호를 정한다. 딕셔너리의 키가 문자열이라 `str()`로 바꾼다.
3. 번호를 입력받는다. 1~3이 아니면 안내 후 다시 입력받는다(안쪽 `while`).
4. 내가 낸 것(`if`)과 컴퓨터가 낸 것(안쪽 `if`)으로 승리, 패배, 비김을 가른다.
5. 결과를 기록 리스트에 넣고, 승리나 패배가 2번이 되면 최종 스코어와 승패 목록을 출력하고 `break`.
6. 아니면 현재 스코어와 연승 카운트를 출력하고 다시 도전할지 묻는다.
7. `no`면 `bye bye`와 승패 목록을 출력하고 끝난다. `yes`면 처음 `while`로 돌아간다.

