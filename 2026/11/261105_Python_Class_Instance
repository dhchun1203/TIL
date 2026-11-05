## 1. 오늘의 흐름

1. **클래스와 인스턴스:** 설계도로 객체를 만든다.
2. **생성자 `__init__`:** 인스턴스가 만들어질 때 자동으로 실행된다.
3. **`self`:** 만들어진 인스턴스 자기 자신.
4. **속성과 메서드:** 데이터는 속성에, 동작은 메서드에.
5. **`__str__`, `__repr__`:** 객체를 출력할 때 보이는 모양.
6. **실습:** 은행 계좌, 쇼핑 카트.

## 2. 클래스와 인스턴스

```py
# 클래스 정의(선언)

class User:
    def __init__(self, name): # 생성자: 클래스가 처음 생성될 때 자동으로 호출
        self.name = name # 인스턴스 안에 정의된 변수를 초기화

    def show(self): # 메서드(Method)
        return f"회원 이름 : {self.name}"

# 인스턴스 생성 : 붕어빵 만들기

user_a = User("손흥민")
user_b = User("이강인")

# User 클래스로 생성한 인스턴스 (메모리에 생성)

# 클래스의 인스턴스 변수
print(user_a.name)
print(user_b.name)

# 클래스의 메서드 접근
print(user_a.show())
print(user_b.show())

print(type(user_a))
print(user_a is user_b)
print(id(user_a))
print(id(user_b))
```

```txt
손흥민
이강인
회원 이름 : 손흥민
회원 이름 : 이강인
<class '__main__.User'>
False
(id 값 두 개, 서로 다름)
```

| 비유 | 파이썬 |
| --- | --- |
| 붕어빵 틀 | 클래스 `User` (설계도) |
| 붕어빵 | 인스턴스 `user_a`, `user_b` |
| 틀에 넣는 속 재료 | `User("손흥민")`에서 넘기는 값 |

- `class User:`로 클래스를 선언한다. 클래스 이름은 대문자로 시작한다(예제가 그렇다).
- `User("손흥민")`처럼 클래스를 함수처럼 호출하면 인스턴스가 만들어지고, 그때 `__init__`이 자동으로 실행된다.
- `self.name = name`: 넘겼던 값을 인스턴스의 변수(속성)에 담는다. 인스턴스마다 자기 `name`을 따로 가진다.
- `user_a.show()`: 인스턴스 뒤에 `.`을 붙여 메서드를 호출한다. 클래스 안에 만든 함수를 메서드라고 부른다.
- `user_a is user_b`는 `False`. 같은 클래스에서 만들어도 서로 다른 객체다. `id()`도 두 값이 다르다.
- `type(user_a)`는 `<class '__main__.User'>`. 파일을 직접 실행할 때의 출력이다.

## 3. self

```py
class Monitor:
    def display_info(self): # self 는 만들어진 인스턴스 자체를 가리킨다.(메소드 호출부의 '.' 앞에 인스턴스)
        self.resolution = "1920X1080"
        self.size = "27인치"
        print(f"모니터 정보\n해상도: {self.resolution}\n크기: {self.size}\n객체 ID: {id(self)}")

monitor = Monitor() # 인스턴스 생성
print(id(monitor)) # 모니터 인스턴스의 ID 출력

monitor.display_info()
```

```txt
(id 값)
모니터 정보
해상도: 1920X1080
크기: 27인치
객체 ID: (위와 같은 id 값)
```

- `monitor.display_info()`에서 `.` 앞의 `monitor`가 메서드의 `self`로 들어간다.
- 직접 실행하니 `id(monitor)`와 메서드 안의 `id(self)`가 같은 숫자다. 같은 객체다.
- 이 예제는 `__init__` 없이 메서드 안에서 속성을 만든다. 직접 실행하니 `display_info()`를 호출하기 전에 `monitor.resolution`을 쓰면 `AttributeError: 'Monitor' object has no attribute 'resolution'`가 난다. 속성은 `self.resolution = ...`이 실행된 뒤에 생긴다.

