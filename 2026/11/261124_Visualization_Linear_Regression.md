## 1. 오늘의 흐름

데이터를 그림으로 보고, 두 값의 관계를 숫자로 잰 뒤, 첫 머신러닝 모델인 선형회귀로 가격을 예측했다. "면적이 넓으면 비싸다"를 직선 하나로 표현하고, 그 직선이 얼마나 틀리는지까지 쟀다.

1. matplotlib(선·히스토그램·산점도)과 seaborn(박스플롯)으로 데이터 그려 보기
2. 상관계수(`corr`)로 두 열의 관계를 숫자로 보기
3. 가상 오피스텔 데이터로 선형회귀를 학습·예측하고 MAE·RMSE·R²로 평가하기

## 2. 구조

```txt
데이터 만들기   price = 450 × area + 5000 + 잡음   (특이매물 5건은 가격을 더 올림)
   ↓
학습            model.fit(X, y)       → 기울기 w, 절편 b를 찾는다
   ↓
예측            model.predict(새 면적) → 가격
   ↓
평가            MAE · RMSE · R²      → 예측이 실제와 얼마나 다른가
```

| 지표 | 뜻 | 오늘 값 (200건) |
| --- | --- | --- |
| MAE | 오차 절댓값의 평균. 평균적으로 얼마나 틀리나 | 2,493 |
| MSE | 오차 제곱의 평균. 단위도 제곱 | 16,240,971 |
| RMSE | MSE의 제곱근. 원래 단위로 돌아옴 | 4,030 |
| R² | 가격 차이를 모델이 얼마나 설명하나. 1에 가까울수록 좋다 | 0.803 |

입력 `X`는 표(2차원)여야 해서 `office[["area"]]`로, 정답 `y`는 열 하나라 `office["price"]`로 꺼낸다.

### 문법 사용법

```python
import matplotlib.pyplot as plt
import seaborn as sns
plt.rcParams["font.family"] = "AppleGothic"     # 한글 글꼴 (윈도우는 "Malgun Gothic")

# 그림: 캔버스(Figure) + 그래프(Axes)
fig, ax = plt.subplots(figsize=(6, 3))
ax.plot(x, y, marker="o", label="이름")   # 선
ax.scatter(x, y, alpha=0.6)                  # 산점도 (alpha: 투명도)
ax.hist(값, bins=20)                         # 히스토그램
ax.set_title("제목"); ax.set_xlabel("x"); ax.set_ylabel("y")
ax.legend()                                  # 범례 (label을 준 것만)
plt.show()

sns.boxplot(x="그룹열", y="값열", data=df, ax=ax)   # 그룹별 분포

# 상관계수: -1 ~ 1
df["열1"].corr(df["열2"])

# 난수 (같은 seed면 항상 같은 값)
rng = np.random.default_rng(40)
rng.uniform(최소, 최대, 개수)        # 균등하게
rng.normal(평균, 표준편차, 개수)     # 정규분포
rng.choice(n, 5, replace=False)    # 0~n-1에서 중복 없이 5개

# 선형회귀
from sklearn.linear_model import LinearRegression
model = LinearRegression()
model.fit(X, y)                    # X: 표(2차원), y: 열 하나
model.coef_[0], model.intercept_   # 기울기 w, 절편 b
model.predict(새_X)                # 예측

# 평가
from sklearn.metrics import mean_absolute_error, mean_squared_error, root_mean_squared_error, r2_score
mean_absolute_error(y_실제, y_예측)   # 두 배열의 길이가 같아야 한다

# 저장
df.to_csv("경로.csv", index=False)   # index=False: 행 번호는 저장하지 않음
```

## 3. 코드

### 1) `01_ml.ipynb` 추가 셀: 박스플롯

```python
import numpy as np
import pandas as pd
import seaborn as sns # 통계 그래프 라이브러리
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(10, 3))

sns.boxplot(x="class", y="fare", data=df, ax=ax, hue="class", palette="Set2", legend=False)
```

### 2) `02_ml.ipynb`

