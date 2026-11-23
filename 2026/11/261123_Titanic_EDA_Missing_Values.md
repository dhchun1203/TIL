## 1. 오늘의 흐름

머신러닝에 넣기 전에 데이터를 먼저 살피고 빈칸을 채웠다. 모델은 빈칸(NaN)을 그대로 받지 못하니, 열의 성격에 맞는 값으로 채우는 게 첫 단계다.

1. seaborn의 타이타닉 데이터(891행 × 15열)를 불러와 `shape`, `info`, `describe`로 살펴보기
2. `isnull().sum()`으로 열마다 빈칸 세기
3. 숫자는 평균, 글자는 최빈값, category는 `'Unknown'`으로 채우기
4. IQR로 튀는 요금 찾기, 중복 행 확인, 나이 분포 히스토그램 그리기

## 2. 구조

```txt
불러오기          sns.load_dataset('titanic')
   ↓
살펴보기          shape → info → describe
   ↓
빈칸 찾기         df.isnull().sum()
   ↓
채우기            숫자 → 평균 / 글자 → 최빈값 / category → 'Unknown'
   ↓
다시 확인         df.isnull().sum() 이 0인지
   ↓
튀는 값          IQR로 상한선 계산 → 극단값(fare 512) 제거
   ↓
중복 행          df.duplicated().sum()
   ↓
그림으로 보기    ax.hist(df['age'])
```

| 빈칸 있는 열 | 빈칸 | dtype | 채운 값 |
| --- | --- | --- | --- |
| `age` | 177 | `float64` | 평균 29.7 |
| `embark_town` | 2 | `str` | 최빈값 `Southampton` |
| `embarked` | 2 | `str` | `embark_town`의 첫 글자 |
| `deck` | 688 | `category` | `'Unknown'` |

`deck`은 891행 중 688행(77%)이 비어 있어서, 평균이나 최빈값으로 채우면 없는 정보를 지어내게 된다. 그래서 "모름"을 뜻하는 `'Unknown'`으로 채운다.

### 문법 사용법

```python
import seaborn as sns
df = sns.load_dataset('titanic')

# 살펴보기
df.shape                          # (행, 열)
df.info()                         # 열마다 빈칸 아닌 개수, dtype
df.describe()                     # 숫자 열의 개수·평균·표준편차·사분위수

# 빈칸 세기
df.isnull().sum()                 # 열마다 빈칸 개수
df['열'].value_counts(dropna=False)   # 값별 개수 (NaN까지)

# 빈칸 채우기: 결과를 다시 대입한다
df.loc[:, '열'] = df['열'].fillna(df['열'].mean())     # 숫자 → 평균
df.loc[:, '열'] = df['열'].fillna(df['열'].mode()[0])  # 글자 → 최빈값

# category 열: 목록에 먼저 추가한 뒤 채운다
df['열'] = df['열'].cat.add_categories('Unknown').fillna('Unknown')

# 문자열 첫 글자
df['열'].str[0]

# 튀는 값: IQR = Q3 - Q1, 상한선 = Q3 + 1.5 × IQR
q1 = df['열'].quantile(0.25)
q3 = df['열'].quantile(0.75)
upper = q3 + 1.5 * (q3 - q1)
(df['열'] > upper).sum()        # 상한선을 넘는 행 수
df = df[df['열'] < 기준].copy()  # 기준 미만만 남기기

# 중복 행
df.duplicated().sum()                    # 앞에 똑같은 행이 있는 행 수
df[df.duplicated(keep=False)]            # 중복된 행 전부

# 히스토그램
import matplotlib.pyplot as plt
plt.rcParams['font.family'] = 'AppleGothic'   # 맥 한글 글꼴 (윈도우는 'Malgun Gothic')
fig, ax = plt.subplots(figsize=(6, 4))
ax.hist(df['열'], bins=20)
ax.set_title('제목')
```