## 4. 속성과 메서드

```py
class SmartPhone:
    def __init__(self, model):
        self.model = model # 속성 (Attribute)
        self.battery = 50  # 속성

    def charge(self, amount):
        self.battery += amount
        if self.battery > 100:
            self.battery = 100
            print(f"배터리 충전 완료 : {self.battery}%")
        else:
            print(f"배터리 충전 중 ... : {self.battery}%")

phone_a = SmartPhone("아이폰")
print(phone_a.model)
phone_a.charge(30)
phone_a.charge(30)

phone_b = SmartPhone("갤럭시")
```

```txt
아이폰
배터리 충전 중 ... : 80%
배터리 충전 완료 : 100%
```

| 이름 | 무엇 | 예 |
| --- | --- | --- |
| 속성 (attribute) | 인스턴스가 가진 데이터 | `self.model`, `self.battery` |
| 메서드 (method) | 인스턴스가 할 수 있는 동작 | `charge()` |

- `self.battery = 50`: 모든 인스턴스가 `50`에서 시작한다.
- `charge(30)` 첫 번째: 50 + 30 = `80`. 100을 넘지 않아 `else`로 간다.
- 두 번째: 80 + 30 = 110. 100을 넘어서 `100`으로 맞추고 "충전 완료"를 출력한다.
- `phone_b`는 새 인스턴스라서 배터리가 다시 `50`에서 시작한다. 직접 실행하니 `phone_a.battery`는 `100`, `phone_b.battery`는 `50`이다.

## 5. 실습: 은행 계좌 클래스

```py
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner          # 예금주
        self.balance = balance      # 잔액

    def __str__(self):
        return f"예금주 : {self.owner}, 잔액: {self.balance:,}원"

    def __repr__(self):
        return f"예금주 : {self.owner}, 잔액: {self.balance:,}원"

    # 입금 처리 메서드
    def deposit(self, amount):
        self.balance += amount
        print(f"{amount:,}원 입금 완료 / 잔액 : {self.balance:,}원")

    # 출금 처리 메서드
    # 잔액 확인 => 잔액 부족 메시지 출력
    def withdraw(self, amount):

        # Psudo Code
        # 잔액 확인
        if self.balance < amount:
            print(f"출금 잔액이 부족합니다. 잔액 : {self.balance:,}원 || 출금 요청액 : {amount:,}원")
            return
        self.balance -= amount
        # print(f"{amount:,}원 출금 완료 / 잔액 : {self.balance:,}원")
        print(f"{amount:,}원 출금 완료")
        return self.balance # 출금 후 잔액 리턴

account_a = BankAccount("전대환", 100000)
print(account_a)
account_a.deposit(500000)
account_a.withdraw(1000000)
print(f"잔액 : {account_a.withdraw(200000):,}원")
print(f"잔액 : {account_a.withdraw(300000):,}원")

account_c = BankAccount("홍길동", 200000)
print([account_a, account_c])
```

```txt
예금주 : 전대환, 잔액: 100,000원
500,000원 입금 완료 / 잔액 : 600,000원
출금 잔액이 부족합니다. 잔액 : 600,000원 || 출금 요청액 : 1,000,000원
200,000원 출금 완료
잔액 : 400,000원
300,000원 출금 완료
잔액 : 100,000원
[예금주 : 전대환, 잔액: 100,000원, 예금주 : 홍길동, 잔액: 200,000원]
```

### 어떻게 흐르나

| 줄 | 동작 | 잔액 |
| --- | --- | --- |
| `BankAccount("전대환", 100000)` | 계좌 생성 | 100,000 |
| `deposit(500000)` | 입금 | 600,000 |
| `withdraw(1000000)` | 잔액 부족 메시지, `return`으로 끝남 | 600,000 |
| `withdraw(200000)` | 출금 | 400,000 |
| `withdraw(300000)` | 출금 | 100,000 |

