## 1. 오늘의 흐름

한 달 지출 12건을 리스트 → NumPy 배열 → pandas DataFrame 순서로 다뤘다. 반복문으로 하나씩 하던 계산·필터·집계를 한 줄로 끝낸다.

1. NumPy 배열로 계산, 통계, 조건 필터
2. pandas DataFrame 만들기, 열 선택, 조건 필터, 정렬
3. 새 열 추가, 값 수정(`loc`), `groupby`·`value_counts` 집계

## 2. 구조

```txt
list       [3200, 1500, ...]                     for문으로 하나씩
   ↓ np.array(...)
ndarray    arr * 2, arr.sum(), arr[arr >= 10000]  배열 전체에 한 번에
   ↓ pd.DataFrame({...})
DataFrame  열 이름이 붙은 표                     열 하나 = Series
```

| 열 | 담은 것 | dtype |
| --- | --- | --- |
| `day` | 날짜(일) | `int64` |
| `shop` | 가게 | `str` |
| `category` | 분류 (식비·교통·문화·쇼핑) | `str` |
| `amount` | 금액 | `int64` |

DataFrame은 엑셀 표나 DB 테이블과 같은 모양이고, 열 하나를 꺼내면 Series다.

### 문법 사용법

```python
import numpy as np
import pandas as pd

# NumPy: 배열 전체에 한 번에
arr = np.array(리스트)
arr * 2, arr.sum(), arr.mean(), arr.min(), arr.max()
arr.shape, arr.dtype
arr[arr >= 10000]                 # 조건이 True인 원소만

# DataFrame 만들기: {열 이름: 리스트} (리스트 길이가 같아야 한다)
df = pd.DataFrame({"열1": [...], "열2": [...]})
df.head(5), df.info()

# 열 고르기
df["열"]                          # Series
df[["열1", "열2"]]                # DataFrame (대괄호 두 겹)

# 행 고르기: 조건마다 괄호, 그리고 &, 또는 |
df[df["열"] == "값"]
df[(조건1) & (조건2)]

# 정렬
df.sort_values("열", ascending=False).head(3)

# 새 열 / 값 하나 수정
df["새열"] = df["열"] // 100
df.loc[행라벨, "열"] = 새값

# 집계
df.groupby("묶을열")["계산할열"].sum().sort_values(ascending=False)
df["열"].value_counts()
```

## 3. 코드

`pandas.ipynb`의 셀을 순서대로 옮긴 것이다.

```python
# [셀 1] 리스트
amounts = [3200, 1500, 15000, 6500, 32000, 4800,
           10900, 1500, 2700, 3000, 18500, 7000]

print(amounts)
print("합계", sum(amounts))

# [셀 2] NumPy 배열
import numpy as np

arr = np.array(amounts)
print(arr * 2)

print("합계:",arr.sum())
print("평균:",round(arr.mean()))
print("최소:",arr.min())
print("최대:",arr.max())

print("모양: ", arr.shape, " 데이터 타입: ", arr.dtype)

# [셀 3] 불리언 필터
print(arr >= 10000)
arr[arr >= 10000]

# [셀 4] 반복문으로 식비 합계
amounts = [3200, 1500, 15000, 6500, 32000, 4800,
           10900, 1500, 2700, 3000, 18500, 7000]

shops = ["편의점", "버스", "영화관", "분식집", "온라인쇼핑", "카페",
         "음원앱", "버스", "편의점", "웹툰", "온라인쇼핑", "분식집"]

categories = ["식비", "교통", "문화", "식비", "쇼핑", "식비",
              "문화", "교통", "식비", "문화", "쇼핑", "식비"]

print(shops)

food = 0
for c, a in zip(categories, amounts):
    if c == '식비':
        food += a
        
print(food)

# [셀 5] DataFrame 만들기
import pandas as pd

arr_amounts = pd.DataFrame(amounts)
arr_shops = pd.DataFrame(shops)
arr_categories = pd.DataFrame(categories)

spend = pd.DataFrame({
    "day": [2,3,4,5,7,9,12,14,16,19,20,25],
    "shop": shops,
    "category": categories,
    "amount": amounts
})

spend.head(5)
spend.info()

# [셀 6] 열 하나(Series)와 통계
print(spend['amount'])

print('합계:', spend['amount'].sum())
print('평균:', round(spend['amount'].mean(),2))

# [셀 7] 여러 열
spend[ ['shop', 'amount'] ]

# [셀 8] 조건식 결과
spend["category"] == "문화"

# [셀 9] 조건 필터
# spend[조건] 조건이 True 인 행만 필터링
spend[spend["category"] == "문화"]

# [셀 10]
# 조건식
# spend[조건식]
spend['amount'] >= 10000
spend[spend['amount'] >= 10000]

# [셀 11] 조건 두 개
# 식비 5000원 이상 사용한 행
spend[(spend["category"] == '식비') & (spend["amount"] >= 5000)]

# [셀 12] 정렬
top3 = spend.sort_values('amount', ascending=False)
top3

# [셀 13] 새 열
# spend['새컴럼명'] = 계산식
spend['point'] = spend['amount'] // 100 # 100원 마다 1 포인트 
# del spend['point']
spend.head(5)
print(spend['point'].sum())

spend

# [셀 14] 값 수정
spend.loc[2, 'amount'] = 14000
spend['point'] = spend['amount'] // 100
spend.head(3)

# [셀 15~16] 그룹별 합계
spend.groupby('category')['amount'].sum()
spend.groupby('category')['amount'].sum().sort_values(ascending=False)

# [셀 17] 값별 개수
spend['category'].value_counts()

spend['category']
```