## 3. 코드

`01_ml.ipynb`의 셀을 순서대로 옮긴 것이다. 셀 12의 `add_categories` 줄은 지금 주석 처리돼 있지만, 출력은 그 줄을 실행했을 때의 결과다.

```python
# [셀 1]
import pandas as pd
import numpy as np
import seaborn as sns
# seaborn: 파이썬 시각화 라이브러리

print("pandas:", pd.__version__, "numpy:", np.__version__, "seaborn:", sns.__version__)

# [셀 2]
df = sns.load_dataset('titanic')
df.head(5)

# [셀 3~4]
df.shape
df.info()

# [셀 5]
df.describe()

# [셀 6]
# 나이 분포
df.isnull().sum()

# [셀 7]
# 나이가 비어 있는 승객 수, 성별, 클래스
empty_age = df['age'].isnull()
print('나이 빈칸: ', empty_age.sum(), '명')
df.loc[empty_age, ['sex', 'age', 'class']]

# [셀 8]
# 평균값, 중앙값 산출
print("평균:", round(df['age'].mean()))
print("중앙값:", round(df['age'].median()))

mean_age = df['age'].mean()
# fillna : 빈칸(NaN)만 주어진 값을 새로운 값으로 채우는 메소드 df['열'] = df['열'].fillna(새로운값)
df.loc[:,'age'] = df['age'].fillna(mean_age)
# df['age'].isnull().sum()
df.loc[empty_age, ['sex', 'age', 'class']].head()

# [셀 9]
# 탑승 항구 : 최빈값
df['embark_town'].value_counts(dropna=False)

# [셀 10]
# 최빈값 추출후 빈칸에 채우는 로직
mode_town = df["embark_town"].mode()
print('최빈값: ', mode_town)

df.loc[:, "embark_town"] = df['embark_town'].fillna(mode_town)
df['embark_town'].value_counts(dropna=False)

# [셀 11]
# embarked town
df.loc[:, 'embarked'] = df['embark_town'].str[0]
df[['embarked', 'embark_town']].isnull().sum()

# [셀 12]
print(df['deck'].dtype)
df['deck'].cat.categories

# df['deck'] = df['deck'].cat.add_categories('Unknown').fillna('Unknown')
print(df['deck'].cat.categories.to_list())
print(df['deck'].value_counts())

# [셀 13]
df.isnull().sum()

# [셀 14]
# 튀는 값 확인

print(df['fare'].quantile([0.25,0.5,0.75]))
df['fare'].describe()[["25%", "50%", "75%"]]

# IQR 계산 (Q3 - Q1)
# 이상수치 Q3 + 1.5 * IQR

q1 = df['fare'].quantile(0.25)
q3 = df['fare'].quantile(0.75)
iqr = q3 - q1
upper = q3 + 1.5 * iqr

print(f"Q1 = {q1:.2f}, Q3 = {q3:.2f} IQR = {iqr:.2f} UPPER = {upper:.2f}")
print("이상 승선요금: ",(df["fare"] > upper).sum(), "명")

# [셀 15]
df[df['fare'] > 512]

# [셀 16]
before = len(df)
df = df[df['fare'] < 512].copy()
print(before, "=>", len(df))

# [셀 17]
# 중복되는 행 확인

print("중복 행:", df.duplicated().sum(), '개')
df[df.duplicated(keep=False)].sort_values(['fare', 'age']).head(5)

# [셀 18]
import platform
import matplotlib.pyplot as plt

os_name = platform.system()
if os_name == "Windows":
    plt.rcParams['font.family'] = "Malgun Gothic"
elif os_name == "Darwin":
    plt.rcParams["font.family"] = "AppleGothic"

# 나이 분포 차트
fig, ax = plt.subplots(figsize=(6,4))
ax.hist(df["age"], bins = 20, color="blue", edgecolor="black")
ax.set_title("나이분포")
ax.set_xlabel("나이(세)")
```