- `withdraw`: 잔액이 부족하면 메시지만 출력하고 `return`으로 함수를 끝낸다. 잔액은 바뀌지 않는다.
- `{self.balance:,}`: 천 단위 쉼표를 붙인다.
- `print(f"잔액 : {account_a.withdraw(200000):,}원")`: f-string 안의 `withdraw(200000)`가 먼저 실행되어 `200,000원 출금 완료`가 찍히고, 돌려받은 잔액 `400000`이 문장에 들어간다. 그래서 출력 순서가 "출금 완료" 다음 "잔액"이다.
- 직접 실행하니 잔액이 부족한 출금은 `None`을 돌려준다. 그 값을 `f"{...:,}"`에 넣으면 `TypeError: unsupported format string passed to NoneType.__format__`가 난다. 이 예제에서는 부족한 출금을 f-string 안에서 호출하지 않아 오류가 나지 않는다.

### `__str__`와 `__repr__`

- `print(account_a)`는 `__str__`의 반환값을 출력한다.
- `print([account_a, account_c])`처럼 리스트 안에 들어 있을 때는 `__repr__`의 반환값이 출력된다. 이 예제는 두 메서드가 같은 문자열을 돌려줘서 어느 쪽이 쓰였는지 출력만으로는 구분되지 않는다. 두 문자열을 다르게(`STR`, `REPR`) 해서 직접 실행하니 `print(a)`는 `STR`, `print([a])`는 `[REPR]`이다.
- 두 메서드가 없으면 `print(객체)`는 `<__main__.Item object at 0x...>` 형태로 나온다. 직접 실행해서 확인했다.

## 6. 실습: 쇼핑 카트

요구사항(파일 맨 위 주석):

- `shopping cart` 클래스 선언, 상품 목록 저장 기능
- 메서드: 장바구니 담기, 장바구니 빼기, 상품 목록 조회
- `상품` 클래스: 속성은 제품명, 가격

```py
class ShoppingCard:
    # 저장 기능
    def __init__(self):
        self.bucket = []
    # 장바구니 담기
    def add_item(self, item):
        self.bucket.append(item)
    # 장바구니 빼기
    def remove_item(self, item):
        self.bucket.remove(item)
    # 상품 목록 조회
    def show_items(self):
        return self.bucket

# 상품 클래스
class Item:
    def __init__(self, name, price):
        self.name = name
        self.price = price

# 쇼핑 카트 생성
cart = ShoppingCard()

# 상품 3개 생성
item_a = Item("초코파이", 5000)
item_b = Item("새우깡", 2000)
item_c = Item("빅파이", 3000)

# 상품 1개 장바구니 담기
cart.add_item(item_a)
cart.add_item(item_b)
cart.add_item(item_c)
# 상품 1개 장바구니에서 빼기
cart.remove_item(item_b)
# 장바구니에 담긴 상품 목록 조회
for item in cart.show_items():
    print(item.name, item.price)
```

```txt
초코파이 5000
빅파이 3000
```

| 클래스 | 속성 | 메서드 |
| --- | --- | --- |
| `ShoppingCard` | `bucket` (빈 리스트에서 시작) | `add_item`, `remove_item`, `show_items` |
| `Item` | `name`, `price` | 없음 |

- `self.bucket = []`: 카트마다 자기 리스트를 하나씩 가진다. 직접 실행하니 카트 하나에 담아도 다른 카트의 `bucket`은 비어 있다.
- `add_item`은 리스트의 `append`, `remove_item`은 리스트의 `remove`를 감싼다. 클래스 안에서 기존 함수를 재사용한 것이다.
- `Item`은 데이터만 담는 클래스라 메서드가 없다. `__init__`으로 `name`, `price`를 받아 속성으로 저장한다.
- `remove_item(item_b)`로 `새우깡`이 빠져 `초코파이`와 `빅파이`만 출력된다.
- 클래스 이름은 코드에 `ShoppingCard`로 적혀 있다.
