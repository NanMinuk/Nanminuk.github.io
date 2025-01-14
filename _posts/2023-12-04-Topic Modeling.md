---
title: 토픽 모델링
tags: NLP
typora-root-url: ../
---

### Newsgroup 토픽 모델링

**20개 중 8개의 주제 데이터 로드 및 Count기반 피처 벡터화. LDA는 Count기반 Vectorizer만 적용**


```python
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.decomposition import LatentDirichletAllocation

# 모토사이클, 야구, 그래픽스, 윈도우즈, 중동, 기독교, 전자공학, 의학 등 8개 주제를 추출. 
cats = ['rec.motorcycles', 'rec.sport.baseball', 'comp.graphics', 'comp.windows.x',
        'talk.politics.mideast', 'soc.religion.christian', 'sci.electronics', 'sci.med'  ]

# 위에서 cats 변수로 기재된 category만 추출. featch_20newsgroups( )의 categories에 cats 입력
news_df = fetch_20newsgroups(subset='all',remove=('headers', 'footers', 'quotes'), 
                            categories=cats, random_state=0)

#LDA 는 Count기반의 Vectorizer만 적용합니다.  
count_vect = CountVectorizer(max_df=0.95, max_features=1000, min_df=2, stop_words='english', ngram_range=(1,2))
feat_vect = count_vect.fit_transform(news_df.data)
print('CountVectorizer Shape:', feat_vect.shape)
```

    CountVectorizer Shape: (7862, 1000)



