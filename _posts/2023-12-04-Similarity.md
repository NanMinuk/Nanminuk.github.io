---
title: 문서 유사도
tags: NLP
typora-root-url: ../
---

### 문서 유사도 측정 방법 – 코사인 유사도


```python
import numpy as np

def cos_similarity(v1, v2):
    dot_product = np.dot(v1, v2)
    l2_norm = (np.sqrt(sum(np.square(v1))) * np.sqrt(sum(np.square(v2))))
    similarity = dot_product / l2_norm     
    
    return similarity

```


```python
from sklearn.feature_extraction.text import TfidfVectorizer

doc_list = ['if you take the blue pill, the story ends' ,
            'if you take the red pill, you stay in Wonderland',
            'if you take the red pill, I show you how deep the rabbit hole goes']

tfidf_vect_simple = TfidfVectorizer()
feature_vect_simple = tfidf_vect_simple.fit_transform(doc_list)
print(feature_vect_simple.shape)
```

    (3, 18)



```python
feature_vect_dense = feature_vect_simple.todense()
feature_vect_dense
```




    matrix([[0.4155636 , 0.        , 0.4155636 , 0.        , 0.        ,
             0.        , 0.24543856, 0.        , 0.24543856, 0.        ,
             0.        , 0.        , 0.        , 0.4155636 , 0.24543856,
             0.49087711, 0.        , 0.24543856],
            [0.        , 0.        , 0.        , 0.        , 0.        ,
             0.        , 0.23402865, 0.39624495, 0.23402865, 0.        ,
             0.3013545 , 0.        , 0.39624495, 0.        , 0.23402865,
             0.23402865, 0.39624495, 0.4680573 ],
            [0.        , 0.30985601, 0.        , 0.30985601, 0.30985601,
             0.30985601, 0.18300595, 0.        , 0.18300595, 0.30985601,
             0.23565348, 0.30985601, 0.        , 0.        , 0.18300595,
             0.3660119 , 0.        , 0.3660119 ]])




```python
np.array(feature_vect_dense[0]).reshape(-1,)
```




    array([0.4155636 , 0.        , 0.4155636 , 0.        , 0.        ,
           0.        , 0.24543856, 0.        , 0.24543856, 0.        ,
           0.        , 0.        , 0.        , 0.4155636 , 0.24543856,
           0.49087711, 0.        , 0.24543856])




```python
# TFidfVectorizer로 transform()한 결과는 Sparse Matrix이므로 Dense Matrix로 변환. 
feature_vect_dense = feature_vect_simple.todense()

#첫번째 문장과 두번째 문장의 feature vector  추출
vect1 = np.array(feature_vect_dense[0]).reshape(-1,)
vect2 = np.array(feature_vect_dense[1]).reshape(-1,)

#첫번째 문장과 두번째 문장의 feature vector로 두개 문장의 Cosine 유사도 추출
similarity_simple = cos_similarity(vect1, vect2 )
print('문장 1, 문장 2 Cosine 유사도: {0:.3f}'.format(similarity_simple))

```

    문장 1, 문장 2 Cosine 유사도: 0.402



```python
vect1 = np.array(feature_vect_dense[0]).reshape(-1,)
vect3 = np.array(feature_vect_dense[2]).reshape(-1,)
similarity_simple = cos_similarity(vect1, vect3 )
print('문장 1, 문장 3 Cosine 유사도: {0:.3f}'.format(similarity_simple))

vect2 = np.array(feature_vect_dense[1]).reshape(-1,)
vect3 = np.array(feature_vect_dense[2]).reshape(-1,)
similarity_simple = cos_similarity(vect2, vect3 )
print('문장 2, 문장 3 Cosine 유사도: {0:.3f}'.format(similarity_simple))
```

    문장 1, 문장 3 Cosine 유사도: 0.404
    문장 2, 문장 3 Cosine 유사도: 0.456



```python
from sklearn.metrics.pairwise import cosine_similarity

similarity_simple_pair = cosine_similarity(feature_vect_simple[0] , feature_vect_simple)
print(similarity_simple_pair)

```

    [[1.         0.40207758 0.40425045]]



```python
from sklearn.metrics.pairwise import cosine_similarity

similarity_simple_pair = cosine_similarity(feature_vect_simple[0] , feature_vect_simple[1:])
print(similarity_simple_pair)
```

    [[0.40207758 0.40425045]]



