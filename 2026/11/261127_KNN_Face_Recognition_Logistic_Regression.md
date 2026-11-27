# KNN Face Recognition & Logistic Regression

얼굴 사진 400장으로 KNN 이웃 투표(K에 따라 답이 바뀜, 정확도 87%), 60점 기준 합격을 로지스틱 회귀로 예측(합격 확률 92.58%)하고 혼동 행렬로 채점

## 1. 오늘의 흐름

어제 숫자 표로 열어 본 얼굴 사진 400장으로 KNN을 학습했다. 229번 사진 한 장을 빼 두고, 나머지 399장 중 가장 가까운 사진 K장에게 투표를 받아 누구인지 맞혔다. 처음 보는 100장으로 채점한 다음, 같은 학생 10명 데이터를 60점 기준 합격, 불합격으로 바꿔 로지스틱 회귀로 넘어갔다.

1. 40명 첫 사진을 4 × 10 칸에 → 사람마다 얼굴 한 장씩
2. KNN으로 229번 사진 맞히기 → 이웃 `[17 22 22]`, 예측 22, 정답 22 (K = 3)
3. 이웃 사진 그리기 → `reshape(64, 64)`로 다시 사진으로
4. KNN 채점 → 300장 학습, 100장 시험, 정확도 87%
5. 로지스틱 회귀 → 새 학생 합격 확률 92.58%, 시험용 3명 정확도 100%

## 2. 구조

```txt
얼굴 데이터     faces.images (400, 64, 64)   사진 모양 그대로
                faces.data   (400, 4096)     한 장을 한 줄로 편 것 = X
                faces.target (400,)          사람 번호 0~39 = y

KNN 한 장       known = np.arange(400) != 229  → 399장으로 학습
                kneighbors → dist(거리), idx(X_known 안 자리)
                predict    → K장의 다수결, 동점이면 번호가 작은 쪽

KNN 채점        train_test_split(test_size=0.25, stratify=y)
                300장 학습 / 100장 시험 → score = 0.87

로지스틱 회귀   y = df["score"] >= 60     (True 6명, False 4명)
                split(stratify=y) → StandardScaler → LogisticRegression
                z = w·x + b → 시그모이드 → 0~1 확률 → 0.5 이상이면 True
채점            accuracy_score, confusion_matrix(행 실제, 열 예측)
```

## 3. 코드

### 40명 얼굴 한눈에

```python
fig, axes = plt.subplots(4, 10, figsize=(15, 8))

# axes.flat: 4 × 10 칸을 한 줄로 늘어놓기
for person, ax in enumerate(axes.flat):
    # 그 사람의 사진 10장 중 첫 장
    first = faces.images[faces.target == person][0]
    ax.imshow(first, cmap="gray", vmin=0, vmax=1)
    ax.set_title(person, fontsize=10)
    ax.axis("off")

plt.show()
```

### KNN으로 229번 사진 맞히기

```python
X = faces.data
y = faces.target

new_index = 229
new_photo = X[new_index]

# 229번만 False → 399장
known = np.arange(400) != new_index
X_known, y_known = X[known], y[known]

K = 3
model = KNeighborsClassifier(n_neighbors=K)
model.fit(X_known, y_known)

# 사진 한 장도 [ ]로 감싸 "한 장짜리 묶음"으로
dist, idx = model.kneighbors([new_photo], n_neighbors=K)

print("가장 가까운 이웃:", y_known[idx[0]])
print("이웃과의 거리:", dist[0].round(2))
print("예측한 사람 :", model.predict([new_photo])[0])
print("실제 정답:", y[new_index])
```

### 거리를 직접 계산

```python
# 같은 칸끼리 차이를 제곱해 4096칸 더하고 제곱근
diff = X_known - new_photo
d = np.sqrt((diff ** 2).sum(axis=1))
print(np.sort(d)[:3].round(2))
```

### 이웃 사진 그리기