```python
count_vect.get_feature_names()
```

    C:\Users\q\anaconda3\lib\site-packages\sklearn\utils\deprecation.py:87: FutureWarning: Function get_feature_names is deprecated; get_feature_names is deprecated in 1.0 and will be removed in 1.2. Please use get_feature_names_out instead.
      warnings.warn(msg, category=FutureWarning)





    ['00',
     '000',
     '01',
     '02',
     '03',
     '04',
     '05',
     '10',
     '100',
     '11',
     '12',
     '128',
     '13',
     '14',
     '15',
     '16',
     '17',
     '18',
     '19',
     '1990',
     '1991',
     '1992',
     '1993',
     '20',
     '200',
     '21',
     '22',
     '23',
     '24',
     '24 bit',
     '25',
     '256',
     '26',
     '27',
     '28',
     '29',
     '30',
     '300',
     '31',
     '32',
     '35',
     '3d',
     '40',
     '44',
     '50',
     '500',
     '60',
     '80',
     '800',
     '90',
     '91',
     '92',
     '93',
     'ability',
     'able',
     'ac',
     'accept',
     'accepted',
     'access',
     'according',
     'act',
     'action',
     'actions',
     'acts',
     'actually',
     'add',
     'added',
     'addition',
     'address',
     'adl',
     'advance',
     'age',
     'ago',
     'agree',
     'aids',
     'al',
     'allow',
     'american',
     'amiga',
     'analysis',
     'anonymous',
     'anonymous ftp',
     'answer',
     'answers',
     'anti',
     'anybody',
     'apartment',
     'apparently',
     'appear',
     'appears',
     'application',
     'applications',
     'apply',
     'appreciate',
     'appreciated',
     'approach',
     'appropriate',
     'april',
     'arab',
     'arabs',
     'archive',
     'area',
     'areas',
     'aren',
     'argic',
     'argument',
     'armenia',
     'armenian',
     'armenians',
     'army',
     'art',
     'article',
     'articles',
     'ask',
     'asked',
     'assume',
     'attack',
     'attempt',
     'author',
     'available',
     'average',
     'avoid',
     'away',
     'azerbaijan',
     'azerbaijani',
     'bad',
     'ball',
     'base',
     'baseball',
     'based',
     'basic',
     'basis',
     'battery',
     'belief',
     'believe',
     'best',
     'better',
     'bible',
     'big',
     'bike',
     'bit',
     'bits',
     'black',
     'blood',
     'blue',
     'board',
     'body',
     'book',
     'books',
     'born',
     'box',
     'break',
     'bring',
     'british',
     'brought',
     'build',
     'building',
     'built',
     'bus',
     'buy',
     'ca',
     'california',
     'called',
     'came',
     'cancer',
     'car',
     'card',
     'care',
     'case',
     'cases',
     'catholic',
     'cause',
     'center',
     'century',
     'certain',
     'certainly',
     'chance',
     'change',
     'changed',
     'changes',
     'char',
     'check',
     'child',
     'children',
     'chip',
     'choice',
     'christ',
     'christian',
     'christianity',
     'christians',
     'church',
     'circuit',
     'city',
     'claim',
     'claims',
     'class',
     'clear',
     'clearly',
     'client',
     'clients',
     'clinical',
     'close',
     'code',
     'color',
     'colors',
     'com',
     'come',
     'comes',
     'coming',
     'command',
     'comments',
     'commercial',
     'common',
     'community',
     'comp',
     'company',
     'complete',
     'completely',
     'computer',
     'conference',
     'consider',
     'considered',
     'contact',
     'contains',
     'context',
     'continue',
     'contrib',
     'control',
     'conversion',
     'convert',
     'copy',
     'correct',
     'cost',
     'couldn',
     'count',
     'countries',
     'country',
     'couple',
     'course',
     'create',
     'created',
     'cs',
     'culture',
     'current',
     'currently',
     'cut',
     'data',
     'date',
     'david',
     'day',
     'days',
     'dead',
     'deal',
     'death',
     'dec',
     'default',
     'define',
     'defined',
     'definition',
     'deleted',
     'department',
     'described',
     'description',
     'design',
     'designed',
     'details',
     'developed',
     'development',
     'did',
     'didn',
     'die',
     'died',
     'diet',
     'difference',
     'different',
     'difficult',
     'digital',
     'directly',
     'directory',
     'discussion',
     'disease',
     'disk',
     'display',
     'distribution',
     'doctor',
     'dod',
     'does',
     'does know',
     'doesn',
     'dog',
     'doing',
     'don',
     'don know',
     'don think',
     'door',
     'dos',
     'dos dos',
     'double',
     'doubt',
     'dr',
     'draw',
     'drive',
     'driver',
     'drug',
     'earlier',
     'early',
     'earth',
     'easily',
     'east',
     'easy',
     'echo',
     'ed',
     'edu',
     'effect',
     'effects',
     'email',
     'end',
     'entire',
     'entries',
     'entry',
     'environment',
     'error',
     'especially',
     'europe',
     'event',
     'events',
     'evidence',
     'exactly',
     'example',
     'exist',
     'existence',
     'expect',
     'experience',
     'export',
     'extra',
     'eye',
     'eyes',
     'face',
     'fact',
     'fairly',
     'faith',
     'false',
     'family',
     'faq',
     'far',
     'fast',
     'faster',
     'father',
     'fax',
     'features',
     'feel',
     'field',
     'figure',
     'file',
     'files',
     'final',
     'finally',
     'fine',
     'follow',
     'following',
     'font',
     'fonts',
     'food',
     'force',
     'form',
     'format',
     'formats',
     'frame',
     'free',
     'friend',
     'ftp',
     'function',
     'functions',
     'future',
     'game',
     'games',
     'gave',
     'general',
     'generally',
     'genocide',
     'gets',
     'getting',
     'gif',
     'given',
     'gives',
     'giving',
     'god',
     'goes',
     'going',
     'gone',
     'good',
     'got',
     'gov',
     'government',
     'graphics',
     'great',
     'greatly',
     'greece',
     'greek',
     'green',
     'ground',
     'group',
     'groups',
     'guess',
     'guy',
     'guys',
     'half',
     'hand',
     'handle',
     'hands',
     'happen',
     'happened',
     'happens',
     'happy',
     'hard',
     'hardware',
     'hate',
     'haven',
     'having',
     'head',
     'health',
     'hear',
     'heard',
     'heart',
     'heaven',
     'held',
     'hell',
     'help',
     'hi',
     'high',
     'higher',
     'history',
     'hit',
     'hiv',
     'hold',
     'holy',
     'home',
     'homosexual',
     'homosexuality',
     'hope',
     'hospital',
     'host',
     'hot',
     'hours',
     'house',
     'hp',
     'human',
     'ibm',
     'idea',
     'ideas',
     'ii',
     'image',
     'images',
     'imagine',
     'important',
     'include',
     'included',
     'includes',
     'including',
     'individual',
     'infection',
     'info',
     'information',
     'input',
     'inside',
     'instead',
     'int',
     'interested',
     'interesting',
     'interface',
     'international',
     'internet',
     'involved',
     'isn',
     'israel',
     'israeli',
     'issue',
     'issues',
     'istanbul',
     'jesus',
     'jewish',
     'jews',
     'job',
     'john',
     'jpeg',
     'just',
     'key',
     'keyboard',
     'kill',
     'killed',
     'kind',
     'knew',
     'know',
     'knowledge',
     'known',
     'knows',
     'kuwait',
     'land',
     'language',
     'large',
     'later',
     'latest',
     'law',
     'lcs',
     'lcs mit',
     'lead',
     'league',
     'learn',
     'leave',
     'led',
     'left',
     'let',
     'level',
     'lib',
     'library',
     'life',
     'light',
     'like',
     'likely',
     'limited',
     'line',
     'lines',
     'list',
     'little',
     'live',
     'lived',
     'lives',
     'living',
     'll',
     'local',
     'long',
     'longer',
     'look',
     'looked',
     'looking',
     'looks',
     'lord',
     'lost',
     'lot',
     'lots',
     'love',
     'low',
     'mac',
     'machine',
     'machines',
     'mail',
     'mailing',
     'main',
     'major',
     'make',
     'makes',
     'making',
     'man',
     'manager',
     'manual',
     'mark',
     'marriage',
     'mary',
     'matter',
     'maybe',
     'mean',
     'meaning',
     'means',
     'medical',
     'medicine',
     'member',
     'members',
     'memory',
     'men',
     'mentioned',
     'message',
     'method',
     'michael',
     'middle',
     'mike',
     'military',
     'million',
     'mind',
     'mit',
     'mit edu',
     'mode',
     'model',
     'modern',
     'money',
     'months',
     'mother',
     'motif',
     'mouse',
     'movement',
     'mr',
     'ms',
     'msg',
     'multiple',
     'muslim',
     'muslims',
     'national',
     'nature',
     'nazi',
     'near',
     'necessary',
     'need',
     'needed',
     'needs',
     'net',
     'network',
     'new',
     'new york',
     'news',
     'newsgroup',
     'nice',
     'night',
     'non',
     'normal',
     'note',
     'null',
     'number',
     'numbers',
     'object',
     'objects',
     'obviously',
     'office',
     'official',
     'oh',
     'ok',
     'old',
     'ones',
     'open',
     'openwindows',
     'opinion',
     'order',
     'organization',
     'original',
     'os',
     'ottoman',
     'output',
     'outside',
     'package',
     'page',
     'pages',
     'pain',
     'palestinian',
     'palestinians',
     'paper',
     'parents',
     'particular',
     'parts',
     'party',
     'pass',
     'past',
     'path',
     'patient',
     'patients',
     'paul',
     'pc',
     'peace',
     'people',
     'percent',
     'performance',
     'person',
     'personal',
     'phone',
     'picture',
     'pixel',
     'place',
     'places',
     'play',
     'player',
     'players',
     'plus',
     'point',
     'points',
     'police',
     'policy',
     'political',
     'population',
     'position',
     'possible',
     'possibly',
     'post',
     'posted',
     'posting',
     'postscript',
     'power',
     'practice',
     'present',
     'press',
     'pretty',
     'prevent',
     'previous',
     'price',
     'printf',
     'probably',
     'problem',
     'problems',
     'process',
     'processing',
     'product',
     'products',
     'professor',
     'program',
     'programming',
     'programs',
     'project',
     'provide',
     'provided',
     'provides',
     'pub',
     'public',
     'published',
     'purpose',
     'quality',
     'question',
     'questions',
     'quite',
     'r5',
     'radio',
     'range',
     'rate',
     'ray',
     'read',
     'reading',
     'real',
     'really',
     'reason',
     'reasonable',
     'reasons',
     'receive',
     'received',
     'recent',
     'recently',
     'red',
     'reference',
     'references',
     'regarding',
     'related',
     'release',
     'religion',
     'religious',
     'remember',
     'remote',
     'reply',
     'report',
     'reported',
     'reports',
     'request',
     'require',
     'required',
     'requires',
     'research',
     'resource',
     'resources',
     'response',
     'rest',
     'result',
     'results',
     'return',
     'ride',
     'riding',
     'right',
     'rights',
     'risk',
     'road',
     'room',
     'rule',
     'rules',
     'run',
     'running',
     'runs',
     'russian',
     'said',
     'san',
     'save',
     'saw',
     'say',
     'saying',
     'says',
     'school',
     'science',
     'scientific',
     'screen',
     'scripture',
     'search',
     'season',
     'second',
     'section',
     'seen',
     'self',
     'send',
     'sense',
     'sent',
     'serdar',
     'serdar argic',
     'series',
     'server',
     'service',
     'set',
     'setting',
     'sex',
     'sgi',
     'shall',
     'shell',
     'short',
     'shows',
     'signal',
     'similar',
     'simple',
     'simply',
     'sin',
     'single',
     'site',
     'sites',
     'situation',
     'size',
     'small',
     'society',
     'software',
     'soldiers',
     'solution',
     'son',
     'soon',
     'sorry',
     'sort',
     'sound',
     'sounds',
     'source',
     'sources',
     'soviet',
     'space',
     'speak',
     'special',
     'specific',
     'speed',
     'spirit',
     'st',
     'standard',
     'start',
     'started',
     'starting',
     'state',
     'statement',
     'states',
     'stay',
     'steve',
     'stop',
     'story',
     'street',
     'string',
     'strong',
     'studies',
     'study',
     'stuff',
     'subject',
     'suggest',
     'suggestions',
     'sun',
     'support',
     'supported',
     'supports',
     'sure',
     'systems',
     'table',
     'taken',
     'takes',
     'taking',
     'talk',
     'talking',
     'tape',
     'tar',
     'team',
     'technology',
     'tell',
     'term',
     'terms',
     'test',
     'text',
     'thank',
     'thanks',
     'thanks advance',
     'theory',
     'thing',
     'things',
     'think',
     'thinking',
     'thought',
     'tiff',
     'time',
     'times',
     'title',
     'today',
     'told',
     'took',
     'tool',
     'toolkit',
     'tools',
     'total',
     'town',
     'treatment',
     'tried',
     'troops',
     'trouble',
     'true',
     'truth',
     'try',
     'trying',
     'turkey',
     'turkish',
     'turks',
     'turn',
     'turned',
     'tv',
     'type',
     'types',
     'uk',
     'understand',
     'understanding',
     'unfortunately',
     'united',
     'united states',
     'university',
     'unix',
     'unless',
     'use',
     'used',
     'useful',
     'usenet',
     'user',
     'users',
     'uses',
     'using',
     'usr',
     'usually',
     'value',
     'values',
     'various',
     've',
     'version',
     'versions',
     'video',
     'view',
     'village',
     'villages',
     'visual',
     'vitamin',
     'volume',
     'want',
     'wanted',
     'wants',
     'war',
     'washington',
     'wasn',
     'water',
     'way',
     'ways',
     'week',
     'weeks',
     'went',
     'west',
     'western',
     'white',
     'widget',
     'widgets',
     'wife',
     'willing',
     'win',
     'window',
     'window manager',
     'windows',
     'wire',
     'wish',
     'woman',
     'women',
     'won',
     'wondering',
     'word',
     'words',
     'work',
     'worked',
     'working',
     'works',
     'world',
     'worth',
     'wouldn',
     'write',
     'writing',
     'written',
     'wrong',
     'wrote',
     'x11',
     'x11r5',
     'xlib',
     'xt',
     'xterm',
     'xv',
     'xview',
     'year',
     'years',
     'years ago',
     'yes',
     'york',
     'young']



