## 1. 오늘의 흐름

공부시간으로 점수를 예측하는 작은 선형회귀를 만들고 평가했다. 그다음 `LinearRegression`이 한 번에 구한 답(w=10, b=35)을 경사하강법이 조금씩 고쳐 가며 찾아가는 과정을 기록으로 확인했고, 마지막으로 입력이 여러 개인 다중 선형회귀를 위해 상관관계부터 봤다.

1. `LinearRegression`으로 학습하고 예측한 뒤 MAE·MSE·R²로 평가하기
2. `SGDRegressor`와 `partial_fit`으로 1000 epoch 동안 w, b, MSE가 어떻게 바뀌는지 기록하기
3. 다중 선형회귀 데이터의 산점도 3개와 상관계수 히트맵 보기

## 2. 구조

```txt
경사하강법 1 epoch (partial_fit 한 번)

  현재 w, b로 예측  →  정답과 비교해 오차 계산 (손실 함수 = MSE)
        ↑                          ↓
  w, b를 조금 이동  ←  오차가 줄어드는 방향(기울기) 찾기
                       이동 폭 = 학습률 eta0 (0.01)

이 한 바퀴를 1000번 반복 → w ≈ 10, b ≈ 35 로 다가간다
```

| epoch | weight | bias | MSE |
| --- | --- | --- | --- |
| 1 | 8.455 | 2.731 | 1369.82 |
| 10 | 17.061 | 7.616 | 141.38 |
| 100 | 13.295 | 22.401 | 32.28 |
| 500 | 10.224 | 34.373 | 3.30 |
| 1000 | 10.129 | 34.741 | 3.25 |
| 목표 (`LinearRegression`) | 10.00 | 35.00 | 3.20 |

MSE가 1369에서 3.25까지 줄면서 w, b가 `LinearRegression`의 답에 거의 도착했다.

### 문법 사용법

```py
# 선형회귀: 한 번에 답을 계산
from sklearn.linear_model import LinearRegression
model = LinearRegression()
model.fit(X, y)                     # X: df[["열"]] (2차원), y: df["열"] (1차원)
model.coef_[0], model.intercept_    # 기울기 w, 절편 b
model.predict(pd.DataFrame({"hours": [6]}))

# 평가
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
mean_absolute_error(y, y_pred)      # 오차 절댓값 평균
mean_squared_error(y, y_pred)       # 오차 제곱 평균
r2_score(y, y_pred)                 # 결정계수, 1에 가까울수록 좋다

# 확률적 경사하강법: 조금씩 고쳐 가며 답을 찾음
from sklearn.linear_model import SGDRegressor
sgd = SGDRegressor(loss="squared_error", learning_rate="constant",
                   eta0=0.01, max_iter=1000, shuffle=False, random_state=42)
sgd.partial_fit(X, y)               # 데이터 전체를 1번 학습 (1 epoch)
sgd.coef_[0], sgd.intercept_[0]     # 절편도 배열이라 [0]

# 조건에 맞는 행만 보기
history_df[history_df["epoch"].isin([1, 5, 10])]

# 여러 그래프를 한 줄에
fig, axes = plt.subplots(1, 3, figsize=(16, 5))
sns.scatterplot(data=df, x="열1", y="열2", ax=axes[0])

# 상관계수 히트맵
sns.heatmap(df.corr(), annot=True, vmin=-1, vmax=1, fmt=".2f")
```

## 3. 코드

### 1) `03_ml.ipynb` 선형회귀 실습

