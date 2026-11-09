## 1. 오늘의 흐름

가위바위보 게임(3판 2선승제)을 함수로 나눠 `rps_game.py`에 작성했다. 뒤쪽(6, 7번)에는 `class_study.ipynb`에서 배운 상속과 캡슐화를 정리했다.

1. **상수로 빼기:** 손 이름 `HANDS`, 종료 조건 `GOAL_WINS`.
2. **역할별 함수:** 입력, 판정, 출력, 게임 루프.
3. **입력 검증:** 올바른 값이 들어올 때까지 반복.
4. **연승 기록:** 게임이 끝날 때마다 누적.

## 2. 함수 구조

| 함수 | 역할 |
| --- | --- |
| `show_title()` | 제목 출력 |
| `ask_name()` | 이름 입력(비우면 `게스트`) |
| `ask_number()` | 번호 입력(1~3만 허용) |
| `check_result()` | 승패 판정 |
| `show_result()` | 한 판의 결과 출력 |
| `show_score()` | 현재 스코어 출력 |
| `show_final_result()` | 최종 결과 출력 |
| `show_records()` | 판별 승패 목록 출력 |
| `ask_play_again()` | 다시 할지 입력(`yes`/`no`) |
| `play_game()` | 한 게임(3판 2선승제) 진행 |
| `main()` | 전체 흐름, 연승 기록 |

호출 관계:

```txt
main()
├─ show_title()
├─ ask_name()
├─ play_game(name)           # 게임 한 번
│   ├─ ask_number()
│   ├─ check_result()
│   ├─ show_result()
│   ├─ show_score()
│   ├─ show_final_result()
│   └─ show_records()
└─ ask_play_again()          # yes면 play_game으로 다시
```

- 함수 하나가 한 가지 일만 한다. 입력은 `ask_`, 출력은 `show_`로 이름을 붙였다.

## 3. 코드

```py
# 가위 바위 보 게임 (3판 2선승제)
import random

# 번호와 손 저장한 딕셔너리
HANDS = {1:"가위", 2:"바위", 3:"보"}

# 게임의 종료 조건
GOAL_WINS = 2

# 게임 타이틀 출력
def show_title():
    print("="*40)
    print("가위/바위/보 게임")
    print("="*40)

# 플레이어 이름 입력
def ask_name():
    name = input("플레이어 이름을 입력하세요. ").strip()
    if name == "":
        name = "게스트"

    return name

# 플레이어 번호 입력
def ask_number():
    while True:
        print("1. 가위 / 2. 바위 / 3. 보")
        answer = input("번호를 선택하세요. : " ).strip()

        if answer in ("1", "2", "3"):
            return int(answer)

        print("1, 2, 3 중에서 입력하세요.")


# 승패 판정
# 1 - 1 / 2 - 2 / 3 - 3 무승부 조건 판정
# 1 - 3 / 2 - 1 / 3 - 2 플레이어 승리 조건
def check_result(user_number, computer_number):
    if user_number == 1 and computer_number == 3: return "승리"
    if user_number == 2 and computer_number == 1: return "승리"
    if user_number == 3 and computer_number == 2: return "승리"

    return "패배"

# 한 판의 게임 결과 출력
def show_result(name, user_number, computer_number, result):
    print("\n[게임 결과]")
    print(f"{name} : {HANDS[user_number]} VS 컴퓨터: {HANDS[computer_number]}")
    print(f"=> {result}")

# 현재 스코어 표시
def show_score(name, my_wins, computer_wins):
    print(f"\n현재 스코어 {name} {my_wins} : {computer_wins} 컴퓨터\n") # 게스트 1 : 0 컴퓨터

# 최종 결과 표시
def show_final_result(name, my_wins, computer_wins):
    print("-" * 40)
    print(f"[최종 결과] {name} {my_wins} : {computer_wins} 컴퓨터")
    if my_wins > computer_wins:
        print(f"==> {name} 최종 승리!")
    else:
        print("==> 컴퓨터 승리!")

# 승패 판결 목록 표시
def show_records(records):
    print("\n[판별 승패 목록]")
    # "1판 : 가위 vs 바위 => 패배"
    for number, record in enumerate(records, start=1):
        print(f"{number}판 : {record}")

# 게임 재시작 여부
def ask_play_again():
    while True:
        answer = input("\n다시 도전 (yes/no) : ").strip().lower()
        if answer == "yes" or answer == "y":
            return True
        if answer == "no" or answer == "n":
            return False

        print("yes 또는 no 로 입력해주세요.")


# 게임 루프 (3판 2선승제)
def play_game(name):
    my_wins = 0
    computer_wins = 0
    records = [] # 승패를 기록할 리스트

    # 둘 다 2승이 안됐을 때 다음 판을 진행
    while my_wins < GOAL_WINS and computer_wins < GOAL_WINS:
        user_number = ask_number()
        computer_number = random.randint(1, 3) # 1, 2, 3

        # 승패 판정
        result = check_result(user_number, computer_number)

        if result == "승리":
            my_wins += 1
        elif result == "패배":
            computer_wins += 1

        # 승패 결과 목록에 저장
        records.append(f"{HANDS[user_number]} vs {HANDS[computer_number]} => {result}")

        show_result(name, user_number, computer_number, result)
        show_score(name, my_wins, computer_wins)

    show_final_result(name, my_wins, computer_wins)
    show_records(records)
    # 플레이어가 승리하면 True
    return my_wins == GOAL_WINS

# 게임의 전체 루프
def main():
    show_title()
    name = ask_name()

    # 연승 기록을 위한 변수
    co_wins = 0

    while True:
        my_won = play_game(name)
        if my_won:
            co_wins += 1 # 연승 기록 누적
        else:
            co_wins = 0 # 연승 기록 초기화

        print(f"\n현재 연승: {co_wins} 연승") if co_wins > 0 else ""

        if not ask_play_again():
            break

    print("게임을 종료합니다.")


# 게임 시작
main()
```

