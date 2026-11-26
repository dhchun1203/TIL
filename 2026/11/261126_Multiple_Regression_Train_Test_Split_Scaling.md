# Multiple Regression, Train/Test Split & Scaling

특징 셋으로 다중 선형회귀를 학습하고, 7명과 3명으로 나눠 처음 보는 데이터로 채점했다. 단위가 다른 특징을 StandardScaler로 맞춰 SGDRegressor로 다시 풀고, KNN을 위해 얼굴 사진 데이터를 열어 봤다.

## 1. 오늘의 흐름

어제 그려 본 학생 10명 데이터(공부 시간, 수면 시간, 문제 풀이 개수)로 다중 선형회귀를 학습했다. 학습한 데이터로 채점하면 점수가 너무 좋게 나와서, 7명으로 배우고 처음 보는 3명으로 채점하는 법을 배웠다. 특징마다 단위가 달라 경사하강법으로 측정하기 어려우니 표준화로 맞춘 뒤 `SGDRegressor`로 다시 풀었다. 끝으로 KNN을 위해 얼굴 사진 데이터를 열어 봤다.

1. 다중 선형회귀: 특징 셋 → 가중치 셋 `[5.80, 2.68, 0.63]`, 절편 17.66, 새 학생 78.60점
2. 학습한 데이터로 채점: MAE 1.17, R² 0.99
3. `train_test_split`으로 7명, 3명 나누기 → 처음 보는 3명으로 채점하면 MAE 3.09, R² 0.93
4. `StandardScaler`로 특징마다 평균 0, 표준편차 1
5. 표준화한 데이터로 `SGDRegressor` → MAE 2.82, R² 0.95, 500 epoch 기록
6. KNN 준비: 얼굴 사진 400장(40명 × 10장), 한 장은 64 × 64 = 4096개의 0\~1 숫자

## 2. 구조

```txt
데이터          학생 10명, 특징 3개 → score
입력 / 정답     X = df[features]  (10, 3)
                y = df["score"]   (10,)

다중 회귀       ŷ = w1 × 공부 + w2 × 수면 + w3 × 문제 + b
                LinearRegression().fit(X, y)
                → coef_ = [5.80, 2.68, 0.63], intercept_ = 17.66

나누기          train_test_split(X, y, test_size=0.3, random_state=42)
                학습용 7명 → fit()
                시험용 3명 (행 8, 1, 5) → predict()로 채점

표준화          scaler.fit_transform(X_train)   평균, 표준편차를 재고 바꾸기
                scaler.transform(X_test)        학습용 기준으로 바꾸기만
                z = (x - 평균) / 표준편차

경사하강법      SGDRegressor(eta0=0.01).fit(X_train_scaled, y_train)
                → 197 epoch에서 멈춤

얼굴 사진       faces.images (400, 64, 64)   사진
                faces.data   (400, 4096)     한 줄로 편 사진
                faces.target (400,)          사람 번호 0~39
```

## 3. 코드

### 다중 선형회귀

```python
features = ["study_hours", "sleep_hours", "practice_count"]

# 입력은 열 세 개짜리 표, 정답은 열 하나
X = df[features]
y = df["score"]

model = LinearRegression()
model.fit(X, y)

# 가중치 셋(features 순서), 절편 하나
print("가중치", model.coef_)
print("절편", model.intercept_)

new_data = pd.DataFrame({
    "study_hours": [4],
    "sleep_hours": [7],
    "practice_count": [30]
})
prediction = model.predict(new_data)
```

predict가 하는 일을 손으로 하면 이렇다.

```python
w1, w2, w3 = model.coef_
b = model.intercept_

result = w1 * 4 + w2 * 7 + w3 * 30 + b
```

### 예측값과 오차를 표에 붙이기

```python
y_pred = model.predict(X)

# 원본을 건드리지 않게 복사본에
result_df = df.copy()

# 없는 열 이름에 넣으면 새 열이 생긴다
result_df["예측값"] = y_pred
result_df["예측오차"] = y - y_pred

result_df.round(2)
```

### 학습용과 시험용 나누기

```python
from sklearn.model_selection import train_test_split

# 돌려받는 순서: X 학습, X 시험, y 학습, y 시험
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.3,
    random_state=42
)

test_model = LinearRegression()

# 7명으로만 배운다
test_model.fit(X_train, y_train)

# 처음 보는 3명으로 채점
test_pred = test_model.predict(X_test)
print(mean_absolute_error(y_test, test_pred))
```

### 표준화

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()

# 학습용으로 평균, 표준편차를 재고 바로 바꾼다
X_train_scaled = scaler.fit_transform(X_train)

# 시험용은 학습용의 평균, 표준편차로 바꾸기만
X_test_scaled = scaler.transform(X_test)

# 재 둔 값
print(scaler.mean_)
print(scaler.scale_)
```

### 경사하강법으로 다중 회귀

```python
sgd = SGDRegressor(
    loss="squared_error",
    learning_rate="constant",
    eta0=0.01,
    max_iter=3000,
    random_state=42
)