```py
import pandas as pd
import matplotlib.pyplot as plt

# 선형회귀 모델 임포트
from sklearn.linear_model import LinearRegression

# 한글 폰트 설정
# plt.rcParams["font.family"] = "Malgun Gothic"   # 윈도우 사용자
plt.rcParams["font.family"] = "AppleGothic"     # 맥 사용자

# 데이터 (딕셔너리)
data = {
    "hours": [1, 2, 3, 4, 5],
    "scores": [47, 53, 65, 73, 87]
}

# 판다스 데이터프레임으로 전환
df = pd.DataFrame(data)

# 입력 데이터 (피쳐:Feature) : 2차원 - 독립변수
X = df[["hours"]]   # 공부시간

# 출력 데이터 (Target) : 1차원 - 정답 : 종속변수
y = df["scores"]    # 시험점수

# 선형회귀 모델 객체 생성
model = LinearRegression()
# 모델 학습
model.fit(X, y)

# 다중 선형회귀의 경우 결과값이 배열로 리턴
print(f"기울기: {model.coef_[0]:.2f}")  # 기울기 Weight
print(f"절편: {model.intercept_:.2f}")  # 절편 Bias

# 6시간 학습한 학생의 성적 예측
new_data = pd.DataFrame({"hours": [6]}) # X 독립변수 1개 추가
predicted_score = model.predict(new_data)
print(f"6시간 학습한 학생의 예상 성적: {predicted_score[0]:.2f}점")

# 학습 데이터에 대한 예측값
y_pred = model.predict(X)
print("학습 데이터에 대한 예측값:", y_pred)

plt.figure(figsize=(6, 3))
plt.scatter(X["hours"], y, color="blue", label="실제 성적")          # 실제 성적 (산점도)
plt.plot(X["hours"], y_pred, color="red", label="예측 성적")          # 예측한 성적 (회귀직선)
plt.scatter(6, predicted_score[0], color="green", label="6시간 예측 성적")
plt.title("공부시간 대비 성적 예측")
plt.xlabel("공부 시간")
plt.ylabel("성적")
plt.legend()
plt.show()

# 모델 평가
from sklearn.metrics import mean_absolute_error
from sklearn.metrics import mean_squared_error
from sklearn.metrics import r2_score

# MAE : 오차 절대 평균
mae = mean_absolute_error(y, y_pred)
print(f"학습데이터에 대한 평균 절대 오차(MAE): {mae:.2f}")

# MSE : 오차 제곱 평균 (큰 오차에 민감)
mse = mean_squared_error(y, y_pred)
print(f"학습데이터에 대한 평균 제곱 오차(MSE): {mse:.2f}")

# R2 Score : 결정계수 (0 ~ 1, 음수면 학습 실패)
r2 = r2_score(y, y_pred)
print(f"학습데이터에 대한 결정 계수(R^2): {r2:.2f}")
```

### 2) `03_ml.ipynb` 경사하강법

```py
from sklearn.linear_model import LinearRegression
# 배치 경사하강법 (BGD) : 전체 데이터를 사용, 시간, 리소스 많이 사용
# 확률적 경사하강법 (SGD) : 1개의 데이터씩 보며 하강
from sklearn.linear_model import SGDRegressor
from sklearn.metrics import mean_squared_error

df = pd.DataFrame({
    "hours": [1, 2, 3, 4, 5],
    "scores": [47, 53, 65, 73, 87]
})
X = df[["hours"]]   # 독립변수 (Feature)
y = df["scores"]    # 종속변수 (Target)

# 경사하강법으로 도달하려는 목표
model = LinearRegression()
model.fit(X, y)
y_pred = model.predict(X)
print(f"Weight: {model.coef_[0]:.2f}")
print(f"Bias: {model.intercept_:.2f}")
print(f"MSE: {mean_squared_error(y, y_pred):.4f}")

# 확률적 경사하강법 모델
sgd = SGDRegressor(
    loss="squared_error",       # 제곱 오차 기반 손실함수
    learning_rate="constant",   # 학습률을 일정하게 유지
    eta0=0.01,                  # 초기 학습률
    max_iter=1000,              # fit() 반복 횟수
    shuffle=False,              # 학습 데이터 섞지 않음
    random_state=42             # 난수 시드 설정
)

epochs = 1000
history = []    # 학습과정 저장 리스트

for epoch in range(epochs):
    sgd.partial_fit(X, y)                   # 전체 데이터 1회 학습 (1 epoch)
    y_pred = sgd.predict(X)                 # 1회 학습 후 예측
    mse = mean_squared_error(y, y_pred)     # 현재 모델의 MSE
    history.append({
        "epoch": epoch + 1,
        "weight": sgd.coef_[0],
        "bias": sgd.intercept_[0],
        "mse": mse
    })

history_df = pd.DataFrame(history)
history_df[ history_df["epoch"].isin([1,5,10,50,100,300,500,1000]) ]

# 손실함수 차트
plt.figure(figsize=(10,5))
plt.plot(history_df["epoch"], history_df["mse"], color="blue", label="학습 손실(Traing Loss)")
plt.xlabel("Loss")
plt.ylabel("MSE")
plt.grid(alpha=0.3)
plt.show()

# 기울기와 절편 변화
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
axes[0].plot(history_df["epoch"], history_df["weight"], color="blue")
axes[0].grid(alpha=0.3)
axes[0].set_ylabel("Weight")
axes[1].plot(history_df["epoch"], history_df["bias"], color="blue")
axes[1].grid(alpha=0.3)
axes[1].set_ylabel("Bias")
plt.show()

# 최종 모델로 시험 성적 예측
print(f"기울기(w): {sgd.coef_[0]:.2f}")
print(f"절편(b): {sgd.intercept_[0]:.2f}")
new_data = pd.DataFrame({"hours": [6]})
prediction = sgd.predict(new_data)
print(f"예측 성적: {prediction[0]:.2f}점")
```