## 4. 함수별 정리

### 상수: HANDS, GOAL_WINS

- `HANDS = {1:"가위", 2:"바위", 3:"보"}`: 번호를 키로, 손 모양을 값으로 한다. 키가 숫자라서 `ask_number()`가 `int(answer)`로 바꿔 돌려준다.
- `GOAL_WINS = 2`: 종료 조건을 이름 하나로 쓴다. 대문자 이름은 바꾸지 않는 상수라는 관례다.

### ask_name(): 이름 입력

- `.strip()`으로 앞뒤 공백을 지우고, 빈 값이면 `게스트`로 바꾼다. 직접 실행해 공백만 입력하니 `게스트`가 나왔다.

### ask_number(): 번호 입력

- `while True`로 올바른 값이 들어올 때까지 묻는다. 문자열로 먼저 검사한 뒤 `int()`로 바꾸기 때문에, 직접 실행하니 `a`나 `5`를 입력해도 오류 없이 다시 물었다.

### check_result(): 승패 판정

| 내가 낸 것 | 컴퓨터가 낸 것 | 결과 |
| --- | --- | --- |
| 1 (가위) | 3 (보) | 승리 |
| 2 (바위) | 1 (가위) | 승리 |
| 3 (보) | 2 (바위) | 승리 |
| 그 외 |  | 패배 |

- 이긴 3가지만 적고 나머지는 모두 `패배`다. 직접 실행하니 같은 손(가위 vs 가위)도 `패배`로 처리됐다. 주석에는 무승부가 있지만 함수는 두 값만 돌려준다.

### 출력 함수: show_*

- `show_result()`, `show_score()`, `show_final_result()`: 결과·스코어·최종 결과 출력. `HANDS[번호]`로 손 이름을 꺼낸다.
- `show_records()`: `enumerate(records, start=1)`가 (번호, 값) 묶음을 돌려주고, `number, record`로 풀어서 받는다.

### ask_play_again(): 다시 할지

- `.strip().lower()`로 `Y`, `YES`도 통과시킨다. `yes`/`y`는 `True`, `no`/`n`은 `False`, 그 외는 안내 후 다시 묻는다. 직접 실행해 확인했다.

### play_game(): 한 게임

```py
while my_wins < GOAL_WINS and computer_wins < GOAL_WINS:
```

- 한 판: 번호 입력 → 컴퓨터 번호(`random.randint(1, 3)`) → `check_result()` → 점수 → 기록 → 출력. 한쪽이 2승이 되면 루프가 끝난다.
- `return my_wins == GOAL_WINS`: 이기면 `True`. `main()`이 이 값으로 연승을 계산한다.

### main(): 연승 기록

- `co_wins`: 이기면 +1, 지면 0으로 초기화하는 연승 횟수다. `print(...) if co_wins > 0 else ""`는 한 줄 조건 표현식이다. 직접 실행하니 지면 연승 줄이 출력되지 않았다.
- 파일 맨 아래에서 `main()`을 바로 호출하므로 `import rps_game`만 해도 게임이 시작된다. 직접 실행해 확인했다.

## 5. 실행 예시

입력과 난수를 정해서 실행한 결과다(이름 `대환`, 가위를 두 번 내고 컴퓨터가 보를 두 번 낸 경우, 다시 도전에 `no`).

