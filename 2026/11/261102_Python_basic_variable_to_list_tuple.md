## 1. 오늘의 흐름

1. **변수:** 변수에 값을 다시 넣는다(재할당). (Chapter01)
2. **연산자:** 계산하고, 비교하고, 조건을 묶는다. (Chapter02)
3. **문자열 포맷:** 값을 보기 좋게 출력한다. (Chapter02)
4. **리스트와 튜플:** 값 여러 개를 한 변수에 담고, 꺼내고, 개수를 세고, 있는지 확인한다. (Chapter03)

## 2. 변수 (Chapter01)

### 변수 재할당

10/30 자료형 셀(`age = 25` 등)에 이어지는 내용.

```py
age = 20  # 메모리에 20 객체를 생성, age는 메모리 주소를 기억
age = 25  # 메모리에 25 객체를 생성, age -> 25
print(age)
```

```plain text
age = 20     age ──▶ [ 20 ]

age = 25     age ──▶ [ 25 ]
```

- 두 번째 줄 뒤에는 `age`가 25 객체를 가리킨다. (노트북 주석: `age -> 25`)

### 자기 자신을 이용한 재할당

```py
money = 5000
```

```py
money = money + 2000
print(money)
```

- 오른쪽 `money + 2000`을 먼저 계산하고, 그 결과를 `money`에 넣는다.
- 직접 실행한 결과 `7000`.
- 줄여 쓴 형태가 3장의 `+=`.

### type()과 id()

```py
x = 10
print(type(x))
print(id(x))

x = "hello"
print(type(x))
print(id(x))
```

- `type(x)`: 직접 실행한 결과 `<class 'int'>`, `<class 'str'>`.
- 같은 `x`에 문자열을 넣어도 에러 없이 `type`이 `str`로 바뀐다.
- `id(x)`: 직접 실행했을 때 `x = 10`일 때와 `x = "hello"`일 때 값이 서로 달랐다.

재할당 전후의 `id`를 비교한 예제.

```py
age = 20
print(id(age))
age = 25
print(id(age))
```

- 실행하니 두 값이 서로 달랐다.

### 가비지 컬렉터(Garbage Collector)

"메모리 할당 해제를 개발자가 신경 쓰지 않는다."

## 3. 연산자 (Chapter02)

### 산술 연산자

```py
1+1
print(10*2)
print(10/2)
print(10%3)
print(10//3)
print(10**2)
```

| 식 | 의미 | 결과 |
| --- | --- | --- |
| `10*2` | 곱셈 | `20` |
| `10/2` | 나눗셈 (결과는 소수) | `5.0` |
| `10%3` | 나머지 | `1` |
| `10//3` | 몫 | `3` |
| `10**2` | 거듭제곱 | `100` |

- 첫 줄 `1+1`의 `2`는 저장된 출력에 없다. 마지막 줄이 `print(...)`다.

### 복합 대입 연산자

```py
score = 100
score += 50  # score = score + 50
score %= 3   # score = score % 3
print(score)
```

- `score`는 100 → 150 → 0. 출력 `0`.
- 2장의 `money = money + 2000`은 `money += 2000`으로 줄여 쓸 수 있다.

### 비교 연산자

```py
a = 10
print(a == 10)  # True
print(a != 10)  # False
print(a > 10)   # False
print(a < 10)   # False
print(a >= 10)  # True
print(a <= 10)  # True
```

- `==` 같다, `!=` 다르다. 대입 `=`과 헷갈리지 않기.
- 문자열도 비교한다. 노트북 출력: `"apple" > "banana"`는 `False`, `ord("a")`는 `97`, `ord("b")`는 `98`.

### 논리 연산자

```py
is_adult = True
has_ticket = False

print(is_adult and has_ticket)  # False
print(is_adult or has_ticket)   # True
print(not is_adult)             # False
```

- `and`: 둘 다 참이어야 참
- `or`: 하나라도 참이면 참
- `not`: 참과 거짓을 뒤집는다

### 단락 평가

```py
print(False and (10/0 > 1))
# print(False or (10/0 > 1))  # ZeroDivisionError
```

- `and`는 왼쪽이 `False`면 오른쪽을 계산하지 않는다. 0으로 나누는 식이 있어도 `False`가 나온다.
- `or`는 왼쪽이 `False`면 오른쪽까지 계산해서 `ZeroDivisionError`가 난다. 직접 실행해서 둘 다 확인했다.