**LDA 객체 생성 후 Count 피처 벡터화 객체로 LDA수행**


```python
lda = LatentDirichletAllocation(n_components=8, random_state=0)
lda.fit(feat_vect)
```




    LatentDirichletAllocation(n_components=8, random_state=0)



**각 토픽 모델링 주제별 단어들의 연관도 확인**  
lda객체의 components_ 속성은 주제별로 개별 단어들의 연관도 정규화 숫자가 들어있음

shape는 주제 개수 X 피처 단어 개수  

components_ 에 들어 있는 숫자값은 각 주제별로 단어가 나타난 횟수를 정규화 하여 나타냄.   

숫자가 클 수록 토픽에서 단어가 차지하는 비중이 높음  


```python
print(lda.components_.shape)
lda.components_
```

    (8, 1000)





    array([[3.60992018e+01, 1.35626798e+02, 2.15751867e+01, ...,
            3.02911688e+01, 8.66830093e+01, 6.79285199e+01],
           [1.25199920e-01, 1.44401815e+01, 1.25045596e-01, ...,
            1.81506995e+02, 1.25097844e-01, 9.39593286e+01],
           [3.34762663e+02, 1.25176265e-01, 1.46743299e+02, ...,
            1.25105772e-01, 3.63689741e+01, 1.25025218e-01],
           ...,
           [3.60204965e+01, 2.08640688e+01, 4.29606813e+00, ...,
            1.45056650e+01, 8.33854413e+00, 1.55690009e+01],
           [1.25128711e-01, 1.25247756e-01, 1.25005143e-01, ...,
            9.17278769e+01, 1.25177668e-01, 3.74575887e+01],
           [5.49258690e+01, 4.47009532e+00, 9.88524814e+00, ...,
            4.87048440e+01, 1.25034678e-01, 1.25074632e-01]])