## 4. 핵심 정리

**Q. NumPy 배열은 리스트와 뭐가 다르지?**

- 연산이 모든 원소에 한 번에 적용된다. `arr * 2`는 값마다 2를 곱한다. 리스트에 `* 2`를 하면 리스트가 두 번 이어 붙는다.
- `sum()`, `mean()`, `min()`, `max()`로 통계를 바로 구한다. `shape`는 모양 `(12,)`, `dtype`은 원소 타입 `int64`다.
- `arr >= 10000`은 True/False 배열을 만들고, `arr[그 배열]`은 True인 원소만 남긴다(불리언 인덱싱).

**Q. DataFrame은 어떻게 만들고 살펴보지?**

- `{열 이름: 리스트}` 딕셔너리로 만든다. 각 리스트가 열 하나가 된다.
- `head(n)`은 앞 n행, `info()`는 행 수·열별 빈 값 개수·dtype을 보여준다. 문자열 열은 `str`로 나왔다.
- `spend['amount']`는 Series, `spend[['shop', 'amount']]`는 DataFrame이다. 열 여러 개는 리스트로 넘기니 대괄호가 두 겹이다.

**Q. 조건으로 행을 고르려면?**

- `spend[조건]`은 조건이 True인 행만 남긴다. SQL의 `WHERE`와 같다.
- 조건 여러 개는 `&`(그리고), `|`(또는)로 잇고 조건마다 괄호로 감싼다.
- 직접 해보니 `and`를 쓰면 `ValueError`, 괄호를 빼면 `TypeError`가 났다. `&`가 `==`, `>=`보다 먼저 계산되기 때문이다.

**Q. 정렬, 새 열, 값 수정은?**

- `sort_values('amount', ascending=False)`는 큰 순서로 정렬한다. 상위 3개만 보려면 `.head(3)`을 붙인다.
- `spend['point'] = spend['amount'] // 100`처럼 없는 열 이름에 대입하면 새 열이 생긴다. `//`는 몫이라 3200원은 32점이다.
- `spend.loc[2, 'amount'] = 14000`은 행 라벨 2, 열 `amount` 한 칸을 바꾼다.
- `point`는 계산해 둔 값이라 `amount`를 바꿔도 저절로 바뀌지 않는다. 그래서 다시 계산했다.

**Q. 그룹별로 묶어 세려면?**

- `groupby('category')['amount'].sum()`은 분류별 금액 합계다. SQL의 `GROUP BY`와 같다.
- `value_counts()`는 값마다 몇 번 나왔는지 많은 순으로 센다.
- 노트북은 셀의 마지막 줄 결과만 화면에 보여준다. 그래서 셀 17의 `value_counts()`와 셀 5의 `head(5)`는 출력이 없다. 중간 결과도 보려면 `print()`로 감싼다.