```txt
========================================
가위/바위/보 게임
========================================
플레이어 이름을 입력하세요. 
1. 가위 / 2. 바위 / 3. 보
번호를 선택하세요. : 

[게임 결과]
대환 : 가위 VS 컴퓨터: 보
=> 승리

현재 스코어 대환 1 : 0 컴퓨터

1. 가위 / 2. 바위 / 3. 보
번호를 선택하세요. : 

[게임 결과]
대환 : 가위 VS 컴퓨터: 보
=> 승리

현재 스코어 대환 2 : 0 컴퓨터

----------------------------------------
[최종 결과] 대환 2 : 0 컴퓨터
==> 대환 최종 승리!

[판별 승패 목록]
1판 : 가위 vs 보 => 승리
2판 : 가위 vs 보 => 승리

현재 연승: 1 연승

다시 도전 (yes/no) : 
게임을 종료합니다.
```

- 실제 화면에서는 입력한 글자가 프롬프트 뒤에 보인다. 위 출력에는 입력값이 빠져 있다.

## 6. 상속 (class_study.ipynb)

### 상속과 메서드 오버라이딩

- 상속(Inheritance): 부모 클래스(Base Class, Super Class, Parent Class)의 기능을 자식 클래스가 물려받는다. `class Dog(Animal):`처럼 클래스 이름 뒤 괄호에 부모를 적는다.
- 메서드 오버라이딩(Method Overriding): 부모에 있는 메서드를 자식이 같은 이름으로 다시 정의한다.

```py
# 부모 클래스 Base Class, Super Class, Parent Class
class Animal():
    # 공통 기능
    def breathe(self):
        print("숨을 쉽니다.")

    def sound(self):
        print("소리를 냅니다.")

# 상속 (Inheritance)
class Dog(Animal):
    # 상속받은 클래스에만 있는 고유 기능
    def bark(self):
        print("멍멍")

# 인스턴스
dog1 = Dog()
dog1.breathe() # 부모 클래스로 부터 물려받은 메서드를 그대로 사용할 수 있다.
dog1.bark()

# 메소드 오버라이딩(Method Overiding)
# 부모 클래스에서 정의된 메소드를 수정 또는 추가하는 기능(재정의)
class Cat(Animal):
    def sound(self):
        print("야옹") # 재정의

nabi = Cat()
nabi.sound()
```

```txt
숨을 쉽니다.
멍멍
야옹
```

- 자식은 부모의 메서드를 그대로 쓴다(`dog1.breathe()`). 반대는 안 된다. 직접 실행하니 `Cat().bark()`는 `AttributeError`가 났다.

### super(): 부모의 `__init__` 호출

자식 클래스가 `__init__`을 새로 정의하면 부모의 `__init__`은 자동으로 실행되지 않는다. `super()`로 부모 쪽을 직접 불러 준다.

```py
class Person():
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    def __init__(self, name, age, major):
        super().__init__(name, age) # super() 를 통해서 부모 클래스의 생성자를 접근하여 name 속성을 설정
        self.major = major

std1 = Student("손흥민", 32, "스포츠댄스")
print(std1.name) # 부모 클래스로 부터 물려받은 속성
print(std1.major) # 자식 고유의 속성
```

```txt
손흥민
스포츠댄스
```

- `super().__init__(name, age)`가 부모의 `__init__`을 실행해 `name`, `age`를 만든다. 직접 실행하니 이 줄을 빼면 `std1.name`에서 `AttributeError`가 났다.

## 7. 캡슐화

### 비공개 변수와 이름 바꾸기 (name mangling)

속성 이름 앞에 밑줄 두 개(`__`)를 붙이면 클래스 바깥에서 그 이름으로 바로 접근할 수 없다.

```py
class Secret():
    def __init__(self):
        self.__password = "1234"

my_password = Secret()
print(my_password._Secret__password)
```

```txt
1234
```

- 직접 실행하니 `my_password.__password`는 `AttributeError`가 났다. 파이썬이 `_클래스이름__속성이름`(`_Secret__password`)으로 이름을 바꿔 저장하기 때문이다.
- 바뀐 이름으로는 접근되므로 완전히 막는 장치는 아니다.

### Getter / Setter

비공개 속성(`__money`)은 메서드를 통해서만 읽고 바꾸게 한다.

```py
class Wallet():
    def __init__(self): # 숨겨진 데이터
        self.__money = 1000

    def get_money(self): # Getter 값을 안전하게 전달하는 용도
        return self.__money

    def set_money(self, amount): # Setter 검증 로직을 추가
        if amount >= 0:
            self.__money += amount
        else:
            print("잘못된 값입니다.")

wallet = Wallet()
wallet.set_money(2000)
print(wallet.get_money())
```

```txt
3000
```

- Getter는 값을 꺼내 주고, Setter는 `amount >= 0` 검사를 통과해야 바꾼다. 직접 실행하니 음수는 `잘못된 값입니다.`가 출력되고 잔액은 그대로였다.
- `+=`라서 1000에 2000이 더해져 `3000`이 된다.