# 표준화한 데이터로 배우고, 표준화한 데이터로 예측
sgd.fit(X_train_scaled, y_train)
sgd_pred = sgd.predict(X_test_scaled)

# 실제로 돈 바퀴 수
print(sgd.n_iter_)
```

500 epoch 기록은 어제처럼 `partial_fit`을 `for`문으로 부르고, 가중치를 `coef_[0]`, `coef_[1]`, `coef_[2]`로 나눠 담았다.

```python
# max_iter 없이 새로 만든다
sgd = SGDRegressor(
    loss="squared_error",
    learning_rate="constant",
    eta0=0.01,
    random_state=42
)

history = []

for epoch in range(500):
    sgd.partial_fit(X_train_scaled, y_train)
    pred = sgd.predict(X_train_scaled)

    history.append({
        "epoch": epoch + 1,
        "w1": sgd.coef_[0],
        "w2": sgd.coef_[1],
        "w3": sgd.coef_[2],
        "bias": sgd.intercept_[0],
        "mse": mean_squared_error(y_train, pred)
    })

history_df = pd.DataFrame(history)
```

### 가중치 세 개를 한 그래프에

```python
fig, ax = plt.subplots(figsize=(10, 5))

ax.plot(history_df["epoch"], history_df["w1"], label="study")
ax.plot(history_df["epoch"], history_df["w2"], label="sleep")
ax.plot(history_df["epoch"], history_df["w3"], label="count")

plt.legend()
plt.grid(alpha=0.3)
plt.show()
```

### 얼굴 사진 데이터

```python
from sklearn.datasets import fetch_olivetti_faces

# 처음 한 번 받아 ./datasets에 저장
faces = fetch_olivetti_faces(data_home="./datasets")

plt.imshow(faces.images[0], cmap="gray", vmin=0, vmax=1)
plt.title(f"Person {faces.target[0]}")
plt.axis("off")
plt.show()

# 사진의 일부(8 × 8)를 숫자로
corner = faces.images[0][20:28, 20:28]
print(np.round(corner, 2))
```

```python
def show_person(person):
    # 그 사람의 사진만 고른다
    pics = faces.images[faces.target == person]

    fig, axes = plt.subplots(1, 10, figsize=(15, 2))
    for ax, img in zip(axes, pics):
        ax.imshow(img, cmap="gray", vmin=0, vmax=1)
        ax.axis("off")
    plt.show()