```python
# [셀 1] 선 그래프
import numpy as np
import pandas as pd
import seaborn as sns # 통계 그래프 라이브러리
import matplotlib.pyplot as plt

# 한글 폰트 설정
#plt.rcParams["font.family"] = "Malgun Gothic"   # Windows OS
plt.rcParams["font.family"] = "AppleGothic"     # 맥 OS

# Figure(캔버스), Axes(그래프)
fig, ax = plt.subplots(figsize=(6, 3))
# X축 데이터 (월)
months = ["1월", "2월", "3월", "4월", "5월", "6월"]
# Y축 데이터 (매출액, 만원)
sales = [120, 135, 120, 160, 170, 192]

# 선 그래프
ax.plot(months, sales, marker="o", label="매출")
ax.set_title("My Mall 월 매출")     # 그래프 제목
ax.set_xlabel("월")
ax.set_ylabel("매출(만워)")
ax.legend()                         # 범례 표시
plt.show()                          # 차트 그리기

# [셀 2] 미국 주택 데이터
house = pd.read_csv('./datasets/usa_housing.csv')
print(house.shape)
house.describe()

# [셀 3] 조건으로 10채 고르기
small = house[(house["YearBuilt"] > 2015) & (house["CrimeRate"] < 40)]
small.shape
small

# [셀 4] 산점도와 상관계수
import matplotlib.pyplot as plt

plt.figure(figsize=(8,4))
plt.scatter(house['SquareFeet'], house['Price'], alpha=0.4, label="전체 300채")
plt.scatter(small["SquareFeet"], small['Price'], color="red", s=50, label="2015이후 건축")
plt.xlabel("면적")
plt.ylabel("가격")
plt.legend()
plt.show()


print("전체 300채의 상관계수:", round(house["SquareFeet"].corr(house["Price"]),2))
print("10채의 상관계수:", round(small["SquareFeet"].corr(small["Price"]),2))

# [셀 5] 가상 오피스텔 데이터 만들기
# 시드(seed) : 난수 발생시 참고하는 데이터, 동일한 seed 값을 적용하는 
# 항상 동일한 난수가 발생
rng = np.random.default_rng(40)

n = 200
area = rng.uniform(20, 85, n) # 20 이상 85 미만에서 균등하게 n개 만큼 난수 발생

# 선형회귀
# y = wx + b
# w: 기울기 450
# b: 절편(편향) 5000
price = (area * 450 + 5000 + rng.normal(0, 2500, n)).round(-1)

# 특이 매물 5건 생성
special = rng.choice(n, 5, replace=False) # replace = False 같은 번호를 중복해서 뽑지 않는다.
price[special] += rng.uniform(15000, 25000, 5).round(-1)

office = pd.DataFrame({'area':area, "price": price})
office.describe().round(1)

# [셀 6] 파일 저장
office.to_csv("./datasets/office.csv", index=False)

# [셀 7] 산점도 차트
# 캔버스(Figure) 생성
plt.figure(figsize=(8,5))
# 산점도 차트
plt.scatter(office['area'], office["price"], alpha=0.6) # 알파는 점의 투명도
# 타이틀과 x, y 레이블
plt.title("오피스텔 시세 200건")
plt.xlabel("면적")
plt.ylabel("가격")
plt.show()

print("상관계수 : ", (office["area"].corr(office["price"])).round(3))

# [셀 8] 선형회귀 학습
from sklearn.linear_model import LinearRegression

X = office[["area"]]
y = office["price"]

# 선형 회귀 모델 만들기
model = LinearRegression()
# 학습 : X(입력값), y(정답)
model.fit(X, y)

print(f"기울기 w = {model.coef_[0].round(2)} 정답: 450")
print(f"절편 b = {model.intercept_.round(2)} 정답: 5000")

# [셀 9] 예측
# 33, 59 가격 예측
new = pd.DataFrame({"area":[33, 59]})
model.predict(new).round(-1)

# [셀 10] 모델의 직선 그리기
# 학습한 모델로 예측하기
y_pred = model.predict(X)

line = X.sort_values("area")
line_pred = model.predict(line)

# 캔버스(Figure) 생성
plt.figure(figsize=(8,5))
# 산점도 차트
plt.scatter(office['area'], office["price"], alpha=0.6, label="실제 시세") # 알파는 점의 투명도
# 모델의 직선
plt.plot(line["area"], line_pred, color="red", label="모델의 직선")

# 타이틀과 x, y 레이블
plt.title("오피스텔 시세 200건")
plt.xlabel("면적")
plt.ylabel("가격")
plt.show()

# [셀 11] 평가 지표
# MAE, MSE, RMSE
from sklearn.metrics import mean_absolute_error, mean_squared_error, root_mean_squared_error, r2_score
# MAE: 오차 절댓값의 평균 => 평균적으로 얼마만큼 틀리나?
print(f"MAE = {mean_absolute_error(y,y_pred):,.0f}")
# MSE: 오차 제곱의 평균 (단위 제곱)
print(f"MSE = {mean_squared_error(y,y_pred):,.0f}")
# RMSE: 오차 제곱 평균의 루트 (원래 단위)
print(f"RMSE = {root_mean_squared_error(y,y_pred):,.0f}")
# R SCORE: 0 ~ 1
print(f"R_SCORE={r2_score(y, y_pred):,.3f}")

# [셀 12] 특이매물 표시
normal = np.ones(len(office),dtype=bool)
normal[special] = False
print("특이매물", special)
print("정상매물", normal.sum())

# [셀 13] 전체 vs 특이매물 제외 비교
all_rows = np.ones(len(office), dtype=bool)
cases = [("전체 200건", all_rows), ("특이매물 제외한 195", normal)]

for name, mask in cases:
    y_true = y[mask]        # 실제가격
    y_hat = y_pred[mask]    # 예측가격

# MAE : 오차 절댓값의 평균
mae = mean_absolute_error(y_true, y_pred)
# RMSE
rmse = root_mean_squared_error(y_true, y_pred)
print(f"{name} MAE {mae} | RMSE {rmse}")
```