```python
similarity_simple_pair = cosine_similarity(feature_vect_simple , feature_vect_simple)
print(similarity_simple_pair)
print('shape:',similarity_simple_pair.shape)
```

    [[1.         0.40207758 0.40425045]
     [0.40207758 1.         0.45647296]
     [0.40425045 0.45647296 1.        ]]
    shape: (3, 3)


### Opinion Review 데이터 셋을 이용한 문서 유사도 측정


```python
from nltk.stem import WordNetLemmatizer
import nltk
import string

remove_punct_dict = dict((ord(punct), None) for punct in string.punctuation)
lemmar = WordNetLemmatizer()

# 입력으로 들어온 token단어들에 대해서 lemmatization 어근 변환. 
def LemTokens(tokens):
    return [lemmar.lemmatize(token) for token in tokens]

# TfidfVectorizer 객체 생성 시 tokenizer인자로 해당 함수를 설정하여 lemmatization 적용
# 입력으로 문장을 받아서 stop words 제거-> 소문자 변환 -> 단어 토큰화 -> lemmatization 어근 변환. 
def LemNormalize(text):
    return LemTokens(nltk.word_tokenize(text.lower().translate(remove_punct_dict)))
```


```python
import pandas as pd
import glob ,os
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.cluster import KMeans
import warnings
warnings.filterwarnings('ignore')

#path = r'C:\Users\q\Text\OpinosisDataset1.0\OpinosisDataset1.0\topics'
path = r'C:\Users\q\PerfectGuide\data\OpinosisDataset1.0\OpinosisDataset1.0\topics'
all_files = glob.glob(os.path.join(path, "*.data"))     
filename_list = []
opinion_text = []

for file_ in all_files:
    df = pd.read_table(file_,index_col=None, header=0,encoding='latin1')
    filename_ = file_.split('\\')[-1]
    filename = filename_.split('.')[0]
    filename_list.append(filename)
    opinion_text.append(df.to_string())

document_df = pd.DataFrame({'filename':filename_list, 'opinion_text':opinion_text})

tfidf_vect = TfidfVectorizer(tokenizer=LemNormalize, stop_words='english' , \
                             ngram_range=(1,2), min_df=0.05, max_df=0.85 )
feature_vect = tfidf_vect.fit_transform(document_df['opinion_text'])

km_cluster = KMeans(n_clusters=3, max_iter=10000, random_state=0)
km_cluster.fit(feature_vect)
cluster_label = km_cluster.labels_
cluster_centers = km_cluster.cluster_centers_
document_df['cluster_label'] = cluster_label
```


```python
document_df.head(51)
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
      <th>filename</th>
      <th>opinion_text</th>
      <th>cluster_label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>accuracy_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>bathroom_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>battery-life_amazon_kindle</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>battery-life_ipod_nano_8gb</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>battery-life_netbook_1005ha</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>buttons_amazon_kindle</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>comfort_honda_accord_2008</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>comfort_toyota_camry_2007</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>directions_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>display_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>eyesight-issues_amazon_kindle</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>features_windows7</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>fonts_amazon_kindle</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>food_holiday_inn_london</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>food_swissotel_chicago</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15</th>
      <td>free_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>16</th>
      <td>gas_mileage_toyota_camry_2007</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>interior_honda_accord_2008</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>interior_toyota_camry_2007</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>keyboard_netbook_1005ha</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>location_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>21</th>
      <td>location_holiday_inn_london</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>22</th>
      <td>mileage_honda_accord_2008</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>navigation_amazon_kindle</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>parking_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>25</th>
      <td>performance_honda_accord_2008</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>performance_netbook_1005ha</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>price_amazon_kindle</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>price_holiday_inn_london</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>29</th>
      <td>quality_toyota_camry_2007</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>30</th>
      <td>rooms_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>31</th>
      <td>rooms_swissotel_chicago</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>32</th>
      <td>room_holiday_inn_london</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>33</th>
      <td>satellite_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>34</th>
      <td>screen_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>screen_ipod_nano_8gb</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>screen_netbook_1005ha</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>37</th>
      <td>seats_honda_accord_2008</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>38</th>
      <td>service_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>39</th>
      <td>service_holiday_inn_london</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>40</th>
      <td>service_swissotel_hotel_chicago</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>41</th>
      <td>size_asus_netbook_1005ha</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>42</th>
      <td>sound_ipod_nano_8gb</td>
      <td>headphone jack i got a clear case for it a...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>43</th>
      <td>speed_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>44</th>
      <td>speed_windows7</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>45</th>
      <td>staff_bestwestern_hotel_sfo</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>46</th>
      <td>staff_swissotel_chicago</td>
      <td>...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>47</th>
      <td>transmission_toyota_camry_2007</td>
      <td>...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>updates_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>49</th>
      <td>video_ipod_nano_8gb</td>
      <td>...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>50</th>
      <td>voice_garmin_nuvi_255W_gps</td>
      <td>...</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
hotel_indexes = document_df[document_df['cluster_label']==2].index
hotel_indexes
```




    Int64Index([1, 13, 14, 15, 20, 21, 24, 28, 30, 31, 32, 38, 39, 40, 45, 46], dtype='int64')