**각 토픽별 중심 단어 확인**


```python
def display_topic_words(model, feature_names, no_top_words):
    for topic_index, topic in enumerate(model.components_):
        print('\nTopic #',topic_index)

        # components_ array에서 가장 값이 큰 순으로 정렬했을 때, 그 값의 array index를 반환. 
        topic_word_indexes = topic.argsort()[::-1]
        top_indexes=topic_word_indexes[:no_top_words]
        
        # top_indexes대상인 index별로 feature_names에 해당하는 word feature 추출 후 join으로 concat
        feature_concat = ' '.join([str(feature_names[i]) for i in top_indexes])
        #feature_concat = ' + '.join([str(feature_names[i])+'*'+str(round(topic[i],1)) for i in top_indexes])                
        print(feature_concat)

# CountVectorizer객체내의 전체 word들의 명칭을 get_features_names( )를 통해 추출
feature_names = count_vect.get_feature_names()

# Topic별 가장 연관도가 높은 word를 15개만 추출
display_topic_words(lda, feature_names, 15)

# 모토사이클, 야구, 그래픽스, 윈도우즈, 중동, 기독교, 전자공학, 의학 등 8개 주제를 추출. 
```


    Topic # 0
    year 10 game medical health team 12 20 disease cancer 1993 games years patients good
    
    Topic # 1
    don just like know people said think time ve didn right going say ll way
    
    Topic # 2
    image file jpeg program gif images output format files color entry 00 use bit 03
    
    Topic # 3
    like know don think use does just good time book read information people used post
    
    Topic # 4
    armenian israel armenians jews turkish people israeli jewish government war dos dos turkey arab armenia 000
    
    Topic # 5
    edu com available graphics ftp data pub motif mail widget software mit information version sun
    
    Topic # 6
    god people jesus church believe christ does christian say think christians bible faith sin life
    
    Topic # 7
    use dos thanks windows using window does display help like problem server need know run