```python
for i in range(K):
    # idx는 X_known 안의 자리
    j = idx[0][i]

    plt.figure(figsize=(1, 1))
    plt.imshow(X_known[j].reshape(64, 64), cmap="gray")
    plt.title(f"Person {y_known[j]}")
    plt.axis("off")
    plt.show()
```

### KNN 채점

```python
X_train, X_test, y_train, y_test = train_test_split(
    faces.data,
    faces.target,
    test_size=0.25,
    random_state=42,
    stratify=faces.target
)

knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train, y_train)
print(f"예측 정확도: {knn.score(X_test, y_test):.2%}")
```

### 로지스틱 회귀

```python
df = pd.DataFrame({
    "study_hours":    [1, 2, 3, 4, 5, 2, 3, 5, 4, 1],
    "sleep_hours":    [6, 8, 5, 7, 8, 5, 8, 6, 6, 8],
    "practice_count": [10, 15, 20, 25, 30, 12, 28, 22, 32, 8],
    "score":          [47, 58, 60, 76, 87, 49, 73, 77, 79, 52],
})

X = df[["study_hours", "sleep_hours", "practice_count"]]
# 점수 대신 합격 여부(True, False)가 정답
y = df["score"] >= 60

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.3,
    random_state=42,
    stratify=y
)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

model = LogisticRegression(max_iter=1000)
model.fit(X_train_scaled, y_train)
print("클래스 :", model.classes_.tolist())
print("가중치 :", model.coef_[0].round(3))
```

### 새 학생 예측과 채점

```python
new = pd.DataFrame([[4, 7, 30]], columns=X.columns)
new_scaled = scaler.transform(new)

predict = model.predict(new_scaled)[0]
# [0, 1]: 첫 학생의 True(합격) 확률
pass_proba = model.predict_proba(new_scaled)[0, 1]
print("예측 결과:", "합격" if predict else "불합격")
print(f"합격 확률: {pass_proba:.2%}")

test_pred = model.predict(X_test_scaled)
print(f"정확도: {accuracy_score(y_test, test_pred):.4%}")
print(confusion_matrix(y_test, test_pred, labels=[False, True]))
```

### 혼동 행렬 히트맵

```python
cm = confusion_matrix(y_test, test_pred, labels=[False, True])

plt.figure(figsize=(4, 3))
sns.heatmap(
    cm,
    annot=True,
    cmap="Blues",
    xticklabels=["Fail", "Pass"],
    yticklabels=["Fail", "Pass"]
)
plt.xlabel("Predicted")
plt.ylabel("Actual")
plt.show()
```

## 4. 핵심 정리

| 개념 | 내 말로 |
| --- | --- |
| KNN | 새 데이터와 가장 가까운 K개를 찾아 그 정답의 다수결로 답한다. 학습은 사실상 외워 두기 |
| 이웃 수 K | 투표할 이웃 수. 작으면 한두 장에 휘둘리고, 크면 먼 사진까지 섞인다 |
| 유클리드 거리 | 같은 칸끼리 차이를 제곱해 다 더하고 제곱근. 사진 두 장이 얼마나 다른지 |
| `stratify` | 정답 종류의 비율을 학습용과 시험용에 고르게 나눈다 |
| 분류 | 답이 정해진 몇 가지 중 하나. 사람 번호, 합격 여부 |
| 로지스틱 회귀 | 가중합 z를 시그모이드로 0~1 확률로 바꿔 0.5 기준으로 고르는 분류 모델 |
| 시그모이드 | 1 ÷ (1 + e^(-z)). z = 0이면 0.5, z = 2이면 0.881 |
| `predict_proba` | `classes_` 순서대로 `[False 확률, True 확률]`. 두 칸 합은 1 |
| 혼동 행렬 | 행이 실제, 열이 예측. 대각선이 맞힌 수 |

