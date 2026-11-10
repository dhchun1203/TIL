## 1. 오늘의 흐름

펫을 키우는 다마고치 게임의 `Pet` 클래스를 만들었다. 펫 한 마리가 자기 상태(배고픔, 행복도, 에너지)를 갖고, 밥 주기, 놀아주기, 잠자기, 하루 지나기로 그 상태가 바뀐다.

## 2. Pet 클래스 구조

| 구분 | 이름 | 역할 |
| --- | --- | --- |
| 클래스 변수 | `count` | 지금까지 입양한 펫 수 |
| 인스턴스 변수 | `name`, `hunger`, `happy`, `energy`, `alive` | 이름, 배고픔(30), 행복도(70), 에너지(70), 생존 여부 |
| 표시 | `__str__()`, `mood()`, `bar()` | 상태 출력, 표정, 10칸 막대 |
| 내부 메서드 | `_clamp()`, `_change()` | 값 범위 유지, 상태 값 변경 |
| 행동 | `feed()`, `play()`, `sleep()` | 밥 주기, 놀아주기, 잠자기 |
| 시간 | `pass_time()` | 하루 지나기, 랜덤 이벤트, 떠나는 조건 |

```txt
feed() / play() / sleep() / pass_time()
        └─ _change(hunger, happy, energy)
              └─ _clamp(값)   # 0 ~ 100 유지
```

## 3. 코드

```python
# 펫 한마리를 표현하는 클래스
import random
class Pet:
    # 클래스 변수
    count = 0 # 지금까지 입양한 펫 수를 기록 
    
    def __init__(self, name):
        self.name = name  # 이름
        self.hunger = 30 # 배고픔(0~100)
        self.happy = 70 # 행복도(0~100)
        self.energy = 70 # 에너지(0~100)
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
            self._change(happy=15)
            print(f"{self.name} 숨겨진 장난감을 찾았어요.")
            
        if self.hunger >= 100 or self.happy <= 0:
            self.alive = False

#-----------------테스트코드-------------------
choco = Pet("초코")
dubu = Pet("두부")

print("입양한 펫 수:", Pet.count)

day = 1

while choco.alive:
    choco.pass_time()
    print(f"{day}일차: 배고픔 {choco.hunger}, 행복도 {choco.happy}")
    day += 1

print(choco)
```

## 4. 핵심 정리

- **클래스 변수:** `count`는 모든 펫이 함께 보는 값이라 `Pet.count += 1`처럼 클래스 이름으로 올린다.
- **`__str__`:** `print(펫)`을 하면 이 메서드가 돌려준 문자열이 출력된다.
- **상태 변경은 한 곳에서:** 모든 행동이 `_change()`를 거치고, `_clamp()`가 값을 0~100으로 맞춘다. 범위 검사를 행동마다 쓰지 않아도 된다.
- **밑줄 하나(`_`)는 관례:** "클래스 안에서 쓰는 메서드"라는 표시일 뿐 막지는 않는다. 직접 실행하니 밖에서 `p._clamp(130)`도 호출됐다. 이름이 바뀌는 밑줄 두 개(`__`)와 다르다.
- **행동 메서드:** 조건에 안 맞으면 메시지만 출력하고 `return`으로 끝나 상태가 바뀌지 않는다.

| 메서드 | 거절 조건 | 배고픔 | 행복도 | 에너지 |
| --- | --- | --- | --- | --- |
| `feed()` | 배고픔 10 이하 | -30 | +5 |  |
| `play()` | 에너지 20 미만 | +10 | +25 | -20 |
| `sleep()` | 없음 | +5 |  | +40 |
| `pass_time()` | 없음 | +10 | -10 (20% 확률로 +15) | -5 |

- **떠나는 조건:** 배고픔 100 이상 또는 행복도 0 이하. 배고픔은 매일 10씩만 오르므로 테스트 코드는 항상 7일차에 끝난다(난수를 바꿔 500번 직접 실행해 확인).

## 5. 실행 예시

장난감 이벤트는 난수라 실행할 때마다 달라진다.

```txt
입양한 펫 수: 2
초코 숨겨진 장난감을 찾았어요.
1일차: 배고픔 40, 행복도 75
초코 숨겨진 장난감을 찾았어요.
2일차: 배고픔 50, 행복도 80
초코 숨겨진 장난감을 찾았어요.
3일차: 배고픔 60, 행복도 85
4일차: 배고픔 70, 행복도 75
5일차: 배고픔 80, 행복도 65
6일차: 배고픔 90, 행복도 55
7일차: 배고픔 100, 행복도 45
초코 떠남
배고픔: ■■■■■■■■■■ 100
행복도: ■■■■□□□□□□  45
에너지: ■■■□□□□□□□  35
```