**개별 문서별 토픽 분포 확인**

lda객체의 transform()을 수행하면 개별 문서별 토픽 분포를 반환함. 


```python
doc_topics = lda.transform(feat_vect)
print(doc_topics.shape)
print(doc_topics[:3])
```

    (7862, 8)
    [[0.01389701 0.01394362 0.01389104 0.48221844 0.01397882 0.01389205
      0.01393501 0.43424401]
     [0.27750436 0.18151826 0.0021208  0.53037189 0.00212129 0.00212102
      0.00212113 0.00212125]
     [0.00544459 0.22166575 0.00544539 0.00544528 0.00544039 0.00544168
      0.00544182 0.74567512]]


**개별 문서별 토픽 분포도를 출력**

20newsgroup으로 만들어진 문서명을 출력.

fetch_20newsgroups()으로 만들어진 데이터의 filename속성은 모든 문서의 문서명을 가지고 있음.

filename속성은 절대 디렉토리를 가지는 문서명을 가지고 있으므로 '\\'로 분할하여 맨 마지막 두번째 부터 파일명으로 가져옴


```python
def get_filename_list(newsdata):
    filename_list=[]

    for file in newsdata.filenames:
            #print(file)
            filename_temp = file.split('\\')[-2:]
            filename = '.'.join(filename_temp)
            filename_list.append(filename)
    
    return filename_list

filename_list = get_filename_list(news_df)
print("filename 개수:",len(filename_list), "filename list 10개만:",filename_list[:10])
```

    filename 개수: 7862 filename list 10개만: ['soc.religion.christian.20630', 'sci.med.59422', 'comp.graphics.38765', 'comp.graphics.38810', 'sci.med.59449', 'comp.graphics.38461', 'comp.windows.x.66959', 'rec.motorcycles.104487', 'sci.electronics.53875', 'sci.electronics.53617']



