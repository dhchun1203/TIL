## 1. 오늘의 흐름

다마고치 코드를 파일 여러 개로 나눴다. 초기 상태 값은 `config.py`로 빼고, `main.py`에서 `Pet` 클래스를 가져다 쓰고, `pet.py`의 테스트 코드는 `if __name__ == "__main__":` 아래로 옮겼다. 이어서 같은 방식으로 파일을 나눠 쇼핑몰 장바구니(`product.py`, `cart.py`)를 만들었다(6번). 마지막으로 Windows에 Docker Desktop을 설치했다(7번).

## 2. 파일 구조

| 파일 | 역할 | 가져오는 것 |
| --- | --- | --- |
| `config.py` | 초기 상태 값 상수 3개 | 없음 |
| `pet.py` | `Pet` 클래스 + 직접 실행할 때만 도는 테스트 코드 | `import config` |
| `main.py` | 프로그램 시작점. 펫을 만들어 출력 | `from pet import Pet` |

```txt
main.py ── from pet import Pet ──▶ pet.py ── import config ──▶ config.py
```

## 3. 코드

`config.py`

```python
# 초기 상태 값
START_HUNGER    = 30
START_HAPPY     = 70
START_ENERGY    = 70
```

`main.py`

```python
from pet import Pet

choco = Pet("초코")
print(choco)

print("pet.py 의 __name__:", __name__)
```

`pet.py`

```python
# 펫 한마리를 표현하는 클래스
import random

# from config import START_HUNGER, START_HAPPY, START_ENERGY
import config


class Pet:
    # 클래스 변수
    count = 0 # 지금까지 입양한 펫 수를 기록 
    
    def __init__(self, name):
        self.name = name  # 이름
        self.hunger = config.START_HUNGER # 배고픔(0~100)
        self.happy = config.START_HAPPY # 행복도(0~100)
        self.energy = config.START_ENERGY # 에너지(0~100)
        self.alive = True 
        Pet.count += 1
    
    def __str__(self):
        state = self.mood() if self.alive else "떠남"
        return (
            f"{self.name} {state}\n"
            f"배고픔: {self.bar(self.hunger)} {self.hunger:>3}\n"
            f"행복도: {self.bar(self.happy)} {self.happy:>3}\n"
            f"에너지: {self.bar(self.energy)} {self.energy:>3}\n"
        )
    
    def mood(self):
        if self.hunger >= 80 or self.happy <= 20: return "😭"
        if self.happy >= 70 and self.hunger <= 40: return "😊"
        return "😐"
    
    def bar(self, value):
        # 0 ~ 100 값을 10칸의 막대로 변환
        filled = value // 10
        return "■" * filled + "□" * (10 - filled)
        
    
    # 공통 메서드
    def _clamp(self, value): # 내부 메서드의 네이밍 컨벤션 _name()
        # 값의 범위를 (0 ~ 100) 유지
        if value < 0: return 0
        if value > 100: return 100
        return value
    
    def _change(self, hunger = 0, happy = 0, energy = 0):
        # 상태값은 반드시 이 메서드를 사용한다.
        self.hunger = self._clamp(self.hunger + hunger)
        self.happy = self._clamp(self.happy + happy)
        self.energy = self._clamp(self.energy + energy)
        
    # 밥주기
    def feed(self):
        if self.hunger <= 10:
            print(f"{self.name}: 배불러요~")
            return False
        self._change(hunger=-30, happy=5)
        print(f"{self.name}에게 밥을 먹였습니다.")
    
    # 놀아주기
    def play(self):
        if self.energy < 20:
            print(f"{self.name}: 너무 피곤해요...")
            return
        self._change(hunger=10, happy=25, energy=-20)
        print(f"{self.name}와 신나게 놀았습니다.")
        return True
    
    # 잠자기
    def sleep(self):
        self._change(hunger=5, energy=40)
        print(f"{self.name}가 잠들었어요 zZZ")
        return True

    # 시간 체크
    def pass_time(self):
        # 하루 지날때 마다 상태값 자연 감소
        self._change(hunger=10, happy=-10, energy=-5)
        # 즐거운 이벤트를 확률계산 (20%)
        if random.randint(1, 5) == 1 :
            self._change(happy=15, hunger=-15)
            print(f"{self.name} 숨겨진 장난감을 찾았어요.")
            
        if self.hunger >= 100 or self.happy <= 0:
            self.alive = False

#-----------------테스트코드-------------------
# pet.py 를 직접 실행했을 때 uv run pet.py 만 실행된다.
# import 했을 때는 실행되지 않는다.
if __name__ == "__main__":
    choco = Pet("초코")
    dubu = Pet("두부")

    print("입양한 펫 수:", Pet.count)

    day = 1

    while choco.alive:
        choco.pass_time()
        print(f"{day}일차: 배고픔 {choco.hunger}, 행복도 {choco.happy}")
        day += 1

    print(choco)

    print("pet.py 의 __name__:", __name__)
```