| 하고 싶은 것 | SQL | pandas |
| --- | --- | --- |
| 열 고르기 | `SELECT shop, amount` | `spend[['shop', 'amount']]` |
| 행 거르기 | `WHERE amount >= 10000` | `spend[spend['amount'] >= 10000]` |
| 정렬 | `ORDER BY amount DESC` | `sort_values('amount', ascending=False)` |
| 그룹별 합계 | `GROUP BY category`와 `SUM(amount)` | `groupby('category')['amount'].sum()` |
| 개수 제한 | `LIMIT 3` | `.head(3)` |

## 5. 오늘 만난 오류

| 메시지 | 원인 |
| --- | --- |
| `ValueError: The truth value of a Series is ambiguous` | 조건 사이에 `and`를 씀 → `&`로 |
| `TypeError: Cannot perform 'rand_' with a dtyped [int64] array and scalar of type [bool]` | `&` 양쪽 조건에 괄호가 없음 |

## 6. 실행 예시

```python
arr = np.array(amounts)
print(arr * 2)
print("합계:",arr.sum())
print("평균:",round(arr.mean()))
print("최소:",arr.min())
print("최대:",arr.max())
print("모양: ", arr.shape, " 데이터 타입: ", arr.dtype)
```

```txt
[ 6400  3000 30000 13000 64000  9600 21800  3000  5400  6000 37000 14000]
합계: 106600
평균: 8883
최소: 1500
최대: 32000
모양:  (12,)  데이터 타입:  int64
```

```python
print(arr >= 10000)
arr[arr >= 10000]
```

```txt
[False False  True False  True False  True False False False  True False]
array([15000, 32000, 10900, 18500])
```

```python
food = 0
for c, a in zip(categories, amounts):
    if c == '식비':
        food += a
print(food)
```

```txt
24200
```

```python
spend.info()
```

```txt
<class 'pandas.DataFrame'>
RangeIndex: 12 entries, 0 to 11
Data columns (total 4 columns):
 #   Column    Non-Null Count  Dtype
---  ------    --------------  -----
 0   day       12 non-null     int64
 1   shop      12 non-null     str  
 2   category  12 non-null     str  
 3   amount    12 non-null     int64
dtypes: int64(2), str(2)
memory usage: 516.0 bytes
```

```python
print('합계:', spend['amount'].sum())
print('평균:', round(spend['amount'].mean(),2))
```

```txt
합계: 106600
평균: 8883.33
```

```python
spend[spend["category"] == "문화"]
```

```txt
   day shop category  amount
2    4  영화관       문화   15000
6   12  음원앱       문화   10900
9   19   웹툰       문화    3000
```

```python
spend[spend['amount'] >= 10000]
```

```txt
    day   shop category  amount
2     4    영화관       문화   15000
4     7  온라인쇼핑       쇼핑   32000
6    12    음원앱       문화   10900
10   20  온라인쇼핑       쇼핑   18500
```

```python
spend[(spend["category"] == '식비') & (spend["amount"] >= 5000)]
```

```txt
    day shop category  amount
3     5  분식집       식비    6500
11   25  분식집       식비    7000
```

```python
spend.sort_values('amount', ascending=False).head(3)
```

```txt
    day   shop category  amount
4     7  온라인쇼핑       쇼핑   32000
10   20  온라인쇼핑       쇼핑   18500
2     4    영화관       문화   15000
```

```python
spend['point'] = spend['amount'] // 100
print(spend['point'].sum())
spend.head(3)
```

```txt
1066
   day shop category  amount  point
0    2  편의점       식비    3200     32
1    3   버스       교통    1500     15
2    4  영화관       문화   15000    150
```

```python
spend.loc[2, 'amount'] = 14000
spend['point'] = spend['amount'] // 100
spend.head(3)
```

```txt
   day shop category  amount  point
0    2  편의점       식비    3200     32
1    3   버스       교통    1500     15
2    4  영화관       문화   14000    140
```

```python
spend.groupby('category')['amount'].sum().sort_values(ascending=False)
```

```txt
category
쇼핑    50500
문화    27900
식비    24200
교통     3000
Name: amount, dtype: int64
```

```python
spend['category'].value_counts()
```

```txt
category
식비    5
문화    3
교통    2
쇼핑    2
Name: count, dtype: int64
```