```python
news_df.filenames
```




    array(['C:\\Users\\q\\scikit_learn_data\\20news_home\\20news-bydate-train\\soc.religion.christian\\20630',
           'C:\\Users\\q\\scikit_learn_data\\20news_home\\20news-bydate-test\\sci.med\\59422',
           'C:\\Users\\q\\scikit_learn_data\\20news_home\\20news-bydate-test\\comp.graphics\\38765',
           ...,
           'C:\\Users\\q\\scikit_learn_data\\20news_home\\20news-bydate-train\\rec.sport.baseball\\102656',
           'C:\\Users\\q\\scikit_learn_data\\20news_home\\20news-bydate-train\\sci.electronics\\53606',
           'C:\\Users\\q\\scikit_learn_data\\20news_home\\20news-bydate-train\\talk.politics.mideast\\76505'],
          dtype='<U91')



**DataFrame으로 생성하여 문서별 토픽 분포도 확인**


```python
import pandas as pd 

topic_names = ['Topic #'+ str(i) for i in range(0, 8)]
doc_topic_df = pd.DataFrame(data=doc_topics, columns=topic_names, index=filename_list)
doc_topic_df.head(20)
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
      <th>Topic #0</th>
      <th>Topic #1</th>
      <th>Topic #2</th>
      <th>Topic #3</th>
      <th>Topic #4</th>
      <th>Topic #5</th>
      <th>Topic #6</th>
      <th>Topic #7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>soc.religion.christian.20630</th>
      <td>0.013897</td>
      <td>0.013944</td>
      <td>0.013891</td>
      <td>0.482218</td>
      <td>0.013979</td>
      <td>0.013892</td>
      <td>0.013935</td>
      <td>0.434244</td>
    </tr>
    <tr>
      <th>sci.med.59422</th>
      <td>0.277504</td>
      <td>0.181518</td>
      <td>0.002121</td>
      <td>0.530372</td>
      <td>0.002121</td>
      <td>0.002121</td>
      <td>0.002121</td>
      <td>0.002121</td>
    </tr>
    <tr>
      <th>comp.graphics.38765</th>
      <td>0.005445</td>
      <td>0.221666</td>
      <td>0.005445</td>
      <td>0.005445</td>
      <td>0.005440</td>
      <td>0.005442</td>
      <td>0.005442</td>
      <td>0.745675</td>
    </tr>
    <tr>
      <th>comp.graphics.38810</th>
      <td>0.005439</td>
      <td>0.005441</td>
      <td>0.005449</td>
      <td>0.578959</td>
      <td>0.005440</td>
      <td>0.388387</td>
      <td>0.005442</td>
      <td>0.005442</td>
    </tr>
    <tr>
      <th>sci.med.59449</th>
      <td>0.006584</td>
      <td>0.552000</td>
      <td>0.006587</td>
      <td>0.408485</td>
      <td>0.006585</td>
      <td>0.006585</td>
      <td>0.006588</td>
      <td>0.006585</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>sci.electronics.53785</th>
      <td>0.015652</td>
      <td>0.015648</td>
      <td>0.015635</td>
      <td>0.429298</td>
      <td>0.476854</td>
      <td>0.015641</td>
      <td>0.015643</td>
      <td>0.015629</td>
    </tr>
    <tr>
      <th>talk.politics.mideast.77381</th>
      <td>0.003574</td>
      <td>0.003585</td>
      <td>0.003573</td>
      <td>0.128361</td>
      <td>0.670401</td>
      <td>0.003573</td>
      <td>0.183359</td>
      <td>0.003573</td>
    </tr>
    <tr>
      <th>rec.motorcycles.103140</th>
      <td>0.004823</td>
      <td>0.966293</td>
      <td>0.004810</td>
      <td>0.004815</td>
      <td>0.004818</td>
      <td>0.004813</td>
      <td>0.004813</td>
      <td>0.004814</td>
    </tr>
    <tr>
      <th>rec.motorcycles.105381</th>
      <td>0.025030</td>
      <td>0.025057</td>
      <td>0.025035</td>
      <td>0.824793</td>
      <td>0.025018</td>
      <td>0.025012</td>
      <td>0.025023</td>
      <td>0.025032</td>
    </tr>
    <tr>
      <th>comp.windows.x.67546</th>
      <td>0.001182</td>
      <td>0.001181</td>
      <td>0.001180</td>
      <td>0.001181</td>
      <td>0.001181</td>
      <td>0.085647</td>
      <td>0.043141</td>
      <td>0.865307</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 8 columns</p>
</div>




```python

```