### 3) `03_ml.ipynb` 다중 선형회귀: 상관관계 확인

```py
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.DataFrame({
    "study_hours":    [1, 2, 3, 4, 5, 2, 3, 5, 4, 1],
    "sleep_hours":    [6, 8, 5, 7, 8, 5, 8, 6, 6, 8],
    "practice_count": [10, 15, 20, 25, 30, 12, 28, 22, 32, 8],
    "score":          [47, 58, 60, 76, 87, 49, 73, 77, 79, 52]
})

# 1행 3열의 그래프 영역 생성
fig, axes = plt.subplots(1, 3, figsize=(16,5))
sns.scatterplot(data=df, x="study_hours", y="score", ax=axes[0])     # 공부시간과 성적
sns.scatterplot(data=df, x="sleep_hours", y="score", ax=axes[1])     # 수면시간과 성적
sns.scatterplot(data=df, x="practice_count", y="score", ax=axes[2])  # 문제풀이 개수와 성적
plt.show()

# 모든 컬럼 사이의 상관계수
corr = df.corr()
plt.figure(figsize=(7,5))
sns.heatmap(
    corr,
    annot=True,     # 상관계수 표시
    vmin=-1,        # 최소값
    vmax=1,         # 최대값
    fmt=".2f"       # 소숫점 두자리
)
plt.show()
```

## 4. 핵심 정리

**Q. X는 왜 대괄호 두 개고 y는 하나지?**

- X는 표(2차원), y는 열 하나(1차원)다. 모델은 특성이 하나여도 입력을 표로 받는다.

**Q. 평가 지표는 어떻게 읽지?**

- MAE 1.60은 평균 1.6점 틀린다는 뜻이고, MSE 3.20은 오차를 제곱해 평균낸 값이다.
- R² 0.98은 점수 차이의 98%를 공부시간으로 설명한다는 뜻이다.

**Q. 경사하강법은 뭘 하지?**

- 손실 함수(MSE)가 작아지는 방향으로 w, b를 조금씩 옮긴다. 옮기는 폭이 학습률(`eta0`)이다.
- BGD는 전체 데이터를 보고 한 번, SGD는 데이터 1개마다 옮긴다.

**Q. `partial_fit`은 뭐지?**

- 한 번 부를 때마다 데이터 전체를 1바퀴(1 epoch) 학습한다. 그래서 `for`문으로 1000번 부르며 w, b, MSE를 기록했다.
- 1000 epoch 뒤 w = 10.13, b = 34.74로 `LinearRegression`의 답(10, 35)에 거의 도착했다.

**Q. 다중 선형회귀 전에 상관계수를 왜 보지?**

- 점수와 함께 움직이는 입력을 먼저 찾기 위해서다. study_hours와 practice_count가 0.93, sleep_hours는 0.29다.

## 5. 오늘 만난 오류

**히트맵에서 마이너스 기호 글꼴 경고 (상관 계수 확인 셀)**

