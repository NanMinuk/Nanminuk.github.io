---
title: 한국어 감정 분석
tags: NLP
typora-root-url: ../
---

```python
from konlpy.tag import Twitter
from konlpy.tag import Okt
from konlpy.tag import Kkma
```


    ---------------------------------------------------------------------------

    ModuleNotFoundError                       Traceback (most recent call last)

    <ipython-input-1-5da8afa8a1af> in <module>
    ----> 1 from konlpy.tag import Twitter
          2 from konlpy.tag import Okt
          3 from konlpy.tag import Kkma
    

    ModuleNotFoundError: No module named 'konlpy'



```python
import pandas as pd
import warnings
warnings.filterwarnings('ignore')

# 컬럼 분리 문자 \t 
train_df = pd.read_csv('ratings_train.txt', sep='\t') # 한글 encoding시 encoding='cp949' 적용.
train_df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>document</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9976970</td>
      <td>아 더빙.. 진짜 짜증나네요 목소리</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3819312</td>
      <td>흠...포스터보고 초딩영화줄....오버연기조차 가볍지 않구나</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10265843</td>
      <td>너무재밓었다그래서보는것을추천한다</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9045019</td>
      <td>교도소 이야기구먼 ..솔직히 재미는 없다..평점 조정</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6483659</td>
      <td>사이몬페그의 익살스런 연기가 돋보였던 영화!스파이더맨에서 늙어보이기만 했던 커스틴 ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5403919</td>
      <td>막 걸음마 뗀 3세부터 초등학교 1학년생인 8살용영화.ㅋㅋㅋ...별반개도 아까움.</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7797314</td>
      <td>원작의 긴장감을 제대로 살려내지못했다.</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>9443947</td>
      <td>별 반개도 아깝다 욕나온다 이응경 길용우 연기생활이몇년인지..정말 발로해도 그것보단...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>7156791</td>
      <td>액션이 없는데도 재미 있는 몇안되는 영화</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>5912145</td>
      <td>왜케 평점이 낮은건데? 꽤 볼만한데.. 헐리우드식 화려함에만 너무 길들여져 있나?</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
train_df['label'].value_counts( )
```




    0    75173
    1    74827
    Name: label, dtype: int64




```python
train_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 150000 entries, 0 to 149999
    Data columns (total 3 columns):
     #   Column    Non-Null Count   Dtype 
    ---  ------    --------------   ----- 
     0   id        150000 non-null  int64 
     1   document  149995 non-null  object
     2   label     150000 non-null  int64 
    dtypes: int64(2), object(1)
    memory usage: 3.4+ MB
    


```python
import re

train_df = train_df.fillna(' ')
# 정규 표현식을 이용하여 숫자를 공백으로 변경(정규 표현식으로 \d 는 숫자를 의미함.) 
train_df['document'] = train_df['document'].apply( lambda x : re.sub(r"\d+", " ", x) )

# 테스트 데이터 셋을 로딩하고 동일하게 Null 및 숫자를 공백으로 변환. 
test_df = pd.read_csv('ratings_test.txt', sep='\t') # 한글 encoding시 encoding='cp949' 적용.
test_df = test_df.fillna(' ')
test_df['document'] = test_df['document'].apply( lambda x : re.sub(r"\d+", " ", x) )

# id 컬럼 삭제 수행. 
train_df.drop('id', axis=1, inplace=True) 
test_df.drop('id', axis=1, inplace=True)

```


```python
from konlpy.tag import Okt

okt = Okt()
def tw_tokenizer(text):
    # 입력 인자로 들어온 text 를 형태소 단어로 토큰화 하여 list 객체 반환
    tokens_ko = okt.morphs(text)
    return tokens_ko

#tw_tokenizer('아버지가방에 들어가신다')
okt.morphs('아버지가방에 들어가신다')
```




    ['아버지', '가방', '에', '들어가신다']




```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import GridSearchCV

# Okt 객체의 morphs( ) 객체를 이용한 tokenizer를 사용. ngram_range는 (1,2) 
tfidf_vect = TfidfVectorizer(tokenizer=tw_tokenizer, ngram_range=(1,2), min_df=3, max_df=0.9)
tfidf_vect.fit(train_df['document'])
tfidf_matrix_train = tfidf_vect.transform(train_df['document'])
```


```python
print(tfidf_matrix_train.shape)
```

    (150000, 129276)
    


```python
# Logistic Regression 을 이용하여 감성 분석 Classification 수행. 
lg_clf = LogisticRegression(random_state=0, solver='liblinear')

# Parameter C 최적화를 위해 GridSearchCV 를 이용. 
params = { 'C': [1 ,3.5, 4.5, 5.5, 10 ] }
grid_cv = GridSearchCV(lg_clf , param_grid=params , cv=3 ,scoring='accuracy', verbose=1 )
grid_cv.fit(tfidf_matrix_train , train_df['label'] )
print(grid_cv.best_params_ , round(grid_cv.best_score_,4))

```

    Fitting 3 folds for each of 5 candidates, totalling 15 fits
    {'C': 3.5} 0.8593
    


```python
test_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>document</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>굳 ㅋ</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>GDNTOPCLASSINTHECLUB</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>뭐야 이 평점들은.... 나쁘진 않지만  점 짜리는 더더욱 아니잖아</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>지루하지는 않은데 완전 막장임... 돈주고 보기에는....</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>D만 아니었어도 별 다섯 개 줬을텐데.. 왜  D로 나와서 제 심기를 불편하게 하죠??</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
from sklearn.metrics import accuracy_score

# 학습 데이터를 적용한 TfidfVectorizer를 이용하여 테스트 데이터를 TF-IDF 값으로 Feature 변환함. 
tfidf_matrix_test = tfidf_vect.transform(test_df['document'])

# classifier 는 GridSearchCV에서 최적 파라미터로 학습된 classifier를 그대로 이용
best_estimator = grid_cv.best_estimator_
preds = best_estimator.predict(tfidf_matrix_test)

print('Logistic Regression 정확도: ',accuracy_score(test_df['label'],preds))
```

    Logistic Regression 정확도:  0.86172
    


```python

```