## 4. 핵심 정리

**Q. 그래프는 어떻게 그리지?**

- `fig, ax = plt.subplots()`로 캔버스(Figure)와 그래프 칸(Axes)을 만들고, `ax`에 그린 뒤 제목·축 이름을 붙인다.
- 보고 싶은 것에 따라 그래프를 고른다. 시간 흐름은 선, 한 값의 분포는 히스토그램, 두 값의 관계는 산점도, 그룹별 분포와 튀는 값은 박스플롯이다.
- `label`을 준 뒤 `legend()`를 부르면 범례가 나온다. 한글이 깨지지 않게 글꼴을 먼저 지정한다.

**Q. 상관계수는 뭘 말해 주지?**

- `corr`은 두 값이 함께 움직이는 정도를 -1~1로 나타낸다. 1에 가까우면 한쪽이 커질 때 다른 쪽도 커진다.
- 미국 주택 300채는 면적과 가격의 상관계수가 0.08이었는데, 조건으로 고른 10채만 보면 0.89였다. 표본이 작거나 골라낸 데이터만 보면 관계가 강해 보일 수 있다.
- 오피스텔 200건은 0.896이다. 면적으로 가격을 설명하기 좋은 데이터다.

**Q. 선형회귀는 뭘 하지?**

- 점들을 가장 잘 지나는 직선 `y = wx + b`를 찾는다. `fit`이 학습, `coef_`가 기울기 w, `intercept_`가 절편 b다.
- 학습 결과는 w = 439.46, b = 5926.68이다. 데이터를 만들 때 쓴 450, 5000과 가깝지만 같지 않다. 잡음과 특이매물 때문이다.
- `predict`로 새 면적의 가격을 예측한다. 33은 20,430, 59는 31,850이다. 입력도 학습 때와 같은 열 이름(`area`)의 표로 만든다.

**Q. seed는 왜 주지?**

- 난수는 실행할 때마다 바뀌지만, `default_rng(40)`처럼 seed를 고정하면 항상 같은 값이 나온다. 그래서 누가 돌려도 결과를 비교할 수 있다.
- 직접 같은 코드를 돌려 보니 저장된 `office.csv`와 값이 완전히 같았고, 특이매물 번호도 `[53 13 2 92 71]`로 같았다.
- `uniform`은 구간 안에서 고르게, `normal`은 평균 근처에 몰리게, `choice(..., replace=False)`는 같은 번호를 두 번 뽑지 않게 만든다.

**Q. MAE와 RMSE는 어떻게 다르지?**