### 연산자 우선순위

```plain text
**
*
/
//
%
+
-
```

```py
result = 10 + 2 * 5 > 25    # False
result = (10 + 2) * 5 > 25  # True
print(result)
```

- 곱셈이 먼저라 첫 식은 `10 + 10 > 25`, 즉 `False`.
- 괄호가 먼저. 둘째 식은 `60 > 25`, 즉 `True`. 마지막 `print`는 `True`를 출력한다.

## 4. 문자열 포맷 (Chapter02)

값을 문자열에 넣는 방법이 세 가지 나왔다.

### % 방식

```py
print("선발 출전: %s, %d" % ("손흥민", 20))
print("선발 출전: [%5d]" % (20))
print("소수점 표기 : %.2f" % (3.141592))
```

- `%s` 문자열, `%d` 정수.
- `%5d` 출력 `[   20]`.
- `%.2f` 출력 `3.14`.

### format 방식과 f-string 방식

```py
name = "Cake"
price = 6500
qty = 2

# format 방식
format_str = "주문내역: {0} {1} -> 총 {2}원".format(name, qty, price)
print(format_str)

# f-string 방식(주로 사용)
format_str = f"주문내역: {name} {price} -> 총 {(price * qty):>10,}원"
print(format_str)
```

```plain text
주문내역: Cake 2 -> 총 6500원
주문내역: Cake 6500 -> 총     13,000원
```

- format: `{0}`, `{1}`, `{2}`는 `.format()`에 넣은 값의 순서.
- f-string: 문자열 앞에 `f`, `{}` 안에 변수나 식을 바로 쓴다.
- `{값:>10,}`: `>` 오른쪽 정렬, `10` 폭 10칸, `,` 천 단위 쉼표. 출력 `13,000`이 10칸 안에서 오른쪽에 붙어 있다.
- 문자열에도 쓴다. `print(f"금액: {15000:>8,}원")`의 출력은 `금액:   15,000원`.

### 영수증 만들기

```py
# 가상의 주문 데이터
orders = [
    ("Latte", 5000, 2),
    ("Americano", 3000, 1),
    ("Cake", 6000, 1)
]
print("=" * 34)
print(f"{'메뉴명':<10} {'단가':>8} {'금액':>9}")
print("-" * 34)

total_amount = 0

for menu, price, qty in orders:
    subtotal = price * qty
    total_amount += subtotal
    print(f"{menu:<10} {price:>12,} {subtotal:>10,}")

print("-" * 34)
print(f"{'합계금액':<10}{total_amount:>20,}원")
```

```plain text
==================================
메뉴명              단가        금액
----------------------------------
Latte             5,000     10,000
Americano         3,000      3,000
Cake              6,000      6,000
----------------------------------
합계금액                    19,000원
```

- `"=" * 34`: 문자열을 34번 반복. 구분선용.
- `orders`는 값 세 개짜리 묶음(튜플)을 담은 리스트. 튜플은 5장에서 다룬다.
- `for menu, price, qty in orders`: 묶음 하나를 꺼내 세 변수에 나눠 담는다.
- `total_amount += subtotal`로 소계를 더해 합계를 만든다. 10,000 + 3,000 + 6,000 = 19,000.

## 5. 리스트와 튜플 (Chapter03)

### 괄호로 구분한다

| 자료구조 | 괄호 | 노트북 내용 |
| --- | --- | --- |
| 리스트 | `[ ]` | 값을 바꿀 수 있다 |
| 튜플 | `( )` | 만든 뒤에는 추가, 수정, 삭제를 할 수 없다 |
| 딕셔너리 | `{ }` | 제목과 괄호 설명만 있고 코드는 아직 없다 |

### 리스트

```py
shopping_list = ["apple", "banana", "cherry"]

print(shopping_list)
print(shopping_list[0])

shopping_list[0] = "kiwi"
print(shopping_list[0])
print(shopping_list)
```

```plain text
['apple', 'banana', 'cherry']
apple
kiwi
['kiwi', 'banana', 'cherry']
```

- 인덱스로 요소를 바로 바꿀 수 있다. 튜플은 같은 방식으로 바꾸면 에러가 난다(아래 튜플 참고).

### 리스트 메서드