## 4. 핵심 정리

**Q. 데이터를 처음 받으면 뭘 보지?**

- `shape`로 크기를 본다. 타이타닉은 891행 × 15열이다.
- `info()`의 Non-Null Count가 891보다 작으면 빈칸이 있다는 뜻이다. dtype으로 숫자·글자·category를 구분한다.
- `describe()`는 숫자 열만 요약한다. `age`의 count가 714라서 여기서도 177개가 빈 것을 알 수 있다.
- `isnull().sum()`으로 열마다 빈칸 수를 정확히 센다.

**Q. 숫자 빈칸은 뭘로 채우지?**

- 평균(`mean()`)이나 중앙값(`median()`)으로 채운다. `age`는 평균 29.7, 중앙값 28이다.
- `fillna`는 채운 결과를 돌려줄 뿐 원본을 바꾸지 않으니 `df.loc[:, 'age'] = ...`로 다시 넣는다.
- 채운 뒤에 평균·중앙값을 다시 구하면 채운 값이 섞인다. 직접 해보니 중앙값이 28에서 29.7로 바뀌었다. 노트북의 "중앙값: 30"은 셀을 다시 실행해서 나온 값이다.

**Q. 글자 빈칸은?**

- 가장 많이 나온 값(최빈값)으로 채운다. `embark_town`은 Southampton이 644명으로 가장 많다.
- `value_counts(dropna=False)`로 세면 NaN도 따로 세어서 채우기 전후를 비교할 수 있다.
- `mode()`는 값이 아니라 Series를 돌려준다. 값 하나를 꺼내려면 `mode()[0]`이라고 쓴다.

**Q. category 열은 왜 따로 다루지?**

- `category`는 들어갈 수 있는 값 목록(`categories`)이 정해진 타입이다. `deck`은 A~G만 들어갈 수 있다.
- 목록에 없는 `'Unknown'`은 바로 못 넣는다. `cat.add_categories('Unknown')`로 목록에 먼저 추가한 뒤 `fillna`한다.

**Q. 다른 열에서 값을 만들려면?**

- `df['embark_town'].str[0]`은 문자열마다 첫 글자를 꺼낸다. Southampton → S로 `embarked`를 다시 만들었다.
- 원본이 NaN이면 결과도 NaN이다. 그래서 `embark_town`을 먼저 채워야 `embarked`도 채워진다.

**Q. 튀는 값(이상치)은 어떻게 찾지?**

- 데이터를 크기 순으로 줄 세웠을 때 25% 지점이 Q1, 75% 지점이 Q3이고, 그 사이 폭이 IQR(Q3 - Q1)이다.
- Q3 + 1.5 × IQR을 넘으면 튀는 값으로 본다. `fare`는 상한선 65.63, 넘는 승객이 116명이었다.
- 116명을 다 지우지는 않았다. 최댓값 512.33만 따로 확인해 3행을 지웠고, 891행이 888행이 됐다.
- `df = df[조건].copy()`: 조건에 맞는 행만 남긴 표를 `df`에 다시 넣는다.

**Q. 중복 행과 그래프는?**

- `duplicated()`는 앞에 모든 열이 똑같은 행이 있으면 True다. `keep=False`를 주면 첫 행까지 포함해 중복 묶음 전부를 보여준다.
- 중복은 107행이었다. 이 데이터에는 이름이나 승객 번호 열이 없어서, 조건이 같은 다른 승객도 중복으로 잡힌다. 그래서 확인만 하고 지우지는 않았다.
- 히스토그램은 값을 구간(`bins`)으로 나눠 구간마다 몇 명인지 막대로 그린다. 한글 제목이 깨지지 않도록 운영체제에 맞는 한글 글꼴을 먼저 지정한다.
- 직접 세 보니 28.3~32.3세 구간이 261명으로 다른 구간(최대 98명)보다 훨씬 높다. 빈 나이 177개를 모두 평균 29.7로 채워서 그 막대에 몰린 것이다. 평균으로 채우면 분포가 한가운데로 쏠린다.