```py
plt.rcParams["font.family"] = "AppleGothic"
sns.heatmap(corr, annot=True, vmin=-1, vmax=1, fmt=".2f")
plt.show()
```

```txt
UserWarning: Glyph 8722 (\N{MINUS SIGN}) missing from font(s) AppleGothic.
```

- Glyph 8722는 유니코드 U+2212 마이너스 기호(−)다. matplotlib는 축 눈금의 음수를 기본으로 이 기호로 쓰는데, AppleGothic에는 이 글자가 없다는 경고다.
- `vmin=-1`이라 오른쪽 색 막대 눈금에 -1.00, -0.75 같은 음수가 생겨서 나왔다. 그림은 그려지지만 마이너스가 깨져 보일 수 있다.
- 직접 해보니 아래 한 줄을 넣으면 색 막대 눈금이 U+2212 대신 일반 하이픈(-)으로 바뀌었다.

```py
# 고친 코드
plt.rcParams["font.family"] = "AppleGothic"
plt.rcParams["axes.unicode_minus"] = False   # 음수 기호를 일반 하이픈으로

sns.heatmap(corr, annot=True, vmin=-1, vmax=1, fmt=".2f")
plt.show()
```

**손실함수 차트의 x축 이름**

```py
plt.plot(history_df["epoch"], history_df["mse"], ...)
plt.xlabel("Loss")
```

- 에러는 아니지만 x축에 넣은 값은 epoch이고, Loss(MSE)는 y축이다.

```py
# 고친 코드
plt.xlabel("Epoch")
plt.ylabel("MSE (Loss)")
```

## 6. 실행 예시

```py
print(f"기울기: {model.coef_[0]:.2f}")
print(f"절편: {model.intercept_:.2f}")
```

```txt
기울기: 10.00
절편: 35.00
```

```py
new_data = pd.DataFrame({"hours": [6]})
predicted_score = model.predict(new_data)
print(f"6시간 학습한 학생의 예상 성적: {predicted_score[0]:.2f}점")

y_pred = model.predict(X)
print("학습 데이터에 대한 예측값:", y_pred)
```

```txt
6시간 학습한 학생의 예상 성적: 95.00점
학습 데이터에 대한 예측값: [45. 55. 65. 75. 85.]
```

```py
print(f"학습데이터에 대한 평균 절대 오차(MAE): {mae:.2f}")
print(f"학습데이터에 대한 평균 제곱 오차(MSE): {mse:.2f}")
print(f"학습데이터에 대한 결정 계수(R^2): {r2:.2f}")
```

```txt
학습데이터에 대한 평균 절대 오차(MAE): 1.60
학습데이터에 대한 평균 제곱 오차(MSE): 3.20
학습데이터에 대한 결정 계수(R^2): 0.98
```

```py
history_df[ history_df["epoch"].isin([1,5,10,50,100,300,500,1000]) ]
```

```txt
     epoch     weight       bias          mse
0        1   8.455225   2.731163  1369.816039
4        5  16.728403   6.216494   167.673523
9       10  17.060518   7.615610   141.377004
49      50  15.031795  15.629821    72.111791
99     100  13.294764  22.401342    32.278720
299    300  10.678818  32.599245     4.254411
499    500  10.223920  34.372564     3.302214
999   1000  10.129335  34.741238     3.250205
```

```py
print(f"기울기(w): {sgd.coef_[0]:.2f}")
print(f"절편(b): {sgd.intercept_[0]:.2f}")
print(f"예측 성적: {prediction[0]:.2f}점")
```

```txt
기울기(w): 10.13
절편(b): 34.74
예측 성적: 95.52점
```

```py
df.corr().round(2)
```

```txt
                study_hours  sleep_hours  practice_count  score
study_hours            1.00         0.00            0.85   0.93
sleep_hours            0.00         1.00            0.14   0.29
practice_count         0.85         0.14            1.00   0.93
score                  0.93         0.29            0.93   1.00
```

그래프 출력은 노트북에 이미지로 있다. 공부시간 대비 성적 산점도와 빨간 회귀직선, epoch별 MSE 곡선, weight·bias 변화 곡선 2개, 산점도 3개, 상관계수 히트맵이다.