## 4. 핵심 정리

- **모듈:** `.py` 파일 하나가 모듈이다. 다른 파일에서 `import`로 가져다 쓴다.
- **설정값 분리:** 초기 상태 값을 `config.py` 상수로 모아 두면, 값을 바꿀 때 클래스 코드를 건드리지 않고 이 파일만 고친다. 대문자 이름은 상수라는 관례다.
- **`__name__`:** 파이썬이 모듈마다 넣어 주는 변수다. 직접 실행한 파일에서는 `"__main__"`, `import`된 파일에서는 파일 이름이다. 직접 실행하니 `import pet` 뒤 `pet.__name__`은 `pet`이었다.
- **`if __name__ == "__main__":`:** 이 파일을 직접 실행할 때만 아래 코드가 돈다. `main.py`가 `pet.py`를 가져와도 테스트 코드(초코가 떠날 때까지 도는 반복)는 실행되지 않는다.
- **`main.py`의 마지막 줄:** `main.py` 자신의 `__name__`을 찍으므로 `python main.py`로 실행하면 `__main__`이 나온다. 문구는 `pet.py 의 __name__`이지만 찍히는 것은 `main.py`의 값이다.

| 가져오는 방법 | 쓰는 법 | 특징 |
| --- | --- | --- |
| `import config` | `config.START_HUNGER` | 어느 파일의 값인지 이름에 드러난다 |
| `from config import START_HUNGER` | `START_HUNGER` | 짧게 쓸 수 있다. 가져온 시점의 값을 따로 들고 있다 |

- 직접 실행하니 `from config import START_HUNGER`로 가져온 뒤 `config.START_HUNGER = 99`로 바꿔도, 가져온 `START_HUNGER`는 `30` 그대로였다.
- 장난감 이벤트가 행복도 +15와 함께 배고픔 -15를 해서, 초코가 떠나는 날은 실행마다 다르다. 난수를 바꿔 500번 직접 실행하니 7일차에서 24일차 사이에 끝났다.

## 5. 실행 예시

`python main.py`

```txt
초코 😊
배고픔: ■■■□□□□□□□  30
행복도: ■■■■■■■□□□  70
에너지: ■■■■■■■□□□  70

pet.py 의 __name__: __main__
```

`python pet.py` (장난감 이벤트는 난수라 실행마다 다르다)

```txt
입양한 펫 수: 2
초코 숨겨진 장난감을 찾았어요.
1일차: 배고픔 25, 행복도 75
초코 숨겨진 장난감을 찾았어요.
2일차: 배고픔 20, 행복도 80
초코 숨겨진 장난감을 찾았어요.
3일차: 배고픔 15, 행복도 85
4일차: 배고픔 25, 행복도 75
5일차: 배고픔 35, 행복도 65
6일차: 배고픔 45, 행복도 55
7일차: 배고픔 55, 행복도 45
8일차: 배고픔 65, 행복도 35
9일차: 배고픔 75, 행복도 25
10일차: 배고픔 85, 행복도 15
초코 숨겨진 장난감을 찾았어요.
11일차: 배고픔 80, 행복도 20
12일차: 배고픔 90, 행복도 10
13일차: 배고픔 100, 행복도 0
초코 떠남
배고픔: ■■■■■■■■■■ 100
행복도: □□□□□□□□□□   0
에너지: □□□□□□□□□□   5

pet.py 의 __name__: __main__
```

## 6. 쇼핑몰 장바구니

### 구조

| 파일 | 클래스 | 아는 것 | 하는 일 |
| --- | --- | --- | --- |
| `product.py` | `Product` | 번호(`id`), 이름, 가격, 재고 | 품절 확인(`is_sold_out()`), 목록 한 줄(`__str__`) |
| `cart.py` | `Cart` | `items = {상품: 수량}` | 담기, 빼기, 상품 합계, 배송비, 결제 금액, 출력 |

```txt
cart.py ── from product import Product ──▶ product.py
```

### 코드

`product.py`