## 5. 오늘 만난 오류

**1) `fillna(mode_town)`을 했는데 빈칸이 그대로 2개**

```python
mode_town = df["embark_town"].mode()
df.loc[:, "embark_town"] = df['embark_town'].fillna(mode_town)
```

```txt
embark_town
Southampton    644
Cherbourg      168
Queenstown      77
NaN              2
```

오류 메시지 없이 조용히 안 채워진다. `mode()`는 인덱스가 0 하나뿐인 Series를 돌려준다. `fillna`에 Series를 넘기면 같은 인덱스끼리 짝지어 채우는데, 빈칸이 있는 행은 0번이 아니라서 짝이 없다. `embarked`의 빈칸 2개도 이것 때문에 남았다.

```python
# 고친 코드: [0]으로 값 하나를 꺼낸다
mode_town = df["embark_town"].mode()[0]          # 'Southampton'
df.loc[:, "embark_town"] = df['embark_town'].fillna(mode_town)
```

직접 해보니 고친 코드는 빈칸이 0개가 됐다.

**2) category 열에 바로 `'Unknown'`을 채운 경우**

```python
df['deck'].fillna('Unknown')
```

```txt
TypeError: Cannot setitem on a Categorical with a new category (Unknown), set the categories first
```

`'Unknown'`이 `deck`의 값 목록(A~G)에 없어서 거절된다. 메시지대로 목록부터 늘린다.

```python
# 고친 코드
df['deck'] = df['deck'].cat.add_categories('Unknown').fillna('Unknown')
```

## 6. 실행 예시

```python
df.info()
```

```txt
<class 'pandas.DataFrame'>
RangeIndex: 891 entries, 0 to 890
Data columns (total 15 columns):
 #   Column       Non-Null Count  Dtype   
---  ------       --------------  -----   
 0   survived     891 non-null    int64   
 1   pclass       891 non-null    int64   
 2   sex          891 non-null    str     
 3   age          714 non-null    float64 
 4   sibsp        891 non-null    int64   
 5   parch        891 non-null    int64   
 6   fare         891 non-null    float64 
 7   embarked     889 non-null    str     
 8   class        891 non-null    category
 9   who          891 non-null    str     
 10  adult_male   891 non-null    bool    
 11  deck         203 non-null    category
 12  embark_town  889 non-null    str     
 13  alive        891 non-null    str     
 14  alone        891 non-null    bool    
dtypes: bool(2), category(2), float64(2), int64(4), str(5)
```

```python
df.describe()
```

```txt
         survived      pclass         age       sibsp       parch        fare
count  891.000000  891.000000  714.000000  891.000000  891.000000  891.000000
mean     0.383838    2.308642   29.699118    0.523008    0.381594   32.204208
std      0.486592    0.836071   14.526497    1.102743    0.806057   49.693429
min      0.000000    1.000000    0.420000    0.000000    0.000000    0.000000
25%      0.000000    2.000000   20.125000    0.000000    0.000000    7.910400
50%      0.000000    3.000000   28.000000    0.000000    0.000000   14.454200
75%      1.000000    3.000000   38.000000    1.000000    0.000000   31.000000
max      1.000000    3.000000   80.000000    8.000000    6.000000  512.329200
```

```python
df.isnull().sum()
```

```txt
survived         0
pclass           0
sex              0
age            177
sibsp            0
parch            0
fare             0
embarked         2
class            0
who              0
adult_male       0
deck           688
embark_town      2
alive            0
alone            0
dtype: int64
```

```python
empty_age = df['age'].isnull()
print('나이 빈칸: ', empty_age.sum(), '명')
df.loc[empty_age, ['sex', 'age', 'class']]
```