```

## 4. 핵심 정리

| 개념 | 내 말로 |
| --- | --- |
| 다중 선형회귀 | 특징이 여러 개. 특징마다 가중치가 하나씩 생기고 절편은 하나 |
| 가중치 (weight) | 그 특징이 1 오를 때 예측이 얼마나 오르나. 공부 1시간 = 5.80점 |
| 학습 / 시험 데이터 | 배우는 데 쓰는 것 / 처음 보는 척 채점하는 데 쓰는 것. 학습한 데이터로 채점하면 점수가 부풀려진다 |
| `test_size=0.3` | 30%를 시험용으로. 10명이면 3명 |
| `random_state` | 누가 몇 번 돌려도 같은 사람이 같은 쪽에 뽑히게 하는 seed |
| 특성 스케일링 | 단위와 범위가 다른 특징을 비슷한 크기로 맞추는 전처리 |
| 표준화 (`StandardScaler`) | (값 - 평균) ÷ 표준편차. 평균 0, 표준편차 1로 |
| `fit_transform` / `transform` | 재고 바꾸기 / 재 둔 기준으로 바꾸기만. 시험용에는 `transform`만 |
| 픽셀 | 사진의 점 하나. 흑백 사진은 0(검정)\~1(흰색) 숫자 하나 |

- 손 계산: `5.80 × 4 + 2.68 × 7 + 0.63 × 30 + 17.66 = 78.60`. `predict`도 이 식을 계산할 뿐이다
- 같은 모델이 학습한 10명으로는 MAE 1.17, R² 0.99인데, 처음 보는 3명으로는 MAE 3.09, R² 0.93이다. 시험용이 진짜 실력이다. 다만 3명뿐이라 `random_state`를 0으로 바꾸면 MAE 1.46이 나올 만큼 흔들린다
- 특징 범위가 공부 1\~5, 수면 5\~8, 문제 8\~32로 다르다. 가중치 0.63(문제 1개당)이 작아 보여도 문제 수는 범위가 넓어서 영향이 작은 게 아니다. 표준화한 데이터로 학습하면 가중치가 9.97, 3.63, 3.24라 같은 자로 비교할 수 있다
- 표준화한 데이터로는 절편이 67.43이 된다. 특징이 모두 평균(0)인 학생의 예측이라 학습용 7명의 평균 점수와 같다
- 표준화 없이 `SGDRegressor`에 원래 숫자를 넣으면 가중치가 `3.37e+10`(백억 단위)까지 튀고 6바퀴 만에 멈춘다. 범위가 큰 문제 수 열이 한 걸음을 너무 크게 만든다
- 500 epoch의 SGD(9.93, 3.62, 3.29, 67.42)가 `LinearRegression`의 답(9.97, 3.63, 3.24, 67.43)에 거의 닿았다. 손실은 1 epoch 4084에서 84 epoch에 1 아래로, 500 epoch에 0.14
- 사진 한 장은 64 × 64 = 4096개의 숫자다. KNN은 이 숫자 줄끼리 얼마나 가까운지로 같은 사람을 찾을 것이다

## 5. 헷갈린 점, 고친 점

- `describe()`의 표준편차(공부 1.49)와 `StandardScaler`가 쓰는 표준편차가 다르다. `describe()`는 n-1로 나누고(표본 표준편차), `StandardScaler`는 n으로 나눈다(10명 전체로 하면 1.41). 게다가 scaler는 학습용 7명으로만 재서 1.55다
- 시험용에 `fit_transform`을 쓰면 안 된다. 시험용 3명의 평균으로 다시 재면 학습 때와 다른 자로 바꾸는 셈이다. 시험용은 처음 보는 데이터라 기준을 재는 데 쓰지 않는다
- 표준화한 데이터로 배운 모델에 원래 숫자(공부 4, 수면 7, 문제 30)를 그대로 넣으면 엉뚱한 값이 나온다. 500 epoch 모델로 해 보니 231.15점, `scaler.transform()`을 거쳐 넣으면 77.36점이었다. 새 입력도 반드시 같은 scaler를 거친다
- `max_iter=3000`으로 줬는데 `fit()`은 197바퀴에서 멈췄다. 손실이 더 줄지 않으면 스스로 멈춘다. 몇 바퀴 돌았는지는 `sgd.n_iter_`
- 수업 노트에 StandardScaler 결과를 "-1 \~ 0 \~ +1"로 적었는데 범위가 정해진 게 아니다. 평균이 0, 표준편차가 1일 뿐이고 학습용 7명에서 실제 값은 -1.65부터 1.21까지, 시험용에서는 1.47도 나왔다
- `show_person`은 수업에서 그리는 줄까지만 쳤다. `ax.axis("off")`와 `plt.show()`를 붙여야 깔끔하게 한 줄로 나온다
- 학습 칸을 두 번 실행하면 `partial_fit`이 이어서 배워 숫자가 달라진다. 모델 만드는 칸부터 다시

## 6. 실행 예시

```txt
가중치 [5.80222725 2.68100001 0.63219852]
절편 17.660208080206615
예측 성적: 78.60점
직접 계산: 78.6021
predict 계산: 78.6021
```

```txt
   study_hours  sleep_hours  practice_count  score    예측값  예측오차
0            1            6              10     47  45.87  1.13
1            2            8              15     58  60.20 -2.20
2            3            5              20     60  61.12 -1.12
3            4            7              25     76  75.44  0.56
4            5            8              30     87  87.09 -0.09
5            2            5              12     49  50.26 -1.26
6            3            8              28     73  74.22 -1.22
7            5            6              22     77  76.67  0.33
8            4            6              32     79  77.19  1.81
9            1            8               8     52  49.97  2.03
```

```txt
학습한 10명으로 채점   MAE: 1.1739  MSE: 1.8252   R2: 0.9900
처음 보는 3명 (LR)     MAE: 3.0870  MSE: 10.6380  R2: 0.9327
처음 보는 3명 (SGD)    MAE: 2.8169  MSE: 8.5966   R2: 0.9456
```

```txt
   study_hours  sleep_hours  practice_count
0        -1.38        -0.76           -1.32
1         1.20        -0.76            0.20
2        -0.09        -1.65           -0.05
3        -1.38         1.02           -1.58
4         1.20         1.02            1.21
5         0.55         0.13            0.58
6        -0.09         1.02            0.96
```

```txt
     epoch    w1    w2    w3   bias   mse
495    496  9.92  3.62  3.29  67.42  0.14
496    497  9.92  3.62  3.29  67.42  0.14
497    498  9.92  3.62  3.29  67.42  0.14
498    499  9.93  3.62  3.29  67.42  0.14
499    500  9.93  3.62  3.29  67.42  0.14
```

```txt
사진 묶음 크기 :  (400, 64, 64)
정답 묶음 크기 :  (400,)
사람 종류 :  [ 0  1  2 ... 39]
[[0.57 0.58 0.61 0.57 0.54 0.55 0.59 0.64]
 [0.6  0.6  0.58 0.56 0.55 0.57 0.6  0.65]
 ...
 [0.69 0.69 0.7  0.71 0.71 0.72 0.7  0.7 ]]
가장 어두운 값:  0.0
가장 밝은 값:  1.0
```

---