```python
from sklearn.metrics.pairwise import cosine_similarity

# cluster_label=2인 데이터는 호텔로 클러스터링된 데이터임. DataFrame에서 해당 Index를 추출
hotel_indexes = document_df[document_df['cluster_label']==2].index
print('호텔로 클러스터링 된 문서들의 DataFrame Index:', hotel_indexes)

# 호텔로 클러스터링된 데이터 중 첫번째 문서를 추출하여 파일명 표시.  
comparison_docname = document_df.iloc[hotel_indexes[0]]['filename']
print('##### 비교 기준 문서명 ',comparison_docname,' 와 타 문서 유사도######')

''' document_df에서 추출한 Index 객체를 feature_vect로 입력하여 호텔 클러스터링된 feature_vect 추출 
이를 이용하여 호텔로 클러스터링된 문서 중 첫번째 문서와 다른 문서간의 코사인 유사도 측정.'''
similarity_pair = cosine_similarity(feature_vect[hotel_indexes[0]] , feature_vect[hotel_indexes])
print(similarity_pair)

```

    호텔로 클러스터링 된 문서들의 DataFrame Index: Int64Index([1, 13, 14, 15, 20, 21, 24, 28, 30, 31, 32, 38, 39, 40, 45, 46], dtype='int64')
    ##### 비교 기준 문서명  bathroom_bestwestern_hotel_sfo  와 타 문서 유사도######
    [[1.         0.0430688  0.05221059 0.06189595 0.05846178 0.06193118
      0.03638665 0.11742762 0.38038865 0.32619948 0.51442299 0.11282857
      0.13989623 0.1386783  0.09518068 0.07049362]]



```python
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

# argsort()를 이용하여 앞예제의 첫번째 문서와 타 문서간 유사도가 큰 순으로 정렬한 인덱스 추출하되 자기 자신은 제외. 
sorted_index = similarity_pair.argsort()[:,::-1]
sorted_index = sorted_index[:, 1:]

# 유사도가 큰 순으로 hotel_indexes를 추출하여 재 정렬
hotel_sorted_indexes = hotel_indexes[sorted_index.reshape(-1)]

# 유사도가 큰 순으로 유사도 값을 재정렬하되 자기 자신은 제외
hotel_1_sim_value = np.sort(similarity_pair.reshape(-1))[::-1]
hotel_1_sim_value = hotel_1_sim_value[1:]

# 유사도가 큰 순으로 정렬된 인덱스와 유사도 값을 이용해 파일명과 유사도값을 막대 그래프로 시각화
hotel_1_sim_df = pd.DataFrame()
hotel_1_sim_df['filename'] = document_df.iloc[hotel_sorted_indexes]['filename']
hotel_1_sim_df['similarity'] = hotel_1_sim_value
print('가장 유사도가 큰 파일명 및 유사도:\n', hotel_1_sim_df.iloc[0, :])

sns.barplot(x='similarity', y='filename',data=hotel_1_sim_df)
plt.title(comparison_docname)

```

    가장 유사도가 큰 파일명 및 유사도:
     filename      room_holiday_inn_london
    similarity                   0.514423
    Name: 32, dtype: object





    Text(0.5, 1.0, 'bathroom_bestwestern_hotel_sfo')



![output_17_2](/images/2023-12-05-Similarity/output_17_2.png)