```txt
나이 빈칸:  177 명
        sex  age   class
5      male  NaN   Third
17     male  NaN  Second
19   female  NaN   Third
...
888  female  NaN   Third

[177 rows x 3 columns]
```

```python
mean_age = df['age'].mean()
df.loc[:,'age'] = df['age'].fillna(mean_age)
df.loc[empty_age, ['sex', 'age', 'class']].head()
```

```txt
       sex        age   class
5     male  29.699118   Third
17    male  29.699118  Second
19  female  29.699118   Third
26    male  29.699118   Third
28  female  29.699118   Third
```

```python
df['embark_town'].value_counts(dropna=False)
```

```txt
embark_town
Southampton    644
Cherbourg      168
Queenstown      77
NaN              2
Name: count, dtype: int64
```

```python
print(df['deck'].cat.categories.to_list())
print(df['deck'].value_counts())
```

```txt
['A', 'B', 'C', 'D', 'E', 'F', 'G', 'Unknown']
deck
Unknown    688
C           59
B           47
D           33
E           32
A           15
F           13
G            4
Name: count, dtype: int64
```

```python
df.isnull().sum()
```

```txt
survived       0
pclass         0
sex            0
age            0
sibsp          0
parch          0
fare           0
embarked       2
class          0
who            0
adult_male     0
deck           0
embark_town    2
alive          0
alone          0
dtype: int64
```

`embarked`, `embark_town`의 2개는 위 오류 1)을 고치면 0이 된다.

```python
q1 = df['fare'].quantile(0.25)
q3 = df['fare'].quantile(0.75)
iqr = q3 - q1
upper = q3 + 1.5 * iqr
print(f"Q1 = {q1:.2f}, Q3 = {q3:.2f} IQR = {iqr:.2f} UPPER = {upper:.2f}")
print("이상 승선요금: ",(df["fare"] > upper).sum(), "명")
```

```txt
Q1 = 7.91, Q3 = 31.00 IQR = 23.09 UPPER = 65.63
이상 승선요금:  116 명
```

```python
df[df['fare'] > 512]
```

```txt
     survived  pclass     sex   age  sibsp  parch      fare embarked  class    who  ...     deck embark_town alive  alone
258         1       1  female  35.0      0      0  512.3292        C  First  woman  ...  Unknown   Cherbourg   yes   True
679         1       1    male  36.0      0      1  512.3292        C  First    man  ...        B   Cherbourg   yes  False
737         1       1    male  35.0      0      0  512.3292        C  First    man  ...        B   Cherbourg   yes   True
```

```python
before = len(df)
df = df[df['fare'] < 512].copy()
print(before, "=>", len(df))
```

```txt
891 => 888
```

```python
print("중복 행:", df.duplicated().sum(), '개')
df[df.duplicated(keep=False)].sort_values(['fare', 'age']).head(5)
```

```txt
중복 행: 107 개
     survived  pclass   sex        age  sibsp  parch  fare embarked   class  ...     deck  embark_town alive  alone
277         0       2  male  29.699118      0      0   0.0        S  Second  ...  Unknown  Southampton    no   True
413         0       2  male  29.699118      0      0   0.0        S  Second  ...  Unknown  Southampton    no   True
466         0       2  male  29.699118      0      0   0.0        S  Second  ...  Unknown  Southampton    no   True
481         0       2  male  29.699118      0      0   0.0        S  Second  ...  Unknown  Southampton    no   True
674         0       2  male  29.699118      0      0   0.0        S  Second  ...  Unknown  Southampton    no   True
```

```python
fig, ax = plt.subplots(figsize=(6,4))
ax.hist(df["age"], bins = 20, color="blue", edgecolor="black")
ax.set_title("나이분포")
ax.set_xlabel("나이(세)")
```

결과는 파란 막대 20개의 나이 분포 그래프다. 28.3~32.3세 막대(261명)가 혼자 우뚝 솟아 있다.