```python
class Product:
    next_id = 1 # 클래스 변수
    
    # 새 상품의 초기 정보 저장
    def __init__(self, name, price, stock):
        self.id = Product.next_id
        Product.next_id += 1 # 다음 상품을 위해 Index 값 + 1
        self.name = name
        self.price = price
        self.stock = stock
        
    def is_sold_out(self):
        return self.stock == 0
        
    def __str__(self):
        stock_text = "품절" if self.is_sold_out() else f"재고 {self.stock}개"
        return f"{self.id}: {self.name:<10} {self.price:>8,} {self.stock} ({stock_text})"

# 테스트 코드=========================================
# 이 파일을 직접 실행시켰을 때만 코드가 실행 되도록 하는 로직
# uv run product.py
if __name__ == "__main__":
    products = [
        Product("무선이어폰", 89_000, 3),
        Product("텀블러", 15_000, 10),
        Product("보조배터리", 32_000, 5),
        Product("마우스패드", 9_000, 0),
        Product("USB 허브", 30_000, 10),
    ]

    for p in products: print(p)
```

`cart.py`

```python
from product import Product

class Cart:
    def __init__(self):
        # 빈 장바구니 생성
        self.items = {} # {상품: 수량} 

    # 장바구니에 상품 담기
    def add(self, product, qty):
        # 품절 상품은 담을 수 없다.
        if product.is_sold_out():
            print("품절된 상품입니다.")
            return
        
        # 이미 장바구니에 담겨있는 수량
        in_cart_qty = self.items.get(product, 0)
        # 더 담을 수 있는 수량
        extra_qty = product.stock - in_cart_qty
        if qty > extra_qty: # 재고 수량 범위를 벗어난 경우
            print(f"재고가 부족합니다. (담을 수 있는 수량: {extra_qty}개)")
            return
        
        self.items[product] = self.items.get(product, 0) + qty
        print(f"{product.name} {qty}개를 담았습니다.")
        
    # 장바구니에서 상품 제거(빼기)
    def remove(self,product, qty):
            
        if product not in self.items:
            print("장바구니에 해당 상품이 없습니다.")
            return
        
        # 담은 수량 이상 빼면 상품을 통째로 삭제
        if qty >= self.items[product]:
            del self.items[product]
            print(f"{product.name}을 장바구니에서 모두 뺐습니다.")
        else:
            self.items[product] -= qty
            print(f"{product.name} {qty}개를 뺐습니다.")
    
    # 장바구니 비었는지 확인
    def is_empty(self):
        return len(self.items) == 0
    
    # 상품 합계 계산
    def subtotal(self):
        return sum(p.price * q for p, q in self.items.items())
    
    # 배송비
    def shipping(self):
        if self.is_empty(): return 0
        return 0 if self.subtotal() >= 30_000 else 3_000
    
    # 결제 금액 계산
    def total(self):
        return self.subtotal() + self.shipping()

    # 장바구니 내용 출력
    def show(self):
        line = "-" * 40
        print(line)
        
        if self.is_empty():
            print("장바구니가 비어있습니다.")
        
        for p, q in self.items.items():
            # 제품명 가격 x 수량 = 총합계
            print(f"{p.name:<10} x {q} = {p.price * q:>12,}원")
            
        print(line)
        print(f"{'상품 합계':<10} {self.subtotal():>15,}원")
        
        print(f"{'배송비':<10} {self.shipping():>15,}원")
        print(f"{'결제 금액':<10} {self.total():>15,}원")
        print(line)
        
#테스트 코드=========================================

# 제품 인스턴스 생성 
tumbler = Product("텀블러", 15_000, 10)
earphone = Product("무선이어폰", 89_000, 3)

# 카트 인스턴스 생성
cart = Cart()
cart.add(tumbler, 9)
cart.add(earphone, 2)
cart.show()
```

### 핵심 정리

