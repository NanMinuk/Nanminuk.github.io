---
title: 문서 클러스터링
tags: NLP
typora-root-url: ../
---

### Opinion Review 데이터 세트를 이용한 문서 군집화

사용 데이터셋: [opinosis · Datasets at Hugging Face](https://huggingface.co/datasets/opinosis)




```python
import pandas as pd
import glob ,os
import warnings 
warnings.filterwarnings('ignore')
pd.set_option('display.max_colwidth', 700)

path = r'C:\Users\q\PerfectGuide\data\OpinosisDataset1.0\OpinosisDataset1.0\topics'

# path로 지정한 디렉토리 밑에 있는 모든 .data 파일들의 파일명을 리스트로 취합
all_files = glob.glob(os.path.join(path, "*.data"))    
filename_list = []
opinion_text = []

# 개별 파일들의 파일명은 filename_list 리스트로 취합, 
# 개별 파일들의 파일내용은 DataFrame로딩 후 다시 string으로 변환하여 opinion_text 리스트로 취합 
for file_ in all_files:
    # 개별 파일을 읽어서 DataFrame으로 생성 
    df = pd.read_table(file_,index_col=None, header=0,encoding='latin1')

    
    # 절대경로로 주어진 file 명을 가공. 만일 Linux에서 수행시에는 아래 \\를 / 변경. 
    # 맨 마지막 .data 확장자도 제거
    filename_ = file_.split('\\')[-1]
    filename = filename_.split('.')[0]

    #파일명 리스트와 파일내용 리스트에 파일명과 파일 내용을 추가. 
    filename_list.append(filename)
    opinion_text.append(df.to_string())

# 파일명 리스트와 파일내용 리스트를  DataFrame으로 생성
document_df = pd.DataFrame({'filename':filename_list, 'opinion_text':opinion_text})
document_df.head()
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>accuracy_garmin_nuvi_255W_gps</td>
      <td>, and is very, very accurate .\n0                                                                                                                                                                           but for the most part, we find that the Garmin software provides accurate directions, whereever we intend to go .\n1                                                                                                              This functi...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>bathroom_bestwestern_hotel_sfo</td>
      <td>The room was not overly big, but clean and very comfortable beds, a great shower and very clean bathrooms .\n0                                                                                                                                                                                                                          The second room was smaller, with a very inconvenient bathroom layout, but at least it was quieter and we were able to sleep .\n1 ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>battery-life_amazon_kindle</td>
      <td>After I plugged it in to my USB hub on my computer to charge the battery the charging cord design is very clever !\n0                                                                                                                                     After you have paged tru a 500, page book one, page, at, a, time to get from Chapter 2 to Chapter 15, see how excited you are about a low battery and all the time it took to get there !\n1                                                     ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>battery-life_ipod_nano_8gb</td>
      <td>short battery life  I moved up from an 8gb .\n0                                                                                                                                                                                                                                                                                            I love this ipod except for the battery life .\n1                             ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>battery-life_netbook_1005ha</td>
      <td>6GHz 533FSB cpu, glossy display, 3, Cell 23Wh Li, ion Battery  , and a 1 .\n0                                                                                                                                                                                                                                                                                              Not to mention that as of now...</td>
    </tr>
  </tbody>
</table>
</div>



#### TfidfVectorizer의 tokenizer인자로 사용될 lemmatization 어근 변환 함수를 설정. 


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

### TF-IDF 기반 Vectorization 적용 및 KMeans 군집화 수행. 
* Stemming과 Lemmatization 같은 어근 변환은 TfidfVectorizer에서 직접 지원하진 않으나 tokenizer 파라미터에 커스텀 어근 변환 함수를 적용하여 어근 변환을 수행할 수 있음
* TfidfVectorizer 생성자의 tokenizer인자로 위에서 생성 LemNormalize 함수 설정.


```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf_vect = TfidfVectorizer(tokenizer=LemNormalize, stop_words='english' , \
                             ngram_range=(1,2), min_df=0.05, max_df=0.85 )

#opinion_text 컬럼값으로 feature vectorization 수행
feature_vect = tfidf_vect.fit_transform(document_df['opinion_text'])

```


```python
from sklearn.cluster import KMeans

# 5개 집합으로 군집화 수행. 예제를 위해 동일한 클러스터링 결과 도출용 random_state=0 
km_cluster = KMeans(n_clusters=5, max_iter=10000, random_state=0)
km_cluster.fit(feature_vect)
cluster_label = km_cluster.labels_
cluster_centers = km_cluster.cluster_centers_
```


```python
km_cluster.labels_
```




    array([2, 0, 1, 1, 1, 2, 4, 4, 2, 2, 2, 2, 2, 3, 3, 3, 4, 4, 4, 1, 3, 3,
           4, 2, 3, 4, 1, 3, 3, 4, 0, 0, 0, 2, 2, 2, 2, 4, 3, 3, 3, 1, 1, 2,
           1, 3, 3, 4, 2, 2, 2])




```python
document_df['cluster_label'] = cluster_label
document_df.head()
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
      <td>, and is very, very accurate .\n0                                                                                                                                                                           but for the most part, we find that the Garmin software provides accurate directions, whereever we intend to go .\n1                                                                                                              This functi...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>bathroom_bestwestern_hotel_sfo</td>
      <td>The room was not overly big, but clean and very comfortable beds, a great shower and very clean bathrooms .\n0                                                                                                                                                                                                                          The second room was smaller, with a very inconvenient bathroom layout, but at least it was quieter and we were able to sleep .\n1 ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>battery-life_amazon_kindle</td>
      <td>After I plugged it in to my USB hub on my computer to charge the battery the charging cord design is very clever !\n0                                                                                                                                     After you have paged tru a 500, page book one, page, at, a, time to get from Chapter 2 to Chapter 15, see how excited you are about a low battery and all the time it took to get there !\n1                                                     ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>battery-life_ipod_nano_8gb</td>
      <td>short battery life  I moved up from an 8gb .\n0                                                                                                                                                                                                                                                                                            I love this ipod except for the battery life .\n1                             ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>battery-life_netbook_1005ha</td>
      <td>6GHz 533FSB cpu, glossy display, 3, Cell 23Wh Li, ion Battery  , and a 1 .\n0                                                                                                                                                                                                                                                                                              Not to mention that as of now...</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
document_df[document_df['cluster_label']==0].sort_values(by='filename')
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
      <th>1</th>
      <td>bathroom_bestwestern_hotel_sfo</td>
      <td>The room was not overly big, but clean and very comfortable beds, a great shower and very clean bathrooms .\n0                                                                                                                                                                                                                          The second room was smaller, with a very inconvenient bathroom layout, but at least it was quieter and we were able to sleep .\n1 ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>room_holiday_inn_london</td>
      <td>We arrived at 23,30 hours and they could not recommend a restaurant so we decided to go to Tesco, with very limited choices but when you are hingry you do not careNext day they rang the bell at 8,00 hours to clean the room, not being very nice being waken up so earlyEvery day they gave u...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>rooms_bestwestern_hotel_sfo</td>
      <td>Great Location ,  Nice   Rooms ,  H...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>rooms_swissotel_chicago</td>
      <td>The Swissotel is one of our favorite hotels in Chicago and the corner rooms have the most fantastic views in the city .\n0                                                                                                                                   The rooms look like they were just remodled and upgraded, there was an HD TV and a nice iHome docking station to put my iPod so I could set the alarm to wake up with my music instead of the radio .\n1                                 ...</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
document_df[document_df['cluster_label']==1].sort_values(by='filename')
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
      <th>2</th>
      <td>battery-life_amazon_kindle</td>
      <td>After I plugged it in to my USB hub on my computer to charge the battery the charging cord design is very clever !\n0                                                                                                                                     After you have paged tru a 500, page book one, page, at, a, time to get from Chapter 2 to Chapter 15, see how excited you are about a low battery and all the time it took to get there !\n1                                                     ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>battery-life_ipod_nano_8gb</td>
      <td>short battery life  I moved up from an 8gb .\n0                                                                                                                                                                                                                                                                                            I love this ipod except for the battery life .\n1                             ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>battery-life_netbook_1005ha</td>
      <td>6GHz 533FSB cpu, glossy display, 3, Cell 23Wh Li, ion Battery  , and a 1 .\n0                                                                                                                                                                                                                                                                                              Not to mention that as of now...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>keyboard_netbook_1005ha</td>
      <td>,  I think the new keyboard rivals the great hp mini keyboards .\n0                                                                                                                                 Since the battery life difference is minimum, the only reason to upgrade would be to get the better keyboard .\n1                                                                                                                                                                                   The keyboard is now as good as t...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>performance_netbook_1005ha</td>
      <td>The Eee Super Hybrid Engine utility lets users overclock or underclock their Eee PC's to boost performance or provide better battery life depending on their immediate requirements .\n0                                                                                                                                                                                                                  In Super Performance mode CPU, Z shows the bus speed to increase up to 169 .\n1                                                                                                                  One...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>41</th>
      <td>size_asus_netbook_1005ha</td>
      <td>A few other things I'd like to point out is that you must push the micro, sized right angle end of the ac adapter until it snaps in place or the battery may not charge .\n0                                                                                                                                                                                                                                                                                                           The full size right shift k...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>42</th>
      <td>sound_ipod_nano_8gb</td>
      <td>headphone jack i got a clear case for it and it  i got a clear case for it and it like prvents me from being able to put the jack all the way in so the sound can b messsed up or i can get it in there and its playing well them go to move or something and it slides out .\n0                                                                                                                                                                                                                 Picture and sound quality are excellent for this typ of devic .\n1                                                                                                                                                 ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>44</th>
      <td>speed_windows7</td>
      <td>Windows 7 is quite simply faster, more stable, boots faster, goes to sleep faster, comes back from sleep faster, manages your files better and on top of that it's beautiful to look at and easy to use .\n0                                                                                                                                                                                                                                           , faster about 20% to 30% faster at running applications than my Vista ,  seriously\n1                                                     ...</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
document_df[document_df['cluster_label']==2].sort_values(by='filename')
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
      <td>, and is very, very accurate .\n0                                                                                                                                                                           but for the most part, we find that the Garmin software provides accurate directions, whereever we intend to go .\n1                                                                                                              This functi...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>buttons_amazon_kindle</td>
      <td>I thought it would be fitting to christen my Kindle with the Stephen King novella UR, so went to the Amazon site on my computer and clicked on the button to buy it .\n0                                                                                                                                                                                                            As soon as I'd clicked the button to confirm my order it appeared on my Kindle almost immediately !\n1                                                                                   ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>directions_garmin_nuvi_255W_gps</td>
      <td>You also get upscale features like spoken directions including street names and programmable POIs .\n0                                                                                                                                                                                                                                                                                                I used to hesitate to go out of my directions but no...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>display_garmin_nuvi_255W_gps</td>
      <td>3 quot  widescreen display was a bonus .\n0                                                                                                                                           This made for smoother graphics on the 255w of the vehicle moving along displayed roads, where the 750's display was more of a  jerky  movement .\n1                                                                                                                         ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10</th>
      <td>eyesight-issues_amazon_kindle</td>
      <td>It feels as easy to read as the K1 but doesn't seem any crisper to my eyes   .\n0                                                                                                                                                            the white is really GREY, and to avoid considerable eye, strain I had to refresh pages   every other page .\n1                                    The dream has always been a portable electronic device that could hold a ton of reading material, automate subscriptions and fa...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>11</th>
      <td>features_windows7</td>
      <td>I had to uninstall anti, virus and selected other programs, some of which did not have listings in the  Programs and Features  Control Panel section .\n0                                                                                                                                                                                                                           This review briefly touches upon some of the key features and enhancements of Microsoft's latest OS .\n1                                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>fonts_amazon_kindle</td>
      <td>Being able to change the font sizes is awesome !\n0                                                                                                                                                                                                                                                                        For whatever reason, Amazon decided to make the Font on the Home Screen   ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>23</th>
      <td>navigation_amazon_kindle</td>
      <td>In fact, the entire navigation structure has been completely revised ,  I'm still getting used to it but it's a huge step forward .\n0                                                                                                                                                                                                                                                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>33</th>
      <td>satellite_garmin_nuvi_255W_gps</td>
      <td>It's fast to acquire satellites .\n0    If you've ever had a  Brand X  GPS take you on some strange route that adds 20 minutes to your trip, has you turn the wrong way down a one way road, tell you to turn AFTER you've passed the street, frequently loses the satellite signal, or has old maps missing streets, you know how important this stuff is .\n1                                                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>34</th>
      <td>screen_garmin_nuvi_255W_gps</td>
      <td>It is easy to read and when touching the screen it works great !\n0                                                                                                                                                                    and zoom out   buttons on the 255w to the same side of the screen which makes it a bit easier .\n1                                                                                                                                                                           ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>35</th>
      <td>screen_ipod_nano_8gb</td>
      <td>As always, the video screen is sharp and bright .\n0                                                                                                                                                                                                        2, inch screen   and a glossy, polished aluminum finish that one CNET editor described as looking like a Christmas tree ornament .\n1                             ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>36</th>
      <td>screen_netbook_1005ha</td>
      <td>Keep in mind that once you get in a room full of light or step outdoors screen reflections could become annoying .\n0                                                                                                                                                                                                                                                                                                       I've used mine outsi...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>43</th>
      <td>speed_garmin_nuvi_255W_gps</td>
      <td>Another feature on the 255w is a display of the posted speed limit on the road which you are currently on right above your current displayed speed .\n0                                                                                                                                                                   I found myself not even looking at my car speedometer as I could easily see my current speed and the speed limit of my route at a glance .\n1                                                                                       ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>48</th>
      <td>updates_garmin_nuvi_255W_gps</td>
      <td>Another thing to consider was that I paid $50 less for the 750 and it came with the FM transmitter cable and a USB cord to connect it to your computer for updates and downloads .\n0                                                                                                                                                                                                update and reroute much _more_ quickly than my other GPS   .\n1                                                                                                       UPDATE ON THIS ,  It finally turned out that to see the elevation contours at lowe...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>49</th>
      <td>video_ipod_nano_8gb</td>
      <td>I bought the 8, gig Ipod Nano that has the built, in video camera .\n0                                                                                                                                                                                                                      Itunes has an on, line store, where you may purchase and download music and videos which will install onto the ipod .\n1                           ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>50</th>
      <td>voice_garmin_nuvi_255W_gps</td>
      <td>The voice prompts and maps are wonderful especially when driving after dark .\n0                                                                                                                                                                               I also thought the the voice prompts of the 750 where more pleasant sounding than the 255w's .\n1                                                                                                                                                       ...</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
document_df[document_df['cluster_label']==3].sort_values(by='filename')
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
      <th>13</th>
      <td>food_holiday_inn_london</td>
      <td>The room was packed to capacity with queues at the food buffets .\n0                                                                                                                                                   The over zealous   staff cleared our unfinished drinks while we were collecting cooked food and movement around the room with plates was difficult in the crowded circumstances .\n1                                         ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>14</th>
      <td>food_swissotel_chicago</td>
      <td>The food for our event was delicious .\n0                                                                                                                                                                                                                                                                                                              ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>15</th>
      <td>free_bestwestern_hotel_sfo</td>
      <td>The wine reception is a great idea as it is nice to meet other travellers and great having access to the free Internet access in our room .\n0                                                                                      They also have a computer available with free internet which is a nice bonus but I didn't find that out till the day before we left but was still able to get on there to check our flight to Vegas the next day .\n1                                                                                                 ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>20</th>
      <td>location_bestwestern_hotel_sfo</td>
      <td>Good Value good location ,  ideal choice .\n0                                                                                                                                                                                                                                                                                          Great Location ,  Nice   Rooms ,  Helpless Concierge\n1                     ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>21</th>
      <td>location_holiday_inn_london</td>
      <td>Great location for tube and we crammed in a fair amount of sightseeing in a short time .\n0                                                                                                                                                                                                                                                                All in all, a normal chain hotel on a nice lo...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>24</th>
      <td>parking_bestwestern_hotel_sfo</td>
      <td>Parking was expensive but I think this is common for San Fran .\n0                                                                                                                                                                                        there is a fee for parking but well worth it seeing no where to park if you do have a car .\n1                                                                                                                                           ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>27</th>
      <td>price_amazon_kindle</td>
      <td>If a case was included, as with the Kindle 1, that would have been reflected in a higher price .\n0                                                                                                                                                                                                                                                                                lower overall price, with nice leather cover .\n1                                                     ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>28</th>
      <td>price_holiday_inn_london</td>
      <td>All in all, a normal chain hotel on a nice location  , I will be back if I do not find anthing closer to Picadilly for a better price .\n0                                                                                                                                                                                                                 ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>38</th>
      <td>service_bestwestern_hotel_sfo</td>
      <td>Both of us having worked in tourism for over 14 years were very disappointed at the level of service provided by this gentleman .\n0                                                                                                                                                                                              The service was good, very friendly staff and we loved the free wine reception each night .\n1                                                                                                                               ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>39</th>
      <td>service_holiday_inn_london</td>
      <td>not customer, oriented hotelvery low service levelboor reception\n0                                                                                                                                                                                                                                                                    The room was quiet, clean, the bed and pillows were comfortable, and the serv...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>40</th>
      <td>service_swissotel_hotel_chicago</td>
      <td>Mediocre room and service for a very extravagant price .\n0                                                                                                                                                                                                                                                                                                             ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>45</th>
      <td>staff_bestwestern_hotel_sfo</td>
      <td>Staff are friendl...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>46</th>
      <td>staff_swissotel_chicago</td>
      <td>The staff at Swissotel were not particularly nice .\n0                                                                                                                                                               Each time I waited at the counter for staff   for several minutes and then was waved to the desk upon my turn with no hello or anything, or apology for waiting in line .\n1                                 ...</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
document_df[document_df['cluster_label']==4].sort_values(by='filename')
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
      <th>6</th>
      <td>comfort_honda_accord_2008</td>
      <td>Drivers seat not comfortable, the car itself compared to other models of similar class .\n0                                                                                                                                                                                                                               ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>comfort_toyota_camry_2007</td>
      <td>Ride seems comfortable and gas mileage fairly good averaging 26 city and 30 open road .\n0                                                                                                        Seats are fine, in fact of all the smaller sedans this is the most comfortable I found for the price as I am 6', 2  and 250# .\n1                                                                                                                                                                                     Great gas mileage and comfortable on long trips ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16</th>
      <td>gas_mileage_toyota_camry_2007</td>
      <td>Ride seems comfortable and gas mileage fairly good averaging 26 city and 30 open road .\n0                                                                                                                                                                                                                                                  ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>17</th>
      <td>interior_honda_accord_2008</td>
      <td>I love the new body style and the interior is a simple pleasure except for the center dash .\n0                                                                                                                                              ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>18</th>
      <td>interior_toyota_camry_2007</td>
      <td>First of all, the interior has way too many cheap plastic parts like the cheap plastic center piece that houses the clock .\n0                                                                                                                                                                       3 blown struts at 30,000 miles, interior trim coming loose and rattling squeaking, stains on paint, and bug splats taking paint off, premature uneven brake wear, on 3rd windsh...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>22</th>
      <td>mileage_honda_accord_2008</td>
      <td>It's quiet, get good gas mileage and looks clean inside and out .\n0                                                                                                                                                                     The mileage is great, and I've had to get used to stopping less for gas .\n1                                                                                                                                                                                                         Thought gas ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>25</th>
      <td>performance_honda_accord_2008</td>
      <td>Very happy with my 08 Accord, performance is quite adequate it has nice looks and is a great long,  distance cruiser .\n0                                                                                                                                 6, 4, 3 eco engine has poor performance and gas mileage of 22 highway .\n1                                                                                                                                                 Overall performance is good but comfort level is poor .\n2                                                                                      ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>29</th>
      <td>quality_toyota_camry_2007</td>
      <td>I previously owned a Toyota 4Runner which had incredible build quality and reliability .\n0                                                                                                                                                                                                                                                                                               I bought the Camry because of Toyota reliability and qua...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>37</th>
      <td>seats_honda_accord_2008</td>
      <td>Front seats are very uncomfortable .\n0                                                                                                                                                                                                                                  No memory seats, no trip computer, can only display outside temp with trip odometer .\n1                                                                   ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>47</th>
      <td>transmission_toyota_camry_2007</td>
      <td>After slowing down, transmission has to be  kicked  to speed up .\n0                                                                                                                                                                                                                                                                       ...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
from sklearn.cluster import KMeans

# 3개의 집합으로 군집화 
km_cluster = KMeans(n_clusters=3, max_iter=10000, random_state=0)
km_cluster.fit(feature_vect)
cluster_label = km_cluster.labels_


# 소속 클러스터를 cluster_label 컬럼으로 할당하고 cluster_label 값으로 정렬
document_df['cluster_label'] = cluster_label
document_df.sort_values(by='cluster_label')
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
      <td>, and is very, very accurate .\n0                                                                                                                                                                           but for the most part, we find that the Garmin software provides accurate directions, whereever we intend to go .\n1                                                                                                              This functi...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>48</th>
      <td>updates_garmin_nuvi_255W_gps</td>
      <td>Another thing to consider was that I paid $50 less for the 750 and it came with the FM transmitter cable and a USB cord to connect it to your computer for updates and downloads .\n0                                                                                                                                                                                                update and reroute much _more_ quickly than my other GPS   .\n1                                                                                                       UPDATE ON THIS ,  It finally turned out that to see the elevation contours at lowe...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>44</th>
      <td>speed_windows7</td>
      <td>Windows 7 is quite simply faster, more stable, boots faster, goes to sleep faster, comes back from sleep faster, manages your files better and on top of that it's beautiful to look at and easy to use .\n0                                                                                                                                                                                                                                           , faster about 20% to 30% faster at running applications than my Vista ,  seriously\n1                                                     ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>43</th>
      <td>speed_garmin_nuvi_255W_gps</td>
      <td>Another feature on the 255w is a display of the posted speed limit on the road which you are currently on right above your current displayed speed .\n0                                                                                                                                                                   I found myself not even looking at my car speedometer as I could easily see my current speed and the speed limit of my route at a glance .\n1                                                                                       ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>42</th>
      <td>sound_ipod_nano_8gb</td>
      <td>headphone jack i got a clear case for it and it  i got a clear case for it and it like prvents me from being able to put the jack all the way in so the sound can b messsed up or i can get it in there and its playing well them go to move or something and it slides out .\n0                                                                                                                                                                                                                 Picture and sound quality are excellent for this typ of devic .\n1                                                                                                                                                 ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>41</th>
      <td>size_asus_netbook_1005ha</td>
      <td>A few other things I'd like to point out is that you must push the micro, sized right angle end of the ac adapter until it snaps in place or the battery may not charge .\n0                                                                                                                                                                                                                                                                                                           The full size right shift k...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>screen_netbook_1005ha</td>
      <td>Keep in mind that once you get in a room full of light or step outdoors screen reflections could become annoying .\n0                                                                                                                                                                                                                                                                                                       I've used mine outsi...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>screen_ipod_nano_8gb</td>
      <td>As always, the video screen is sharp and bright .\n0                                                                                                                                                                                                        2, inch screen   and a glossy, polished aluminum finish that one CNET editor described as looking like a Christmas tree ornament .\n1                             ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>34</th>
      <td>screen_garmin_nuvi_255W_gps</td>
      <td>It is easy to read and when touching the screen it works great !\n0                                                                                                                                                                    and zoom out   buttons on the 255w to the same side of the screen which makes it a bit easier .\n1                                                                                                                                                                           ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>33</th>
      <td>satellite_garmin_nuvi_255W_gps</td>
      <td>It's fast to acquire satellites .\n0    If you've ever had a  Brand X  GPS take you on some strange route that adds 20 minutes to your trip, has you turn the wrong way down a one way road, tell you to turn AFTER you've passed the street, frequently loses the satellite signal, or has old maps missing streets, you know how important this stuff is .\n1                                                                 ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>price_amazon_kindle</td>
      <td>If a case was included, as with the Kindle 1, that would have been reflected in a higher price .\n0                                                                                                                                                                                                                                                                                lower overall price, with nice leather cover .\n1                                                     ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>performance_netbook_1005ha</td>
      <td>The Eee Super Hybrid Engine utility lets users overclock or underclock their Eee PC's to boost performance or provide better battery life depending on their immediate requirements .\n0                                                                                                                                                                                                                  In Super Performance mode CPU, Z shows the bus speed to increase up to 169 .\n1                                                                                                                  One...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>49</th>
      <td>video_ipod_nano_8gb</td>
      <td>I bought the 8, gig Ipod Nano that has the built, in video camera .\n0                                                                                                                                                                                                                      Itunes has an on, line store, where you may purchase and download music and videos which will install onto the ipod .\n1                           ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>navigation_amazon_kindle</td>
      <td>In fact, the entire navigation structure has been completely revised ,  I'm still getting used to it but it's a huge step forward .\n0                                                                                                                                                                                                                                                                 ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>keyboard_netbook_1005ha</td>
      <td>,  I think the new keyboard rivals the great hp mini keyboards .\n0                                                                                                                                 Since the battery life difference is minimum, the only reason to upgrade would be to get the better keyboard .\n1                                                                                                                                                                                   The keyboard is now as good as t...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>50</th>
      <td>voice_garmin_nuvi_255W_gps</td>
      <td>The voice prompts and maps are wonderful especially when driving after dark .\n0                                                                                                                                                                               I also thought the the voice prompts of the 750 where more pleasant sounding than the 255w's .\n1                                                                                                                                                       ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>display_garmin_nuvi_255W_gps</td>
      <td>3 quot  widescreen display was a bonus .\n0                                                                                                                                           This made for smoother graphics on the 255w of the vehicle moving along displayed roads, where the 750's display was more of a  jerky  movement .\n1                                                                                                                         ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>battery-life_amazon_kindle</td>
      <td>After I plugged it in to my USB hub on my computer to charge the battery the charging cord design is very clever !\n0                                                                                                                                     After you have paged tru a 500, page book one, page, at, a, time to get from Chapter 2 to Chapter 15, see how excited you are about a low battery and all the time it took to get there !\n1                                                     ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>battery-life_ipod_nano_8gb</td>
      <td>short battery life  I moved up from an 8gb .\n0                                                                                                                                                                                                                                                                                            I love this ipod except for the battery life .\n1                             ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>battery-life_netbook_1005ha</td>
      <td>6GHz 533FSB cpu, glossy display, 3, Cell 23Wh Li, ion Battery  , and a 1 .\n0                                                                                                                                                                                                                                                                                              Not to mention that as of now...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>buttons_amazon_kindle</td>
      <td>I thought it would be fitting to christen my Kindle with the Stephen King novella UR, so went to the Amazon site on my computer and clicked on the button to buy it .\n0                                                                                                                                                                                                            As soon as I'd clicked the button to confirm my order it appeared on my Kindle almost immediately !\n1                                                                                   ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>fonts_amazon_kindle</td>
      <td>Being able to change the font sizes is awesome !\n0                                                                                                                                                                                                                                                                        For whatever reason, Amazon decided to make the Font on the Home Screen   ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>features_windows7</td>
      <td>I had to uninstall anti, virus and selected other programs, some of which did not have listings in the  Programs and Features  Control Panel section .\n0                                                                                                                                                                                                                           This review briefly touches upon some of the key features and enhancements of Microsoft's latest OS .\n1                                                 ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>eyesight-issues_amazon_kindle</td>
      <td>It feels as easy to read as the K1 but doesn't seem any crisper to my eyes   .\n0                                                                                                                                                            the white is really GREY, and to avoid considerable eye, strain I had to refresh pages   every other page .\n1                                    The dream has always been a portable electronic device that could hold a ton of reading material, automate subscriptions and fa...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>directions_garmin_nuvi_255W_gps</td>
      <td>You also get upscale features like spoken directions including street names and programmable POIs .\n0                                                                                                                                                                                                                                                                                                I used to hesitate to go out of my directions but no...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>47</th>
      <td>transmission_toyota_camry_2007</td>
      <td>After slowing down, transmission has to be  kicked  to speed up .\n0                                                                                                                                                                                                                                                                       ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>37</th>
      <td>seats_honda_accord_2008</td>
      <td>Front seats are very uncomfortable .\n0                                                                                                                                                                                                                                  No memory seats, no trip computer, can only display outside temp with trip odometer .\n1                                                                   ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>comfort_honda_accord_2008</td>
      <td>Drivers seat not comfortable, the car itself compared to other models of similar class .\n0                                                                                                                                                                                                                               ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>comfort_toyota_camry_2007</td>
      <td>Ride seems comfortable and gas mileage fairly good averaging 26 city and 30 open road .\n0                                                                                                        Seats are fine, in fact of all the smaller sedans this is the most comfortable I found for the price as I am 6', 2  and 250# .\n1                                                                                                                                                                                     Great gas mileage and comfortable on long trips ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>gas_mileage_toyota_camry_2007</td>
      <td>Ride seems comfortable and gas mileage fairly good averaging 26 city and 30 open road .\n0                                                                                                                                                                                                                                                  ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>25</th>
      <td>performance_honda_accord_2008</td>
      <td>Very happy with my 08 Accord, performance is quite adequate it has nice looks and is a great long,  distance cruiser .\n0                                                                                                                                 6, 4, 3 eco engine has poor performance and gas mileage of 22 highway .\n1                                                                                                                                                 Overall performance is good but comfort level is poor .\n2                                                                                      ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>interior_honda_accord_2008</td>
      <td>I love the new body style and the interior is a simple pleasure except for the center dash .\n0                                                                                                                                              ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>interior_toyota_camry_2007</td>
      <td>First of all, the interior has way too many cheap plastic parts like the cheap plastic center piece that houses the clock .\n0                                                                                                                                                                       3 blown struts at 30,000 miles, interior trim coming loose and rattling squeaking, stains on paint, and bug splats taking paint off, premature uneven brake wear, on 3rd windsh...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>mileage_honda_accord_2008</td>
      <td>It's quiet, get good gas mileage and looks clean inside and out .\n0                                                                                                                                                                     The mileage is great, and I've had to get used to stopping less for gas .\n1                                                                                                                                                                                                         Thought gas ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>29</th>
      <td>quality_toyota_camry_2007</td>
      <td>I previously owned a Toyota 4Runner which had incredible build quality and reliability .\n0                                                                                                                                                                                                                                                                                               I bought the Camry because of Toyota reliability and qua...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>bathroom_bestwestern_hotel_sfo</td>
      <td>The room was not overly big, but clean and very comfortable beds, a great shower and very clean bathrooms .\n0                                                                                                                                                                                                                          The second room was smaller, with a very inconvenient bathroom layout, but at least it was quieter and we were able to sleep .\n1 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>46</th>
      <td>staff_swissotel_chicago</td>
      <td>The staff at Swissotel were not particularly nice .\n0                                                                                                                                                               Each time I waited at the counter for staff   for several minutes and then was waved to the desk upon my turn with no hello or anything, or apology for waiting in line .\n1                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>45</th>
      <td>staff_bestwestern_hotel_sfo</td>
      <td>Staff are friendl...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>food_swissotel_chicago</td>
      <td>The food for our event was delicious .\n0                                                                                                                                                                                                                                                                                                              ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20</th>
      <td>location_bestwestern_hotel_sfo</td>
      <td>Good Value good location ,  ideal choice .\n0                                                                                                                                                                                                                                                                                          Great Location ,  Nice   Rooms ,  Helpless Concierge\n1                     ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>21</th>
      <td>location_holiday_inn_london</td>
      <td>Great location for tube and we crammed in a fair amount of sightseeing in a short time .\n0                                                                                                                                                                                                                                                                All in all, a normal chain hotel on a nice lo...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>30</th>
      <td>rooms_bestwestern_hotel_sfo</td>
      <td>Great Location ,  Nice   Rooms ,  H...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38</th>
      <td>service_bestwestern_hotel_sfo</td>
      <td>Both of us having worked in tourism for over 14 years were very disappointed at the level of service provided by this gentleman .\n0                                                                                                                                                                                              The service was good, very friendly staff and we loved the free wine reception each night .\n1                                                                                                                               ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>13</th>
      <td>food_holiday_inn_london</td>
      <td>The room was packed to capacity with queues at the food buffets .\n0                                                                                                                                                   The over zealous   staff cleared our unfinished drinks while we were collecting cooked food and movement around the room with plates was difficult in the crowded circumstances .\n1                                         ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>parking_bestwestern_hotel_sfo</td>
      <td>Parking was expensive but I think this is common for San Fran .\n0                                                                                                                                                                                        there is a fee for parking but well worth it seeing no where to park if you do have a car .\n1                                                                                                                                           ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>28</th>
      <td>price_holiday_inn_london</td>
      <td>All in all, a normal chain hotel on a nice location  , I will be back if I do not find anthing closer to Picadilly for a better price .\n0                                                                                                                                                                                                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15</th>
      <td>free_bestwestern_hotel_sfo</td>
      <td>The wine reception is a great idea as it is nice to meet other travellers and great having access to the free Internet access in our room .\n0                                                                                      They also have a computer available with free internet which is a nice bonus but I didn't find that out till the day before we left but was still able to get on there to check our flight to Vegas the next day .\n1                                                                                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>32</th>
      <td>room_holiday_inn_london</td>
      <td>We arrived at 23,30 hours and they could not recommend a restaurant so we decided to go to Tesco, with very limited choices but when you are hingry you do not careNext day they rang the bell at 8,00 hours to clean the room, not being very nice being waken up so earlyEvery day they gave u...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>31</th>
      <td>rooms_swissotel_chicago</td>
      <td>The Swissotel is one of our favorite hotels in Chicago and the corner rooms have the most fantastic views in the city .\n0                                                                                                                                   The rooms look like they were just remodled and upgraded, there was an HD TV and a nice iHome docking station to put my iPod so I could set the alarm to wake up with my music instead of the radio .\n1                                 ...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>39</th>
      <td>service_holiday_inn_london</td>
      <td>not customer, oriented hotelvery low service levelboor reception\n0                                                                                                                                                                                                                                                                    The room was quiet, clean, the bed and pillows were comfortable, and the serv...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>40</th>
      <td>service_swissotel_hotel_chicago</td>
      <td>Mediocre room and service for a very extravagant price .\n0                                                                                                                                                                                                                                                                                                             ...</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



### 군집(Cluster)별 핵심 단어 추출하기



```python
feature_vect.shape
```




    (51, 4611)




```python
cluster_centers = km_cluster.cluster_centers_
print('cluster_centers shape :',cluster_centers.shape)
print(cluster_centers)
```

    cluster_centers shape : (3, 4611)
    [[0.01005322 0.         0.         ... 0.00706287 0.         0.        ]
     [0.         0.00092551 0.         ... 0.         0.         0.        ]
     [0.         0.00099499 0.00174637 ... 0.         0.00183397 0.00144581]]



```python
# 군집별 top n 핵심단어, 그 단어의 중심 위치 상대값, 대상 파일명들을 반환함. 
def get_cluster_details(cluster_model, cluster_data, feature_names, clusters_num, top_n_features=10):
    cluster_details = {}
    
    # cluster_centers array 의 값이 큰 순으로 정렬된 index 값을 반환
    # 군집 중심점(centroid)별 할당된 word 피처들의 거리값이 큰 순으로 값을 구하기 위함.  
    centroid_feature_ordered_ind = cluster_model.cluster_centers_.argsort()[:,::-1]
    
    #개별 군집별로 iteration하면서 핵심단어, 그 단어의 중심 위치 상대값, 대상 파일명 입력
    for cluster_num in range(clusters_num):
        # 개별 군집별 정보를 담을 데이터 초기화. 
        cluster_details[cluster_num] = {}
        cluster_details[cluster_num]['cluster'] = cluster_num
        
        # cluster_centers_.argsort()[:,::-1] 로 구한 index 를 이용하여 top n 피처 단어를 구함. 
        top_feature_indexes = centroid_feature_ordered_ind[cluster_num, :top_n_features]
        top_features = [ feature_names[ind] for ind in top_feature_indexes ]
        
        # top_feature_indexes를 이용해 해당 피처 단어의 중심 위치 상댓값 구함 
        top_feature_values = cluster_model.cluster_centers_[cluster_num, top_feature_indexes].tolist()
        
        # cluster_details 딕셔너리 객체에 개별 군집별 핵심 단어와 중심위치 상대값, 그리고 해당 파일명 입력
        cluster_details[cluster_num]['top_features'] = top_features
        cluster_details[cluster_num]['top_features_value'] = top_feature_values
        filenames = cluster_data[cluster_data['cluster_label'] == cluster_num]['filename']
        filenames = filenames.values.tolist()
        cluster_details[cluster_num]['filenames'] = filenames
        
    return cluster_details

```


```python
def print_cluster_details(cluster_details):
    for cluster_num, cluster_detail in cluster_details.items():
        print('####### Cluster {0}'.format(cluster_num))
        print('Top features:', cluster_detail['top_features'])
        print('Reviews 파일명 :',cluster_detail['filenames'][:7])
        print('==================================================')

```


```python
feature_names = tfidf_vect.get_feature_names()

cluster_details = get_cluster_details(cluster_model=km_cluster, cluster_data=document_df,\
                                  feature_names=feature_names, clusters_num=3, top_n_features=10 )
print_cluster_details(cluster_details)
```

    ####### Cluster 0
    Top features: ['screen', 'battery', 'keyboard', 'battery life', 'life', 'kindle', 'direction', 'video', 'size', 'voice']
    Top features value: [0.13738120192560713, 0.12029588637881046, 0.06416898221290011, 0.06330587553655656, 0.058609811414901014, 0.05646823079053855, 0.05393721234628231, 0.05389363145594924, 0.05127615400589759, 0.05119379623996657]
    Reviews 파일명 : ['accuracy_garmin_nuvi_255W_gps', 'battery-life_amazon_kindle', 'battery-life_ipod_nano_8gb', 'battery-life_netbook_1005ha', 'buttons_amazon_kindle', 'directions_garmin_nuvi_255W_gps', 'display_garmin_nuvi_255W_gps']
    ==================================================
    ####### Cluster 1
    Top features: ['interior', 'seat', 'mileage', 'comfortable', 'gas', 'gas mileage', 'transmission', 'car', 'performance', 'quality']
    Top features value: [0.2278046916838381, 0.1939512034790141, 0.17730089408820418, 0.12790439933840922, 0.12265896733770874, 0.1165812335567078, 0.10486543788164414, 0.1015256988633988, 0.09562358109597213, 0.09233847022967973]
    Reviews 파일명 : ['comfort_honda_accord_2008', 'comfort_toyota_camry_2007', 'gas_mileage_toyota_camry_2007', 'interior_honda_accord_2008', 'interior_toyota_camry_2007', 'mileage_honda_accord_2008', 'performance_honda_accord_2008']
    ==================================================
    ####### Cluster 2
    Top features: ['room', 'hotel', 'service', 'staff', 'food', 'location', 'bathroom', 'clean', 'price', 'parking']
    Top features value: [0.26083463293785525, 0.20029241494780795, 0.17546706020591651, 0.14901486167147948, 0.1277265040158775, 0.12576707418420113, 0.07042395128821588, 0.06971189120637236, 0.059533968934674365, 0.05842875064534637]
    Reviews 파일명 : ['bathroom_bestwestern_hotel_sfo', 'food_holiday_inn_london', 'food_swissotel_chicago', 'free_bestwestern_hotel_sfo', 'location_bestwestern_hotel_sfo', 'location_holiday_inn_london', 'parking_bestwestern_hotel_sfo']
    ==================================================



```python

```