- 둘 다 "평균적으로 얼마나 틀렸나"인데, RMSE는 오차를 제곱해서 더하므로 큰 오차에 훨씬 민감하다.
- 특이매물 5건을 빼고 계산하니 MAE는 2,493 → 2,075, RMSE는 4,030 → 2,732로 줄었다. 크게 틀린 5건이 빠지자 RMSE가 더 많이 줄었다.
- R² 0.803은 가격 차이의 약 80%를 면적으로 설명한다는 뜻이다.

## 5. 오늘 만난 오류

**`for`문 밖에서 길이가 다른 배열을 비교한 경우 (셀 13)**

```python
for name, mask in cases:
    y_true = y[mask]        # 실제가격
    y_hat = y_pred[mask]    # 예측가격

mae = mean_absolute_error(y_true, y_pred)
rmse = root_mean_squared_error(y_true, y_pred)
print(f"{name} MAE {mae} | RMSE {rmse}")
```

```txt
ValueError: Found input variables with inconsistent numbers of samples: [195, 200]
```

메시지는 "두 입력의 개수가 195와 200으로 다르다"는 뜻이다. 원인은 두 가지다.

- `mae`, `rmse`, `print` 줄이 들여쓰기가 없어 `for`문 밖에 있다. 반복이 끝난 뒤 한 번만 실행되니 `y_true`에는 마지막 경우(195건)만 남는다.
- 예측값으로 골라낸 `y_hat`(195건)이 아니라 전체 `y_pred`(200건)를 넣었다.

```python
# 고친 코드: 계산을 for문 안으로, y_pred → y_hat
for name, mask in cases:
    y_true = y[mask]        # 실제가격
    y_hat = y_pred[mask]    # 예측가격

    mae = mean_absolute_error(y_true, y_hat)
    rmse = root_mean_squared_error(y_true, y_hat)
    print(f"{name} MAE {mae:,.0f} | RMSE {rmse:,.0f}")
```

```txt
전체 200건 MAE 2,493 | RMSE 4,030
특이매물 제외한 195 MAE 2,075 | RMSE 2,732
```

## 6. 실행 예시

```python
small = house[(house["YearBuilt"] > 2015) & (house["CrimeRate"] < 40)]
print("전체 300채의 상관계수:", round(house["SquareFeet"].corr(house["Price"]),2))
print("10채의 상관계수:", round(small["SquareFeet"].corr(small["Price"]),2))
```

```txt
전체 300채의 상관계수: 0.08
10채의 상관계수: 0.89
```

```python
office.describe().round(1)
```

```txt
        area    price
count  200.0    200.0
mean    52.8  29138.4
std     18.6   9112.1
min     22.0  11400.0
25%     37.4  21615.0
50%     52.0  28965.0
75%     67.4  36377.5
max     84.9  58460.0
```

```python
print("상관계수 : ", (office["area"].corr(office["price"])).round(3))
```

```txt
상관계수 :  0.896
```

```python
model = LinearRegression()
model.fit(X, y)
print(f"기울기 w = {model.coef_[0].round(2)} 정답: 450")
print(f"절편 b = {model.intercept_.round(2)} 정답: 5000")
```

```txt
기울기 w = 439.46 정답: 450
절편 b = 5926.68 정답: 5000
```

```python
new = pd.DataFrame({"area":[33, 59]})
model.predict(new).round(-1)
```

```txt
array([20430., 31850.])
```

```python
print(f"MAE = {mean_absolute_error(y,y_pred):,.0f}")
print(f"MSE = {mean_squared_error(y,y_pred):,.0f}")
print(f"RMSE = {root_mean_squared_error(y,y_pred):,.0f}")
print(f"R_SCORE={r2_score(y, y_pred):,.3f}")
```

```txt
MAE = 2,493
MSE = 16,240,971
RMSE = 4,030
R_SCORE=0.803
```

```python
normal = np.ones(len(office),dtype=bool)
normal[special] = False
print("특이매물", special)
print("정상매물", normal.sum())
```

```txt
특이매물 [53 13  2 92 71]
정상매물 195
```

그래프 출력은 노트북에 이미지로 있다. 선 그래프(월 매출), 산점도(주택 300채 위에 빨간 점 10채), 산점도와 빨간 모델 직선(오피스텔 200건), 객실 등급별 요금 박스플롯이다.