- K에 따라 229번의 답이 바뀐다. 이웃 10장은 가까운 순서로 `17 22 22 22 17 17 3 17 11 8`. K가 3, 4, 5일 때만 22로 맞히고, K = 1은 맨 앞의 17번 한 장만 보고 틀린다. K = 10은 17번 4표, 22번 3표라 틀린다
- 동점이면 scikit-learn은 번호가 작은 사람을 고른다. K = 2, 6, 7이 17번과 22번 동점이라 17로 틀린다
- 같은 나눔에서 K만 바꾸면 정확도가 K = 1 95%, 3 92%, 5 87%, 7 79%, 10 74%. 사람마다 학습용 사진이 7~8장뿐이라 K가 크면 다른 사람이 많이 섞인다
- `stratify` 없이 나누면 시험용에 7장 들어간 사람도, 0장인 사람(16번)도 생기고 정확도는 83%
- 새 학생 (4, 7, 30)을 표준화하면 `[1.21 0.13 1.22]`. z = 0.873 × 1.21 - 0.37 × 0.13 + 0.877 × 1.22 + 0.447 ≈ 2.52, 시그모이드에 넣으면 0.926 = 92.58%
- 시험용 3명 100%는 운이 크다. 불합격인 5번 학생(49점)의 합격 확률이 0.459로 0.5를 아슬아슬하게 못 넘어서 맞힌 것

## 5. 헷갈린 점, 고친 점

- 수업 노트북은 `K = 10`인데 출력은 `[17 22 22]`, 예측 22였다. 이건 `K = 3`으로 실행한 결과가 남은 것이다. `K = 10`으로 다시 돌리면 `예측한 사람 : 17`로 틀린다
- `plt.axis="off"`는 축을 숨기지 않는다. `plt.axis`라는 이름에 글자를 넣어 함수를 덮어쓴다. 그 뒤 `plt.axis("off")`를 부르면 `TypeError: 'str' object is not callable`. 괄호로 부르고, 이미 실행했으면 커널을 Restart
- `idx`는 원래 400장의 번호가 아니라 `X_known`(399장) 안의 자리다. 229번 뒤의 사진은 하나씩 당겨져 있다. 그래서 정답은 `y[idx[0]]`이 아니라 `y_known[idx[0]]`
- 수업 주석의 "339장"은 399장의 오타다(`known.sum()`이 399)
- KNN에는 `faces.images`(400, 64, 64)가 아니라 한 줄로 편 `faces.data`(400, 4096)를 넣는다. 그릴 때만 `reshape(64, 64)`로 다시 접는다
- `predict_proba(new_scaled)[0, 1]`의 `[0, 1]`은 0번째 학생의 1번째 칸(True). `classes_`가 `[False, True]`라서 1번째가 합격
- `stratify=y`를 줘서 어제(24회차)와 같은 `random_state=42`인데도 시험용이 8, 1, 5번이 아니라 5, 7, 4번 학생이다

## 6. 실행 예시

```txt
X: (400, 4096)
y: (400,)
399
가장 가까운 이웃: [17 22 22]
이웃과의 거리: [6.07 6.26 6.44]
예측한 사람 : 22
실제 정답: 22
[6.07 6.26 6.44]
```

```txt
# K = 10으로 바꿔 실행
가장 가까운 이웃: [17 22 22 22 17 17  3 17 11  8]
이웃과의 거리: [6.07 6.26 6.44 6.63 6.68 6.93 7.02 7.07 7.23 7.27]
예측한 사람 : 17
실제 정답: 22
```

```txt
(300, 4096) (100, 4096)
예측 정확도: 87.00%
```

```txt
학생 10 명, X : 3개
   score  passed
0     47   False
1     58   False
2     60    True
3     76    True
4     87    True
학습 7명, 테스트 3명
첫 학습 학생의 공부시간: 4 시간
표준화 값: 1.21
클래스 : [False, True]
가중치 : [ 0.873 -0.37   0.877]
```

```txt
예측 결과: 합격
합격 확률: 92.58%
실제 정답:  [False, True, True]
예측 결과:  [False, True, True]
정확도: 100.0000%
[[1 0]
 [0 2]]
```