```py
# append
fruits = ["apple"]
fruits.append("banana")
print(fruits)

# insert
fruits.insert(0, "kiwi")
print(fruits)

# remove
# fruits.remove("apple")
# print(fruits)

# fruits.remove("grape")
```

- `append("banana")`: 맨 뒤에 추가. 결과 `['apple', 'banana']`.
- `insert(0, "kiwi")`: 0번 자리에 삽입. 결과 `['kiwi', 'apple', 'banana']`.
- `remove("apple")`: 그 값을 찾아 지운다. 노트북에서는 주석 처리.
- 없는 값(`"grape"`)을 지우면 `ValueError: list.remove(x): x not in list`. 직접 실행해서 확인했다.

### 튜플

```py
axis = (38.123, 41.123)

# axis[0] = 36.1234
# TypeError

point = 1, 2, 3
print(point, type(point))
```

- 출력 `(1, 2, 3) <class 'tuple'>`. 괄호 없이 쉼표만으로도 튜플.
- 요소를 바꾸면 `TypeError: 'tuple' object does not support item assignment`. 직접 실행해서 확인했다.

### 리스트와 튜플의 크기 비교

```py
import sys
print(sys.getsizeof([1,2,3]))  # 리스트 : 데이터의 순서를 보장
print(sys.getsizeof((1,2,3)))  # 튜플 : 데이터 안정성
```

- 노트북 출력 `88`, `72`.
- 내가 실행한 파이썬 3.13.18에서는 `88`, `64`.

### 인덱스

```py
colors = ["red", "green", "blue"]
print(colors[0])
print(colors[-1])
```

| 방향 | red | green | blue |
| --- | --- | --- | --- |
| 정방향 | 0 | 1 | 2 |
| 역방향 | -3 | -2 | -1 |

- 출력 `red`, `blue`. `-1`은 맨 뒤 요소.

### 슬라이싱

```py
numbers = [0, 1, 2, 3, 4, 5]

# 1번 부터 4번 미만까지
numbers[1:4]

# 2칸씩 간격으로 추출
numbers[::2]

# 역순으로 추출
numbers[::-1]

# 앞에서 3개만 추출
numbers[:3]

# 3번부터 끝까지
numbers[3:]
```

- 형식 `[시작:끝:간격]`. 노트북 주석: "1번 부터 4번 미만까지".
- 노트북 저장 출력은 `[3, 4, 5]` 하나뿐이다. 나머지 식은 `print`로 감싸지 않아 출력이 없다.
- 직접 실행한 값:

| 식 | 결과 |
| --- | --- |
| `numbers[1:4]` | `[1, 2, 3]` |
| `numbers[::2]` | `[0, 2, 4]` |
| `numbers[::-1]` | `[5, 4, 3, 2, 1, 0]` |
| `numbers[:3]` | `[0, 1, 2]` |
| `numbers[3:]` | `[3, 4, 5]` |

- 튜플도 슬라이싱된다. `print((10, 20, 30, 40)[1:3])`의 출력 `(20, 30)`.

### 2차원 리스트

```py
matrix = [
    [1,2,3],
    [4,5,6]
]

print(matrix[1])
print(matrix[1][2])
print(len(matrix), len(matrix[0]))
```

```plain text
[4, 5, 6]
6
2 3
```

- 리스트 안에 리스트를 넣은 형태. `[행][열]` 순서로 꺼낸다.
- `matrix[1]`: 두 번째 행 `[4, 5, 6]`.
- `matrix[1][2]`: 그 행의 세 번째 값 `6`. 인덱스는 0부터 센다.
- `len(matrix)`는 행 개수 `2`, `len(matrix[0])`은 첫 행의 값 개수 `3`. 출력 `2 3`.

### 길이와 포함 여부

```py
user_roles = ["admin", "member", "guest"]
print(len(user_roles))
print("admin" not in user_roles)
```

```plain text
3
False
```

- `len(user_roles)`: 요소 개수 `3`.
- `"admin" not in user_roles`: `admin`이 목록에 있으므로 `False`.
- 직접 실행하니 `"admin" in user_roles`는 `True`, `"owner" not in user_roles`도 `True`.
- 직접 실행하니 `len((10, 20, 30))`은 `3`, `len("hello")`는 `5`. 이 두 줄은 노트북에는 없고 확인용으로 돌려 봤다.