- **자동 번호:** `self.id = Product.next_id`로 지금 번호를 먼저 저장하고, `Product.next_id += 1`로 다음 번호를 올린다. 직접 실행하니 상품 5개가 1~5번을 받았다.
- **장바구니는 딕셔너리 `{상품: 수량}`:** 상품 객체를 키로 쓴다. 재고는 상품의 `stock`, 담은 수량은 딕셔너리의 값이라 두 값이 섞이지 않는다.
- **담을 수 있는 수량 = 재고 - 이미 담은 수량:** `self.items.get(product, 0)`은 키가 없으면 `0`을 돌려준다. 직접 실행하니 텀블러(재고 10)를 9개 담은 뒤 2개는 `담을 수 있는 수량: 1개`로 거절됐고, 1개는 담겨 10개가 됐다.
- **빼기:** 담은 수량 이상을 빼면 `del self.items[product]`로 키를 통째로 지우고, 아니면 `-=`로 줄인다. 직접 실행하니 10개에서 3개를 빼면 7개, 99개를 빼면 장바구니가 `{}`가 됐다.
- **금액 계산:** `sum(p.price * q for p, q in self.items.items())`로 합계를 낸다. 배송비는 비어 있으면 0원, 합계가 30,000원 이상이면 0원, 아니면 3,000원이다. `30_000`의 밑줄은 읽기 쉽게 쓰는 숫자 표기로 `30000`과 같다.
- **`cart.py`의 테스트 코드:** `if __name__ == "__main__":` 밖에 있어서, 직접 실행하니 `import cart`만 해도 테스트 출력이 나왔다. `product.py`처럼 감싸면 가져올 때 실행되지 않는다.
- **한글 정렬:** `:<10`은 글자 수로 칸을 맞추는데 한글은 화면에서 2칸이라, 아래 출력처럼 상품명 길이에 따라 줄이 밀린다. 화면 폭(`unicodedata.east_asian_width`)으로 세서 공백을 채우면 맞출 수 있다.

### 실행 예시

`python product.py`

```txt
1: 무선이어폰        89,000 3 (재고 3개)
2: 텀블러          15,000 10 (재고 10개)
3: 보조배터리        32,000 5 (재고 5개)
4: 마우스패드         9,000 0 (품절)
5: USB 허브       30,000 10 (재고 10개)
```

`python cart.py`

```txt
텀블러 9개를 담았습니다.
무선이어폰 2개를 담았습니다.
----------------------------------------
텀블러        x 9 =      135,000원
무선이어폰      x 2 =      178,000원
----------------------------------------
상품 합계              313,000원
배송비                      0원
결제 금액              313,000원
----------------------------------------
```

## 7. Windows에 Docker Desktop 설치

Docker Desktop은 Windows에서 WSL 2(Windows 안에서 Linux를 돌리는 기능) 위에서 동작한다. 그래서 WSL을 먼저 설치하고 Docker Desktop을 설치한다.

### 요구 사항 (WSL 2 기반)

| 항목 | 조건 |
| --- | --- |
| Windows 11 | 64비트 Pro/Enterprise/Education 23H2(빌드 22631) 이상 |
| Windows 10 | 64비트 Pro/Enterprise/Education 22H2(빌드 19045) |
| WSL | 2.1.5 이상 |
| 하드웨어 | 64비트 CPU(SLAT 지원), RAM 8GB, BIOS/UEFI에서 가상화 켜기 |

### 1) WSL 설치

PowerShell을 **관리자 권한으로 실행**(오른쪽 클릭 → 관리자 권한으로 실행)하고 입력한다.

```powershell
wsl --install
```

- WSL에 필요한 기능을 켜고 Ubuntu 배포판을 설치한다. 이 명령으로 설치하면 WSL 2가 기본이다.
- 끝나면 **컴퓨터를 재시작**한다.

설치 확인:

```powershell
wsl --version
wsl -l -v
```

- `wsl --version`: WSL 버전을 보여 준다. Docker Desktop에는 2.1.5 이상이 필요하다.
- `wsl -l -v`: 설치된 배포판과 각각의 WSL 버전(1 또는 2)을 보여 준다.

### 2) Docker Desktop 설치

PowerShell에서 winget으로 설치한다.

```powershell
winget install -e --id Docker.DockerDesktop
```

- `-e`는 ID를 정확히 일치하는 패키지만 고르는 옵션, `--id`는 패키지 ID로 찾는 옵션이다.
- 설치 파일(`Docker Desktop Installer.exe`)로 직접 설치할 때는 설정 화면에서 **Use WSL 2 instead of Hyper-V**를 선택한다.

### 3) 실행과 확인

1. 시작 메뉴에서 **Docker Desktop**을 검색해 실행한다.
2. Docker Subscription Service Agreement가 나오면 **Accept**를 누른다. 동의하지 않으면 Docker Desktop이 실행되지 않는다.
3. 터미널에서 확인한다.

```powershell
docker --version
docker run hello-world
```

- `docker run hello-world`는 테스트용 이미지를 받아 컨테이너로 실행하고, 안내 메시지를 출력한 뒤 끝난다.
