[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://bit.ly/open-data-01-apt-price-input)

# 전국 신규 민간 아파트 분양가격 동향

2013년부터 최근까지 부동산 가격 변동 추세가 아파트 분양가에도 반영될까요? 공공데이터 포털에 있는 데이터를 Pandas 의 melt, concat, pivot, transpose 와 같은 reshape 기능을 활용해 분석해 봅니다. 그리고 groupby, pivot_table, info, describe, value_counts 등을 통한 데이터 요약과 분석을 해봅니다. 이를 통해 전혀 다른 형태의 두 데이터를 가져와 정제하고 병합하는 과정을 다루는 방법을 알게 됩니다. 전처리 한 결과에 대해 수치형, 범주형 데이터의 차이를 이해하고 다양한 그래프로 시각화를 할 수 있게 됩니다.


## 다루는 내용
* 공공데이터를 활용해 전혀 다른 두 개의 데이터를 가져와서 전처리 하고 병합하기
* 수치형 데이터와 범주형 데이터를 바라보는 시각을 기르기
* 데이터의 형식에 따른 다양한 시각화 방법 이해하기

## 실습
* 공공데이터 다운로드 후 주피터 노트북으로 로드하기
* 판다스를 통해 데이터를 요약하고 분석하기
* 데이터 전처리와 병합하기
* 수치형 데이터와 범주형 데이터 다루기
* 막대그래프(bar plot), 선그래프(line plot), 산포도(scatter plot), 상관관계(lm plot), 히트맵, 상자수염그림, swarm plot, 도수분포표, 히스토그램(distplot) 실습하기

## 데이터셋
* 다운로드 위치 : https://www.data.go.kr/dataset/3035522/fileData.do

### 전국 평균 분양가격(2013년 9월부터 2015년 8월까지)
* 전국 공동주택의 3.3제곱미터당 평균분양가격 데이터를 제공

###  주택도시보증공사_전국 평균 분양가격(2019년 12월)
* 전국 공동주택의 연도별, 월별, 전용면적별 제곱미터당 평균분양가격 데이터를 제공
* 지역별 평균값은 단순 산술평균값이 아닌 가중평균값임


```python
%ls data

```

     Volume in drive C has no label.
     Volume Serial Number is B072-72FF
    
     Directory of C:\Users\USER\Downloads\Jupyter\open-data-analysis-basic-master\data
    
    09/18/2021  11:29 PM    <DIR>          .
    09/18/2021  11:29 PM    <DIR>          ..
    09/18/2021  05:30 AM             2,163 전국 평균 분양가격(2013년 9월부터 2015년 8월까지).csv
    09/18/2021  05:55 AM           162,510 주택도시보증공사_전국 평균 분양가격(2019년 12월).csv
                   2 File(s)        164,673 bytes
                   2 Dir(s)  157,241,262,080 bytes free
    


```python
# 파이썬에서 쓸 수 있는 엑셀과도 유사한 판다스 라이브러리를 불러옵니다.
import pandas as pd
```

## 데이터 로드
### 최근 파일 로드
공공데이터 포털에서 "주택도시보증공사_전국 평균 분양가격"파일을 다운로드 받아 불러옵니다.
이 때, 인코딩을 설정을 해주어야 한글이 깨지지 않습니다.
보통 엑셀로 저장된 한글의 인코딩은 cp949 혹은 euc-kr로 되어 있습니다.
df_last 라는 변수에 최근 분양가 파일을 다운로드 받아 로드합니다.

* 한글인코딩 : [‘설믜를 설믜라 못 부르는’ 김설믜씨 “제 이름을 지켜주세요” : 사회일반 : 사회 : 뉴스 : 한겨레](http://www.hani.co.kr/arti/society/society_general/864914.html)

데이터를 로드한 뒤 shape를 통해 행과 열의 갯수를 출력합니다.


```python
# 최근 분양가 파일을 로드해서 df_last 라는 변수에 담습니다.
# 파일로드시 OSError가 발생한다면, engine="python"을 추가해 보세요.
# 윈도우에서 파일탐색기의 경로를 복사해서 붙여넣기 했는데도 파일을 불러올 수 없다면
# 아마도 경로에 있는 ₩ 역슬래시 표시를 못 읽어왔을 가능성이 큽니다. 
# r"경로명" 으로 적어주세요.
# r"경로명"으로 적게 되면 경로를 문자 그대로(raw) 읽으라는 의미입니다.

df_last = pd.read_csv("data/주택도시보증공사_전국 평균 분양가격(2019년 12월).csv", encoding="cp949")
df_last.shape
```




    (4335, 5)




```python
# head 로 파일을 미리보기 합니다.
# 메소드 뒤에 ?를 하면 자기호출 이라는 기능을 통해 메소드의 docstring을 출력합니다.
# 메소드의 ()괄호 안에서 Shift + Tab키를 눌러도 같은 문서를 열어볼 수 있습니다.
# Shift + Tab + Tab 을 하게 되면 팝업창을 키울 수 있습니다.

df_last.head()
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
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>전체</td>
      <td>2015</td>
      <td>10</td>
      <td>5841</td>
    </tr>
    <tr>
      <th>1</th>
      <td>서울</td>
      <td>전용면적 60㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5652</td>
    </tr>
    <tr>
      <th>2</th>
      <td>서울</td>
      <td>전용면적 60㎡초과 85㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5882</td>
    </tr>
    <tr>
      <th>3</th>
      <td>서울</td>
      <td>전용면적 85㎡초과 102㎡이하</td>
      <td>2015</td>
      <td>10</td>
      <td>5721</td>
    </tr>
    <tr>
      <th>4</th>
      <td>서울</td>
      <td>전용면적 102㎡초과</td>
      <td>2015</td>
      <td>10</td>
      <td>5879</td>
    </tr>
  </tbody>
</table>
</div>




```python
# tail 로도 미리보기를 합니다.
df_last.tail()
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
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4330</th>
      <td>제주</td>
      <td>전체</td>
      <td>2019</td>
      <td>12</td>
      <td>3882</td>
    </tr>
    <tr>
      <th>4331</th>
      <td>제주</td>
      <td>전용면적 60㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4332</th>
      <td>제주</td>
      <td>전용면적 60㎡초과 85㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>3898</td>
    </tr>
    <tr>
      <th>4333</th>
      <td>제주</td>
      <td>전용면적 85㎡초과 102㎡이하</td>
      <td>2019</td>
      <td>12</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4334</th>
      <td>제주</td>
      <td>전용면적 102㎡초과</td>
      <td>2019</td>
      <td>12</td>
      <td>3601</td>
    </tr>
  </tbody>
</table>
</div>



### 2015년 부터 최근까지의 데이터 로드
전국 평균 분양가격(2013년 9월부터 2015년 8월까지) 파일을 불러옵니다.
df_first 라는 변수에 담고 shape로 행과 열의 갯수를 출력합니다.


```python
# 해당되는 폴더 혹은 경로의 파일 목록을 출력해 줍니다.
%ls data
```

     Volume in drive C has no label.
     Volume Serial Number is B072-72FF
    
     Directory of C:\Users\USER\Downloads\Jupyter\open-data-analysis-basic-master\data
    
    09/18/2021  11:29 PM    <DIR>          .
    09/18/2021  11:29 PM    <DIR>          ..
    09/18/2021  05:30 AM             2,163 전국 평균 분양가격(2013년 9월부터 2015년 8월까지).csv
    09/18/2021  05:55 AM           162,510 주택도시보증공사_전국 평균 분양가격(2019년 12월).csv
                   2 File(s)        164,673 bytes
                   2 Dir(s)  157,241,253,888 bytes free
    


```python
# df_first 에 담고 shape로 행과 열의 수를 출력해 봅니다.
df_first = pd.read_csv("data/전국 평균 분양가격(2013년 9월부터 2015년 8월까지).csv", encoding="cp949")
df_first.shape
```




    (17, 22)




```python
# df_first 변수에 담긴 데이터프레임을 head로 미리보기 합니다.
df_first.head()
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
      <th>지역</th>
      <th>2013년12월</th>
      <th>2014년1월</th>
      <th>2014년2월</th>
      <th>2014년3월</th>
      <th>2014년4월</th>
      <th>2014년5월</th>
      <th>2014년6월</th>
      <th>2014년7월</th>
      <th>2014년8월</th>
      <th>...</th>
      <th>2014년11월</th>
      <th>2014년12월</th>
      <th>2015년1월</th>
      <th>2015년2월</th>
      <th>2015년3월</th>
      <th>2015년4월</th>
      <th>2015년5월</th>
      <th>2015년6월</th>
      <th>2015년7월</th>
      <th>2015년8월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>18189</td>
      <td>17925</td>
      <td>17925</td>
      <td>18016</td>
      <td>18098</td>
      <td>19446</td>
      <td>18867</td>
      <td>18742</td>
      <td>19274</td>
      <td>...</td>
      <td>20242</td>
      <td>20269</td>
      <td>20670</td>
      <td>20670</td>
      <td>19415</td>
      <td>18842</td>
      <td>18367</td>
      <td>18374</td>
      <td>18152</td>
      <td>18443</td>
    </tr>
    <tr>
      <th>1</th>
      <td>부산</td>
      <td>8111</td>
      <td>8111</td>
      <td>9078</td>
      <td>8965</td>
      <td>9402</td>
      <td>9501</td>
      <td>9453</td>
      <td>9457</td>
      <td>9411</td>
      <td>...</td>
      <td>9208</td>
      <td>9208</td>
      <td>9204</td>
      <td>9235</td>
      <td>9279</td>
      <td>9327</td>
      <td>9345</td>
      <td>9515</td>
      <td>9559</td>
      <td>9581</td>
    </tr>
    <tr>
      <th>2</th>
      <td>대구</td>
      <td>8080</td>
      <td>8080</td>
      <td>8077</td>
      <td>8101</td>
      <td>8267</td>
      <td>8274</td>
      <td>8360</td>
      <td>8360</td>
      <td>8370</td>
      <td>...</td>
      <td>8439</td>
      <td>8253</td>
      <td>8327</td>
      <td>8416</td>
      <td>8441</td>
      <td>8446</td>
      <td>8568</td>
      <td>8542</td>
      <td>8542</td>
      <td>8795</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인천</td>
      <td>10204</td>
      <td>10204</td>
      <td>10408</td>
      <td>10408</td>
      <td>10000</td>
      <td>9844</td>
      <td>10058</td>
      <td>9974</td>
      <td>9973</td>
      <td>...</td>
      <td>10020</td>
      <td>10020</td>
      <td>10017</td>
      <td>9876</td>
      <td>9876</td>
      <td>9938</td>
      <td>10551</td>
      <td>10443</td>
      <td>10443</td>
      <td>10449</td>
    </tr>
    <tr>
      <th>4</th>
      <td>광주</td>
      <td>6098</td>
      <td>7326</td>
      <td>7611</td>
      <td>7346</td>
      <td>7346</td>
      <td>7523</td>
      <td>7659</td>
      <td>7612</td>
      <td>7622</td>
      <td>...</td>
      <td>7752</td>
      <td>7748</td>
      <td>7752</td>
      <td>7756</td>
      <td>7861</td>
      <td>7914</td>
      <td>7877</td>
      <td>7881</td>
      <td>8089</td>
      <td>8231</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>




```python
# df_first 변수에 담긴 데이터프레임을 tail로 미리보기 합니다.
df_first.tail()
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
      <th>지역</th>
      <th>2013년12월</th>
      <th>2014년1월</th>
      <th>2014년2월</th>
      <th>2014년3월</th>
      <th>2014년4월</th>
      <th>2014년5월</th>
      <th>2014년6월</th>
      <th>2014년7월</th>
      <th>2014년8월</th>
      <th>...</th>
      <th>2014년11월</th>
      <th>2014년12월</th>
      <th>2015년1월</th>
      <th>2015년2월</th>
      <th>2015년3월</th>
      <th>2015년4월</th>
      <th>2015년5월</th>
      <th>2015년6월</th>
      <th>2015년7월</th>
      <th>2015년8월</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>전북</td>
      <td>6282</td>
      <td>6281</td>
      <td>5946</td>
      <td>5966</td>
      <td>6277</td>
      <td>6306</td>
      <td>6351</td>
      <td>6319</td>
      <td>6436</td>
      <td>...</td>
      <td>6583</td>
      <td>6583</td>
      <td>6583</td>
      <td>6583</td>
      <td>6542</td>
      <td>6551</td>
      <td>6556</td>
      <td>6601</td>
      <td>6750</td>
      <td>6580</td>
    </tr>
    <tr>
      <th>13</th>
      <td>전남</td>
      <td>5678</td>
      <td>5678</td>
      <td>5678</td>
      <td>5696</td>
      <td>5736</td>
      <td>5656</td>
      <td>5609</td>
      <td>5780</td>
      <td>5685</td>
      <td>...</td>
      <td>5768</td>
      <td>5784</td>
      <td>5784</td>
      <td>5833</td>
      <td>5825</td>
      <td>5940</td>
      <td>6050</td>
      <td>6243</td>
      <td>6286</td>
      <td>6289</td>
    </tr>
    <tr>
      <th>14</th>
      <td>경북</td>
      <td>6168</td>
      <td>6168</td>
      <td>6234</td>
      <td>6317</td>
      <td>6412</td>
      <td>6409</td>
      <td>6554</td>
      <td>6556</td>
      <td>6563</td>
      <td>...</td>
      <td>6881</td>
      <td>6989</td>
      <td>6992</td>
      <td>6953</td>
      <td>6997</td>
      <td>7006</td>
      <td>6966</td>
      <td>6887</td>
      <td>7035</td>
      <td>7037</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경남</td>
      <td>6473</td>
      <td>6485</td>
      <td>6502</td>
      <td>6610</td>
      <td>6599</td>
      <td>6610</td>
      <td>6615</td>
      <td>6613</td>
      <td>6606</td>
      <td>...</td>
      <td>7125</td>
      <td>7332</td>
      <td>7592</td>
      <td>7588</td>
      <td>7668</td>
      <td>7683</td>
      <td>7717</td>
      <td>7715</td>
      <td>7723</td>
      <td>7665</td>
    </tr>
    <tr>
      <th>16</th>
      <td>제주</td>
      <td>7674</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7900</td>
      <td>7914</td>
      <td>7914</td>
      <td>7914</td>
      <td>...</td>
      <td>7724</td>
      <td>7739</td>
      <td>7739</td>
      <td>7739</td>
      <td>7826</td>
      <td>7285</td>
      <td>7285</td>
      <td>7343</td>
      <td>7343</td>
      <td>7343</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



### 데이터 요약하기


```python
# info 로 요약합니다.
df_last.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 5 columns):
     #   Column   Non-Null Count  Dtype 
    ---  ------   --------------  ----- 
     0   지역명      4335 non-null   object
     1   규모구분     4335 non-null   object
     2   연도       4335 non-null   int64 
     3   월        4335 non-null   int64 
     4   분양가격(㎡)  4058 non-null   object
    dtypes: int64(2), object(3)
    memory usage: 169.5+ KB
    

### 결측치 보기

isnull 혹은 isna 를 통해 데이터가 비어있는지를 확인할 수 있습니다.
결측치는 True로 표시되는데, True == 1 이기 때문에 이 값을 다 더해주면 결측치의 수가 됩니다.


```python
True == 1
```




    True




```python
False == 0 
```




    True




```python
# isnull 을 통해 결측치를 봅니다.
df_last.isnull()
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
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4330</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4331</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4332</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4333</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4334</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>4335 rows × 5 columns</p>
</div>




```python
# isnull 을 통해 결측치를 구합니다.
df_last.isnull().sum()
```




    지역명          0
    규모구분         0
    연도           0
    월            0
    분양가격(㎡)    277
    dtype: int64




```python
# isna 를 통해 결측치를 구합니다.
df_last.isna()
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
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4330</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4331</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4332</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4333</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4334</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>4335 rows × 5 columns</p>
</div>



### 데이터 타입 변경
분양가격이 object(문자) 타입으로 되어 있습니다. 문자열 타입을 계산할 수 없기 때문에 수치 데이터로 변경해 줍니다. 결측치가 섞여 있을 때 변환이 제대로 되지 않습니다. 그래서 pd.to_numeric 을 통해 데이터의 타입을 변경합니다.


```python
import numpy as np  #numpy는 공학용계산기 같은것, 왼쪽은 numpy를 불러오는 방법
                #판다스에 내장되었기 때문에  pd.np.nan 처럼 불러와서 사용할 수 있었음.

np.nan #결측치의미
```




    nan




```python
df_last["분양가격"] = pd.to_numeric(df_last["분양가격(㎡)"], errors = 'coerce')
df_last["분양가격"]
```




    0       5841.0
    1       5652.0
    2       5882.0
    3       5721.0
    4       5879.0
             ...  
    4330    3882.0
    4331       NaN
    4332    3898.0
    4333       NaN
    4334    3601.0
    Name: 분양가격, Length: 4335, dtype: float64



### 평당분양가격 구하기
공공데이터포털에 올라와 있는 2013년부터의 데이터는 평당분양가격 기준으로 되어 있습니다.
분양가격을 평당기준으로 보기위해 3.3을 곱해서 "평당분양가격" 컬럼을 만들어 추가해 줍니다.


```python
df_last["평당분양가격"] = df_last["분양가격"] * 3.3
df_last.head(1)
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
      <th>지역명</th>
      <th>규모구분</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격(㎡)</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>전체</td>
      <td>2015</td>
      <td>10</td>
      <td>5841</td>
      <td>5841.0</td>
      <td>19275.3</td>
    </tr>
  </tbody>
</table>
</div>



### 분양가격 요약하기


```python
# info를 통해 분양가격을 봅니다.
df_last.info() #datafram 정보보기
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 7 columns):
     #   Column   Non-Null Count  Dtype  
    ---  ------   --------------  -----  
     0   지역명      4335 non-null   object 
     1   규모구분     4335 non-null   object 
     2   연도       4335 non-null   int64  
     3   월        4335 non-null   int64  
     4   분양가격(㎡)  4058 non-null   object 
     5   분양가격     3957 non-null   float64
     6   평당분양가격   3957 non-null   float64
    dtypes: float64(2), int64(2), object(3)
    memory usage: 237.2+ KB
    


```python
# 변경 전 컬럼인 분양가격(㎡) 컬럼을 요약합니다.

df_last["분양가격(㎡)"].describe()  
#결측치도 데이터가 있는것으로 간주해서 count가 밑에랑 다름
#freq: 가장빈번하게 등장하는 문자가 몇번등장하는가.. 2221이라는 object(문자형데이터)가 17번나옴
```




    count     4058
    unique    1753
    top       2221
    freq        17
    Name: 분양가격(㎡), dtype: object




```python
# 수치데이터로 변경된 분양가격 컬럼을 요약합니다.

df_last["분양가격"].describe()

#max 최대값이 큰것으로 보아 평균값이 크다.
```




    count     3957.000000
    mean      3238.128633
    std       1264.309933
    min       1868.000000
    25%       2441.000000
    50%       2874.000000
    75%       3561.000000
    max      12728.000000
    Name: 분양가격, dtype: float64



### 규모구분을 전용면적 컬럼으로 변경
규모구분 컬럼은 전용면적에 대한 내용이 있습니다. 전용면적이라는 문구가 공통적으로 들어가고 규모구분보다는 전용면적이 좀 더 직관적이기 때문에 전용면적이라는 컬럼을 새로 만들어주고 기존 규모구분의 값에서 전용면적, 초과, 이하 등의 문구를 빼고 간결하게 만들어 봅니다.

이 때 str 의 replace 기능을 사용해서 예를들면 "전용면적 60㎡초과 85㎡이하"라면 "60㎡~85㎡" 로 변경해 줍니다.

* pandas 의 string-handling 기능을 좀 더 보고 싶다면 :
https://pandas.pydata.org/pandas-docs/stable/reference/series.html#string-handling


```python
# 규모구분의 unique 값 보기

df_last["규모구분"].unique()
```




    array(['전체', '전용면적 60㎡이하', '전용면적 60㎡초과 85㎡이하', '전용면적 85㎡초과 102㎡이하',
           '전용면적 102㎡초과'], dtype=object)




```python
df_last["규모구분"].replace("전용면적", "")
#전용면적이라는 단어를 ""빈칸으로 바꾸려고 했는데 안됨
```




    0                      전체
    1              전용면적 60㎡이하
    2        전용면적 60㎡초과 85㎡이하
    3       전용면적 85㎡초과 102㎡이하
    4             전용면적 102㎡초과
                  ...        
    4330                   전체
    4331           전용면적 60㎡이하
    4332     전용면적 60㎡초과 85㎡이하
    4333    전용면적 85㎡초과 102㎡이하
    4334          전용면적 102㎡초과
    Name: 규모구분, Length: 4335, dtype: object




```python
# 규모구분을 전용면적으로 변경하기

df_last["전용면적"] = df_last["규모구분"].str.replace("전용면적", "")
df_last["전용면적"] = df_last["전용면적"].str.replace("초과", "~")
df_last["전용면적"] = df_last["전용면적"].str.replace("이하", "")
df_last["전용면적"].str.replace(" ", "") #~뒤의 빈공간제거
df_last["전용면적"].str.replace(" ", "").str.strip() #앞뒤의 공백제거
df_last["전용면적"]
```




    0               전체
    1              60㎡
    2         60㎡~ 85㎡
    3        85㎡~ 102㎡
    4            102㎡~
               ...    
    4330            전체
    4331           60㎡
    4332      60㎡~ 85㎡
    4333     85㎡~ 102㎡
    4334         102㎡~
    Name: 전용면적, Length: 4335, dtype: object




```python

```

### 필요없는 컬럼 제거하기
drop을 통해 전처리 해준 컬럼을 제거합니다. pandas의 데이터프레임과 관련된 메소드에는 axis 옵션이 필요할 때가 있는데 행과 열중 어떤 기준으로 처리를 할 것인지를 의미합니다. 보통 기본적으로 0으로 되어 있고 행을 기준으로 처리함을 의미합니다. 메모리 사용량이 줄어들었는지 확인합니다.


```python
df_last.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 8 columns):
     #   Column   Non-Null Count  Dtype  
    ---  ------   --------------  -----  
     0   지역명      4335 non-null   object 
     1   규모구분     4335 non-null   object 
     2   연도       4335 non-null   int64  
     3   월        4335 non-null   int64  
     4   분양가격(㎡)  4058 non-null   object 
     5   분양가격     3957 non-null   float64
     6   평당분양가격   3957 non-null   float64
     7   전용면적     4335 non-null   object 
    dtypes: float64(2), int64(2), object(4)
    memory usage: 271.1+ KB
    


```python
# drop 사용시 axis에 유의 합니다.
# axis 0:행, 1:열

df_last = df_last.drop(["규모구분","분양가격(㎡)"], axis = 1)
#사용하지 않는 컬럼 규모구분이랑 분양가격 지우기
#df_last.drop?이걸로 보고 axis넣기 알게됨
#열을 기준으로 지울거니까 axis = 1로 설정. (단, 행이면 axis =0)
```


```python
# 제거가 잘 되었는지 확인 합니다.

df_last.head(1)
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
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
      <th>전용면적</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>서울</td>
      <td>2015</td>
      <td>10</td>
      <td>5841.0</td>
      <td>19275.3</td>
      <td>전체</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 컬럼 제거를 통해 메모리 사용량이 줄어들었는지 확인합니다.

df_last.info()  #제거전은 데이터가 271이었지만 203으로 감소
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4335 entries, 0 to 4334
    Data columns (total 6 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   지역명     4335 non-null   object 
     1   연도      4335 non-null   int64  
     2   월       4335 non-null   int64  
     3   분양가격    3957 non-null   float64
     4   평당분양가격  3957 non-null   float64
     5   전용면적    4335 non-null   object 
    dtypes: float64(2), int64(2), object(2)
    memory usage: 203.3+ KB
    

## groupby 로 데이터 집계하기
groupby 를 통해 데이터를 그룹화해서 연산을 해봅니다.


```python
# 지역명으로 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.
# df.groupby(["인덱스로 사용할 컬럼명"])["계산할 컬럼 값"].연산()

df_last.groupby(["지역명"])["평당분양가격"].mean()
```




    지역명
    강원     7890.750000
    경기    13356.895200
    경남     9268.778138
    경북     8376.536515
    광주     9951.535821
    대구    11980.895455
    대전    10253.333333
    부산    12087.121200
    서울    23599.976400
    세종     9796.516456
    울산    10014.902013
    인천    11915.320732
    전남     7565.316532
    전북     7724.235484
    제주    11241.276712
    충남     8233.651883
    충북     7634.655600
    Name: 평당분양가격, dtype: float64




```python
# 전용면적으로 분양가격의 평균을 구합니다.

df_last.groupby(["전용면적"])["평당분양가격"].mean()
```




    전용면적
     102㎡~        11517.705634
     60㎡          10375.137421
     60㎡~ 85㎡     10271.040071
     85㎡~ 102㎡    11097.599573
    전체            10276.086207
    Name: 평당분양가격, dtype: float64




```python
# 지역명, 전용면적으로 평당분양가격의 평균을 구합니다.

df_last.groupby(["전용면적", "지역명"])["평당분양가격"].mean().unstack().round()

#unstack했더니 뒤의 컬럼 "전용면적"이 컬럼값이 된다. 
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
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>8311.0</td>
      <td>14772.0</td>
      <td>10358.0</td>
      <td>9157.0</td>
      <td>11042.0</td>
      <td>13087.0</td>
      <td>14877.0</td>
      <td>13208.0</td>
      <td>23446.0</td>
      <td>10107.0</td>
      <td>9974.0</td>
      <td>14362.0</td>
      <td>8168.0</td>
      <td>8194.0</td>
      <td>10523.0</td>
      <td>8689.0</td>
      <td>8195.0</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>7567.0</td>
      <td>13252.0</td>
      <td>8689.0</td>
      <td>7883.0</td>
      <td>9431.0</td>
      <td>11992.0</td>
      <td>9176.0</td>
      <td>11354.0</td>
      <td>23213.0</td>
      <td>9324.0</td>
      <td>9202.0</td>
      <td>11241.0</td>
      <td>7210.0</td>
      <td>7610.0</td>
      <td>14022.0</td>
      <td>7911.0</td>
      <td>7103.0</td>
    </tr>
    <tr>
      <th>60㎡~ 85㎡</th>
      <td>7486.0</td>
      <td>12524.0</td>
      <td>8619.0</td>
      <td>8061.0</td>
      <td>9911.0</td>
      <td>11779.0</td>
      <td>9711.0</td>
      <td>11865.0</td>
      <td>22787.0</td>
      <td>9775.0</td>
      <td>10503.0</td>
      <td>11384.0</td>
      <td>7269.0</td>
      <td>7271.0</td>
      <td>10621.0</td>
      <td>7819.0</td>
      <td>7264.0</td>
    </tr>
    <tr>
      <th>85㎡~ 102㎡</th>
      <td>8750.0</td>
      <td>13678.0</td>
      <td>10018.0</td>
      <td>8774.0</td>
      <td>9296.0</td>
      <td>11141.0</td>
      <td>9037.0</td>
      <td>12073.0</td>
      <td>25944.0</td>
      <td>9848.0</td>
      <td>8861.0</td>
      <td>11528.0</td>
      <td>7909.0</td>
      <td>8276.0</td>
      <td>10709.0</td>
      <td>9120.0</td>
      <td>8391.0</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>7478.0</td>
      <td>12560.0</td>
      <td>8659.0</td>
      <td>8079.0</td>
      <td>9904.0</td>
      <td>11771.0</td>
      <td>9786.0</td>
      <td>11936.0</td>
      <td>22610.0</td>
      <td>9805.0</td>
      <td>10493.0</td>
      <td>11257.0</td>
      <td>7284.0</td>
      <td>7293.0</td>
      <td>10785.0</td>
      <td>7815.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_last.groupby(["지역명", "전용면적"])["평당분양가격"].mean().unstack().round()
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
      <th>전용면적</th>
      <th>102㎡~</th>
      <th>60㎡</th>
      <th>60㎡~ 85㎡</th>
      <th>85㎡~ 102㎡</th>
      <th>전체</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>8311.0</td>
      <td>7567.0</td>
      <td>7486.0</td>
      <td>8750.0</td>
      <td>7478.0</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>14772.0</td>
      <td>13252.0</td>
      <td>12524.0</td>
      <td>13678.0</td>
      <td>12560.0</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>10358.0</td>
      <td>8689.0</td>
      <td>8619.0</td>
      <td>10018.0</td>
      <td>8659.0</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>9157.0</td>
      <td>7883.0</td>
      <td>8061.0</td>
      <td>8774.0</td>
      <td>8079.0</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>11042.0</td>
      <td>9431.0</td>
      <td>9911.0</td>
      <td>9296.0</td>
      <td>9904.0</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>13087.0</td>
      <td>11992.0</td>
      <td>11779.0</td>
      <td>11141.0</td>
      <td>11771.0</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>14877.0</td>
      <td>9176.0</td>
      <td>9711.0</td>
      <td>9037.0</td>
      <td>9786.0</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>13208.0</td>
      <td>11354.0</td>
      <td>11865.0</td>
      <td>12073.0</td>
      <td>11936.0</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>23446.0</td>
      <td>23213.0</td>
      <td>22787.0</td>
      <td>25944.0</td>
      <td>22610.0</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>10107.0</td>
      <td>9324.0</td>
      <td>9775.0</td>
      <td>9848.0</td>
      <td>9805.0</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>9974.0</td>
      <td>9202.0</td>
      <td>10503.0</td>
      <td>8861.0</td>
      <td>10493.0</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>14362.0</td>
      <td>11241.0</td>
      <td>11384.0</td>
      <td>11528.0</td>
      <td>11257.0</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>8168.0</td>
      <td>7210.0</td>
      <td>7269.0</td>
      <td>7909.0</td>
      <td>7284.0</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>8194.0</td>
      <td>7610.0</td>
      <td>7271.0</td>
      <td>8276.0</td>
      <td>7293.0</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>10523.0</td>
      <td>14022.0</td>
      <td>10621.0</td>
      <td>10709.0</td>
      <td>10785.0</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>8689.0</td>
      <td>7911.0</td>
      <td>7819.0</td>
      <td>9120.0</td>
      <td>7815.0</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>8195.0</td>
      <td>7103.0</td>
      <td>7264.0</td>
      <td>8391.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_last.groupby(["지역명", "전용면적"])["평당분양가격"].mean().unstack().round().T
#T를 붙여주면 지역명, 전용면적의 순서를 바꿔준다.
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
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>8311.0</td>
      <td>14772.0</td>
      <td>10358.0</td>
      <td>9157.0</td>
      <td>11042.0</td>
      <td>13087.0</td>
      <td>14877.0</td>
      <td>13208.0</td>
      <td>23446.0</td>
      <td>10107.0</td>
      <td>9974.0</td>
      <td>14362.0</td>
      <td>8168.0</td>
      <td>8194.0</td>
      <td>10523.0</td>
      <td>8689.0</td>
      <td>8195.0</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>7567.0</td>
      <td>13252.0</td>
      <td>8689.0</td>
      <td>7883.0</td>
      <td>9431.0</td>
      <td>11992.0</td>
      <td>9176.0</td>
      <td>11354.0</td>
      <td>23213.0</td>
      <td>9324.0</td>
      <td>9202.0</td>
      <td>11241.0</td>
      <td>7210.0</td>
      <td>7610.0</td>
      <td>14022.0</td>
      <td>7911.0</td>
      <td>7103.0</td>
    </tr>
    <tr>
      <th>60㎡~ 85㎡</th>
      <td>7486.0</td>
      <td>12524.0</td>
      <td>8619.0</td>
      <td>8061.0</td>
      <td>9911.0</td>
      <td>11779.0</td>
      <td>9711.0</td>
      <td>11865.0</td>
      <td>22787.0</td>
      <td>9775.0</td>
      <td>10503.0</td>
      <td>11384.0</td>
      <td>7269.0</td>
      <td>7271.0</td>
      <td>10621.0</td>
      <td>7819.0</td>
      <td>7264.0</td>
    </tr>
    <tr>
      <th>85㎡~ 102㎡</th>
      <td>8750.0</td>
      <td>13678.0</td>
      <td>10018.0</td>
      <td>8774.0</td>
      <td>9296.0</td>
      <td>11141.0</td>
      <td>9037.0</td>
      <td>12073.0</td>
      <td>25944.0</td>
      <td>9848.0</td>
      <td>8861.0</td>
      <td>11528.0</td>
      <td>7909.0</td>
      <td>8276.0</td>
      <td>10709.0</td>
      <td>9120.0</td>
      <td>8391.0</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>7478.0</td>
      <td>12560.0</td>
      <td>8659.0</td>
      <td>8079.0</td>
      <td>9904.0</td>
      <td>11771.0</td>
      <td>9786.0</td>
      <td>11936.0</td>
      <td>22610.0</td>
      <td>9805.0</td>
      <td>10493.0</td>
      <td>11257.0</td>
      <td>7284.0</td>
      <td>7293.0</td>
      <td>10785.0</td>
      <td>7815.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 연도, 지역명으로 평당분양가격의 평균을 구합니다.
g = df_last.groupby(["연도", "지역명"])["평당분양가격"].mean()
g.unstack().T
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
      <th>연도</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>2018</th>
      <th>2019</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>7188.060</td>
      <td>7162.903846</td>
      <td>7273.560000</td>
      <td>8219.255000</td>
      <td>8934.475000</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>11060.940</td>
      <td>11684.970000</td>
      <td>12304.980000</td>
      <td>14258.420000</td>
      <td>15665.540000</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>8459.220</td>
      <td>8496.730000</td>
      <td>8786.760000</td>
      <td>9327.670000</td>
      <td>10697.615789</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>7464.160</td>
      <td>7753.405000</td>
      <td>8280.800000</td>
      <td>8680.776923</td>
      <td>9050.250000</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>7916.700</td>
      <td>9190.683333</td>
      <td>9613.977551</td>
      <td>9526.953333</td>
      <td>12111.675000</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>9018.900</td>
      <td>10282.030000</td>
      <td>12206.700000</td>
      <td>12139.252632</td>
      <td>14081.650000</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>8190.600</td>
      <td>8910.733333</td>
      <td>9957.158491</td>
      <td>10234.106667</td>
      <td>12619.200000</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>10377.400</td>
      <td>10743.535000</td>
      <td>11560.680000</td>
      <td>12889.965000</td>
      <td>13537.865000</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>20315.680</td>
      <td>21753.435000</td>
      <td>21831.060000</td>
      <td>23202.245000</td>
      <td>28286.830000</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>8765.020</td>
      <td>8857.805000</td>
      <td>9132.505556</td>
      <td>10340.463158</td>
      <td>11299.394118</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>9367.600</td>
      <td>9582.574138</td>
      <td>10666.935714</td>
      <td>10241.400000</td>
      <td>10216.250000</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>10976.020</td>
      <td>11099.055000</td>
      <td>11640.600000</td>
      <td>11881.532143</td>
      <td>13249.775000</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>6798.880</td>
      <td>6936.600000</td>
      <td>7372.920000</td>
      <td>7929.845000</td>
      <td>8219.275862</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>7110.400</td>
      <td>6906.625000</td>
      <td>7398.973585</td>
      <td>8174.595000</td>
      <td>8532.260000</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>7951.075</td>
      <td>9567.480000</td>
      <td>12566.730000</td>
      <td>11935.968000</td>
      <td>11828.469231</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>7689.880</td>
      <td>7958.225000</td>
      <td>8198.422222</td>
      <td>8201.820000</td>
      <td>8748.840000</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>6828.800</td>
      <td>7133.335000</td>
      <td>7473.120000</td>
      <td>8149.295000</td>
      <td>7970.875000</td>
    </tr>
  </tbody>
</table>
</div>



## pivot table 로 데이터 집계하기
* groupby 로 했던 작업을 pivot_table로 똑같이 해봅니다.
* pivot table은 그룹화한다는 점에서 groupby와 차이남.(groupby가 좀 더 빠르다)


```python
# 지역명을 index 로 평당분양가격 을 values 로 구합니다.

pd.pivot_table(df_last, index = ["지역명"], values = ["평당분양가격"], aggfunc = "sum")
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
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>1909561.5</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>3339223.8</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>2289388.2</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>2018745.3</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>2000258.7</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>2899376.7</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>2030160.0</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>3021780.3</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>5899994.1</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>2321774.4</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>1492220.4</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>2931168.9</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>1876198.5</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>1915610.4</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>2461839.6</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>1967842.8</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>1908663.9</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_last.groupby(["전용면적"])["평당분양가격"].mean()
```




    전용면적
     102㎡~        11517.705634
     60㎡          10375.137421
     60㎡~ 85㎡     10271.040071
     85㎡~ 102㎡    11097.599573
    전체            10276.086207
    Name: 평당분양가격, dtype: float64




```python
pd.pivot_table(df_last, index="전용면적", values="평당분양가격")
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
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>11517.705634</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>10375.137421</td>
    </tr>
    <tr>
      <th>60㎡~ 85㎡</th>
      <td>10271.040071</td>
    </tr>
    <tr>
      <th>85㎡~ 102㎡</th>
      <td>11097.599573</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>10276.086207</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 전용면적을 index 로 평당분양가격 을 values 로 구합니다.

df_last.pivot_table(index=["전용면적", "지역명"], values= "평당분양가격")
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
      <th></th>
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">102㎡~</th>
      <th>강원</th>
      <td>8311.380000</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>14771.790000</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>10358.363265</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>9157.302000</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>11041.532432</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">전체</th>
      <th>전남</th>
      <td>7283.562000</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>7292.604000</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>10784.994000</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>7815.324000</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>7219.014000</td>
    </tr>
  </tbody>
</table>
<p>85 rows × 1 columns</p>
</div>




```python
df_last.groupby(["전용면적", "지역명"])["평당분양가격"].mean().unstack().round()
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
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>8311.0</td>
      <td>14772.0</td>
      <td>10358.0</td>
      <td>9157.0</td>
      <td>11042.0</td>
      <td>13087.0</td>
      <td>14877.0</td>
      <td>13208.0</td>
      <td>23446.0</td>
      <td>10107.0</td>
      <td>9974.0</td>
      <td>14362.0</td>
      <td>8168.0</td>
      <td>8194.0</td>
      <td>10523.0</td>
      <td>8689.0</td>
      <td>8195.0</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>7567.0</td>
      <td>13252.0</td>
      <td>8689.0</td>
      <td>7883.0</td>
      <td>9431.0</td>
      <td>11992.0</td>
      <td>9176.0</td>
      <td>11354.0</td>
      <td>23213.0</td>
      <td>9324.0</td>
      <td>9202.0</td>
      <td>11241.0</td>
      <td>7210.0</td>
      <td>7610.0</td>
      <td>14022.0</td>
      <td>7911.0</td>
      <td>7103.0</td>
    </tr>
    <tr>
      <th>60㎡~ 85㎡</th>
      <td>7486.0</td>
      <td>12524.0</td>
      <td>8619.0</td>
      <td>8061.0</td>
      <td>9911.0</td>
      <td>11779.0</td>
      <td>9711.0</td>
      <td>11865.0</td>
      <td>22787.0</td>
      <td>9775.0</td>
      <td>10503.0</td>
      <td>11384.0</td>
      <td>7269.0</td>
      <td>7271.0</td>
      <td>10621.0</td>
      <td>7819.0</td>
      <td>7264.0</td>
    </tr>
    <tr>
      <th>85㎡~ 102㎡</th>
      <td>8750.0</td>
      <td>13678.0</td>
      <td>10018.0</td>
      <td>8774.0</td>
      <td>9296.0</td>
      <td>11141.0</td>
      <td>9037.0</td>
      <td>12073.0</td>
      <td>25944.0</td>
      <td>9848.0</td>
      <td>8861.0</td>
      <td>11528.0</td>
      <td>7909.0</td>
      <td>8276.0</td>
      <td>10709.0</td>
      <td>9120.0</td>
      <td>8391.0</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>7478.0</td>
      <td>12560.0</td>
      <td>8659.0</td>
      <td>8079.0</td>
      <td>9904.0</td>
      <td>11771.0</td>
      <td>9786.0</td>
      <td>11936.0</td>
      <td>22610.0</td>
      <td>9805.0</td>
      <td>10493.0</td>
      <td>11257.0</td>
      <td>7284.0</td>
      <td>7293.0</td>
      <td>10785.0</td>
      <td>7815.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_last.pivot_table(index="전용면적", columns= "지역명", values="평당분양가격").round()
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
      <th>지역명</th>
      <th>강원</th>
      <th>경기</th>
      <th>경남</th>
      <th>경북</th>
      <th>광주</th>
      <th>대구</th>
      <th>대전</th>
      <th>부산</th>
      <th>서울</th>
      <th>세종</th>
      <th>울산</th>
      <th>인천</th>
      <th>전남</th>
      <th>전북</th>
      <th>제주</th>
      <th>충남</th>
      <th>충북</th>
    </tr>
    <tr>
      <th>전용면적</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>102㎡~</th>
      <td>8311.0</td>
      <td>14772.0</td>
      <td>10358.0</td>
      <td>9157.0</td>
      <td>11042.0</td>
      <td>13087.0</td>
      <td>14877.0</td>
      <td>13208.0</td>
      <td>23446.0</td>
      <td>10107.0</td>
      <td>9974.0</td>
      <td>14362.0</td>
      <td>8168.0</td>
      <td>8194.0</td>
      <td>10523.0</td>
      <td>8689.0</td>
      <td>8195.0</td>
    </tr>
    <tr>
      <th>60㎡</th>
      <td>7567.0</td>
      <td>13252.0</td>
      <td>8689.0</td>
      <td>7883.0</td>
      <td>9431.0</td>
      <td>11992.0</td>
      <td>9176.0</td>
      <td>11354.0</td>
      <td>23213.0</td>
      <td>9324.0</td>
      <td>9202.0</td>
      <td>11241.0</td>
      <td>7210.0</td>
      <td>7610.0</td>
      <td>14022.0</td>
      <td>7911.0</td>
      <td>7103.0</td>
    </tr>
    <tr>
      <th>60㎡~ 85㎡</th>
      <td>7486.0</td>
      <td>12524.0</td>
      <td>8619.0</td>
      <td>8061.0</td>
      <td>9911.0</td>
      <td>11779.0</td>
      <td>9711.0</td>
      <td>11865.0</td>
      <td>22787.0</td>
      <td>9775.0</td>
      <td>10503.0</td>
      <td>11384.0</td>
      <td>7269.0</td>
      <td>7271.0</td>
      <td>10621.0</td>
      <td>7819.0</td>
      <td>7264.0</td>
    </tr>
    <tr>
      <th>85㎡~ 102㎡</th>
      <td>8750.0</td>
      <td>13678.0</td>
      <td>10018.0</td>
      <td>8774.0</td>
      <td>9296.0</td>
      <td>11141.0</td>
      <td>9037.0</td>
      <td>12073.0</td>
      <td>25944.0</td>
      <td>9848.0</td>
      <td>8861.0</td>
      <td>11528.0</td>
      <td>7909.0</td>
      <td>8276.0</td>
      <td>10709.0</td>
      <td>9120.0</td>
      <td>8391.0</td>
    </tr>
    <tr>
      <th>전체</th>
      <td>7478.0</td>
      <td>12560.0</td>
      <td>8659.0</td>
      <td>8079.0</td>
      <td>9904.0</td>
      <td>11771.0</td>
      <td>9786.0</td>
      <td>11936.0</td>
      <td>22610.0</td>
      <td>9805.0</td>
      <td>10493.0</td>
      <td>11257.0</td>
      <td>7284.0</td>
      <td>7293.0</td>
      <td>10785.0</td>
      <td>7815.0</td>
      <td>7219.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 지역명, 전용면적으로 평당분양가격의 평균을 구합니다.
# df_last.groupby(["전용면적", "지역명"])["평당분양가격"].mean().unstack().round()
p = pd.pivot_table(df_last, index=["연도", "지역명"], values="평당분양가격")
p.loc[2017]
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
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>7273.560000</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>12304.980000</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>8786.760000</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>8280.800000</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>9613.977551</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>12206.700000</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>9957.158491</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>11560.680000</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>21831.060000</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>9132.505556</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>10666.935714</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>11640.600000</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>7372.920000</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>7398.973585</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>12566.730000</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>8198.422222</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>7473.120000</td>
    </tr>
  </tbody>
</table>
</div>




```python
pd.pivot_table(df_last, index=["연도"], values="평당분양가격")
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
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>연도</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015</th>
      <td>9202.735802</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>9683.025000</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>10360.487653</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>10978.938411</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>12188.293092</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 연도, 지역명으로 평당분양가격의 평균을 구합니다.
# g = df_last.groupby(["연도", "지역명"])["평당분양가격"].mean()

pd.pivot_table(df_last, index=["연도", "지역명"], values="평당분양가격")
#p.loc[2018]       #loc는 행을 기준으로 가져올때
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
      <th></th>
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>연도</th>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">2015</th>
      <th>강원</th>
      <td>7188.060000</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>11060.940000</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>8459.220000</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>7464.160000</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>7916.700000</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">2019</th>
      <th>전남</th>
      <td>8219.275862</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>8532.260000</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>11828.469231</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>8748.840000</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>7970.875000</td>
    </tr>
  </tbody>
</table>
<p>85 rows × 1 columns</p>
</div>




```python
p = pd.pivot_table(df_last, index=["연도", "지역명"], values="평당분양가격")
p.loc[2018]       #loc는 행을 기준으로 가져올때
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
      <th>평당분양가격</th>
    </tr>
    <tr>
      <th>지역명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강원</th>
      <td>8219.255000</td>
    </tr>
    <tr>
      <th>경기</th>
      <td>14258.420000</td>
    </tr>
    <tr>
      <th>경남</th>
      <td>9327.670000</td>
    </tr>
    <tr>
      <th>경북</th>
      <td>8680.776923</td>
    </tr>
    <tr>
      <th>광주</th>
      <td>9526.953333</td>
    </tr>
    <tr>
      <th>대구</th>
      <td>12139.252632</td>
    </tr>
    <tr>
      <th>대전</th>
      <td>10234.106667</td>
    </tr>
    <tr>
      <th>부산</th>
      <td>12889.965000</td>
    </tr>
    <tr>
      <th>서울</th>
      <td>23202.245000</td>
    </tr>
    <tr>
      <th>세종</th>
      <td>10340.463158</td>
    </tr>
    <tr>
      <th>울산</th>
      <td>10241.400000</td>
    </tr>
    <tr>
      <th>인천</th>
      <td>11881.532143</td>
    </tr>
    <tr>
      <th>전남</th>
      <td>7929.845000</td>
    </tr>
    <tr>
      <th>전북</th>
      <td>8174.595000</td>
    </tr>
    <tr>
      <th>제주</th>
      <td>11935.968000</td>
    </tr>
    <tr>
      <th>충남</th>
      <td>8201.820000</td>
    </tr>
    <tr>
      <th>충북</th>
      <td>8149.295000</td>
    </tr>
  </tbody>
</table>
</div>



## 최근 데이터 시각화 하기
### 데이터시각화를 위한 폰트설정
한글폰트 사용을 위해 matplotlib의 pyplot을 plt라는 별칭으로 불러옵니다.


```python
import matplotlib.pyplot as plt

plt.rc("font", family="Malgun Gothic")
#plt.rc("font", family="AppleGothic")
```

### Pandas로 시각화 하기 - 선그래프와 막대그래프
pandas의 plot을 활용하면 다양한 그래프를 그릴 수 있습니다.
seaborn을 사용했을 때보다 pandas를 사용해서 시각화를 할 때의 장점은 미리 계산을 하고 그리기 때문에 속도가 좀 더 빠릅니다.


```python
# 지역명으로 분양가격의 평균을 구하고 선그래프로 시각화 합니다.

g = df_last.groupby(["지역명"])["평당분양가격"].mean().plot()
g
```




    <AxesSubplot:xlabel='지역명'>




    
![png](output_61_1.png)
    



```python
# 지역명으로 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.
g = df_last.groupby(["지역명"])["평당분양가격"].mean()
g.plot(kind = "bar") #또는 g.plot.bar() 라고 쓰기
```




    <AxesSubplot:xlabel='지역명'>




    
![png](output_62_1.png)
    



```python
g = df_last.groupby(["지역명"])["평당분양가격"].mean().sort_values(ascending=False)
                                 #sort_values()는 크기순서대로 나열
                                 #ascending=False는 큰거부터 작은거대로
g.plot()        
```




    <AxesSubplot:xlabel='지역명'>




    
![png](output_63_1.png)
    



```python
g.plot.bar(rot=0, figsize=(10, 3)) #rot=0은 가로로 지역명, figsize는 간격
```




    <AxesSubplot:xlabel='지역명'>




    
![png](output_64_1.png)
    


전용면적별 분양가격의 평균값을 구하고 그래프로 그려봅니다.


```python
# 전용면적으로 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.

p = df_last.groupby(["전용면적"])["평당분양가격"].mean()
p.plot.bar()
```




    <AxesSubplot:xlabel='전용면적'>




    
![png](output_66_1.png)
    



```python
# 연도별 분양가격의 평균을 구하고 막대그래프(bar)로 시각화 합니다.
p = df_last.groupby(["연도"])["평당분양가격"].mean()
p.plot.bar()

```




    <AxesSubplot:xlabel='연도'>




    
![png](output_67_1.png)
    


### box-and-whisker plot | diagram

* https://pandas.pydata.org/pandas-docs/stable/user_guide/visualization.html
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.boxplot.html

* [상자 수염 그림 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EC%83%81%EC%9E%90_%EC%88%98%EC%97%BC_%EA%B7%B8%EB%A6%BC)
* 가공하지 않은 자료 그대로를 이용하여 그린 것이 아니라, 자료로부터 얻어낸 통계량인 5가지 요약 수치로 그린다.
* 5가지 요약 수치란 기술통계학에서 자료의 정보를 알려주는 아래의 다섯 가지 수치를 의미한다.


1. 최솟값
1. 제 1사분위수
1. 제 2사분위수( ), 즉 중앙값
1. 제 3 사분위 수( )
1. 최댓값

* Box plot 이해하기 : 
    * [박스 플롯에 대하여 :: -[|]- Box and Whisker](https://boxnwhis.kr/2019/02/19/boxplot.html)
    * [Understanding Boxplots – Towards Data Science](https://towardsdatascience.com/understanding-boxplots-5e2df7bcbd51)


```python
# index를 월, columns 를 연도로 구하고 평당분양가격 으로 pivot_table 을 
#구하고 상자수염그림을 그립니다.

df_last.pivot_table(index="월", columns = "연도", values = "평당분양가격").plot.box()

```




    <AxesSubplot:>




    
![png](output_69_1.png)
    



```python
# columns 에 "연도", "전용면적"을 추가해서 pivot_table 을 만들고 시각화 합니다.

p=df_last.pivot_table(index="월", columns = ["연도", "전용면적"], values = "평당분양가격")
p.plot.box(figsize=(15,3), rot=30)  #rot=30 글자 기울여서, figsize=(15,3) 그래프사이즈
```




    <AxesSubplot:>




    
![png](output_70_1.png)
    



```python
p=df_last.pivot_table(index="연도", columns = "월", values = "평당분양가격")
p.plot(figsize=(15,3), rot=30)
```


```python
# index를 월, columns 를 지역명으로 구하고 평당분양가격 으로 pivot_table 을 구하고 선그래프를 그립니다.

p = df_last.pivot_table(index="월", columns = "지역명", values = "평당분양가격").plot()
p.plot(figsize=(15,3), rot=10)
```

### Seaborn 으로 시각화 해보기


```python
# 라이브러리 로드하기
import seaborn as sns

%matplotlib inline
```


```python
# barplot으로 지역별 평당분양가격을 그려봅니다.
plt.figure(figsize = (10,3))
sns.barplot(data = df_last, x = "지역명", y= "평당분양가격")
```




    <AxesSubplot:xlabel='지역명', ylabel='평당분양가격'>




    
![png](output_75_1.png)
    



```python
# barplot으로 연도별 평당분양가격을 그려봅니다.

sns.barplot(data = df_last, x="연도", y="평당분양가격" )
#평균가격이 꾸준히 상승중
```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![png](output_76_1.png)
    



```python
# catplot 으로 서브플롯 그리기

sns.catplot(data=df_last, x="연도", y="평당분양가격", kind="bar", col="지역명", col_wrap=4)

```




    <seaborn.axisgrid.FacetGrid at 0x17df4a684f0>




    
![png](output_77_1.png)
    


https://stackoverflow.com/questions/30490740/move-legend-outside-figure-in-seaborn-tsplot


```python
# lineplot으로 연도별 평당분양가격을 그려봅니다.
# hue 옵션을 통해 지역별로 다르게 표시해 봅니다.
plt.figure(figsize = (10,5))
sns.lineplot(data = df_last, x = "연도", y= "평당분양가격", hue="지역명")
plt.legend(bbox_to_anchor=(1.02, 1), loc=2, borderaxespad=0.)
```




    <matplotlib.legend.Legend at 0x17df24f21f0>




    
![png](output_79_1.png)
    



```python
# relplot 으로 서브플롯 그리기

sns.relplot(data=df_last, x="연도", y="평당분양가격",
            hue="지역명", kind = "line", col="지역명", col_wrap=4, ci=None)

#서울이 분양가격이 제일 높다.
```




    <seaborn.axisgrid.FacetGrid at 0x17df2af54c0>




    
![png](output_80_1.png)
    


### boxplot과 violinplot


```python
# 연도별 평당분양가격을 boxplot으로 그려봅니다.
# 최솟값
# 제 1사분위수
# 제 2사분위수( ), 즉 중앙값
# 제 3 사분위 수( )
# 최댓값

#sns.boxplot(data=df_last, x="지역명", y="평당분양가격", hue="연도") 했더니 그래프 해석
#어렵다. 그러므로 hue는 카테고리 값이 적을때 용이.

sns.boxplot(data=df_last, x="연도", y="평당분양가격")
```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![png](output_82_1.png)
    



```python
# hue옵션을 주어 전용면적별로 다르게 표시해 봅니다.
plt.figure(figsize=(12,3))
sns.boxplot(data=df_last, x="연도", y="평당분양가격", hue="전용면적")
```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![png](output_83_1.png)
    



```python
# 연도별 평당분양가격을 violinplot으로 그려봅니다.

sns.violinplot(data=df_last, x="연도", y="평당분양가격")

#box plot의 단점(raw data의 분포도가 변해도 일정함)을 보완해주는게 
#violinplot(raw data의 변화를 표시해줌)
```




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![png](output_84_1.png)
    


### lmplot과 swarmplot 


```python
# 연도별 평당분양가격을 lmplot으로 그려봅니다. 
# hue 옵션으로 전용면적을 표현해 봅니다.

#sns.regplot(data=df_last, x="연도", y="평당분양가격", hue="전용면적") 
#regplot은 scatter plot에 회귀선(데이터가 얼마나 상관관계가 있는가)을 그어주는것

sns.lmplot(data=df_last, x="연도", y="평당분양가격", hue="전용면적", col="전용면적", col_wrap=3) 
```




    <seaborn.axisgrid.FacetGrid at 0x17dfc358fd0>




    
![png](output_86_1.png)
    



```python
# 연도별 평당분양가격을 swarmplot 으로 그려봅니다. 
# swarmplot은 범주형(카테고리) 데이터의 산점도를 표현하기에 적합합니다.
plt.figure(figsize=(15,3))
sns.swarmplot(data=df_last, x="연도", y="평당분양가격", hue="전용면적")
```

    C:\Users\USER\anaconda3\lib\site-packages\seaborn\categorical.py:1296: UserWarning: 40.8% of the points cannot be placed; you may want to decrease the size of the markers or use stripplot.
      warnings.warn(msg, UserWarning)
    C:\Users\USER\anaconda3\lib\site-packages\seaborn\categorical.py:1296: UserWarning: 73.3% of the points cannot be placed; you may want to decrease the size of the markers or use stripplot.
      warnings.warn(msg, UserWarning)
    C:\Users\USER\anaconda3\lib\site-packages\seaborn\categorical.py:1296: UserWarning: 62.5% of the points cannot be placed; you may want to decrease the size of the markers or use stripplot.
      warnings.warn(msg, UserWarning)
    C:\Users\USER\anaconda3\lib\site-packages\seaborn\categorical.py:1296: UserWarning: 62.7% of the points cannot be placed; you may want to decrease the size of the markers or use stripplot.
      warnings.warn(msg, UserWarning)
    C:\Users\USER\anaconda3\lib\site-packages\seaborn\categorical.py:1296: UserWarning: 60.0% of the points cannot be placed; you may want to decrease the size of the markers or use stripplot.
      warnings.warn(msg, UserWarning)
    




    <AxesSubplot:xlabel='연도', ylabel='평당분양가격'>




    
![png](output_87_2.png)
    


### 이상치 보기


```python
# 평당분양가격의 최대값을 구해서 max_price 라는 변수에 담습니다.
df_last["평당분양가격"].describe()
```




    count     3957.000000
    mean     10685.824488
    std       4172.222780
    min       6164.400000
    25%       8055.300000
    50%       9484.200000
    75%      11751.300000
    max      42002.400000
    Name: 평당분양가격, dtype: float64




```python
max_price = df_last["평당분양가격"].max()
max_price
```




    42002.399999999994




```python
# 서울의 평당분양가격이 특히 높은 데이터가 있습니다. 해당 데이터를 가져옵니다.

df_last[df_last["평당분양가격"] == max_price]
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
      <th>지역명</th>
      <th>연도</th>
      <th>월</th>
      <th>분양가격</th>
      <th>평당분양가격</th>
      <th>전용면적</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3743</th>
      <td>서울</td>
      <td>2019</td>
      <td>6</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
    <tr>
      <th>3828</th>
      <td>서울</td>
      <td>2019</td>
      <td>7</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
    <tr>
      <th>3913</th>
      <td>서울</td>
      <td>2019</td>
      <td>8</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
    <tr>
      <th>3998</th>
      <td>서울</td>
      <td>2019</td>
      <td>9</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
    <tr>
      <th>4083</th>
      <td>서울</td>
      <td>2019</td>
      <td>10</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
    <tr>
      <th>4168</th>
      <td>서울</td>
      <td>2019</td>
      <td>11</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
    <tr>
      <th>4253</th>
      <td>서울</td>
      <td>2019</td>
      <td>12</td>
      <td>12728.0</td>
      <td>42002.4</td>
      <td>85㎡~ 102㎡</td>
    </tr>
  </tbody>
</table>
</div>



### 수치데이터 히스토그램 그리기

distplot은 결측치가 있으면 그래프를 그릴 때 오류가 납니다. 
따라서 결측치가 아닌 데이터만 따로 모아서 평당분양가격을 시각화하기 위한 데이터를 만듭니다.
데이터프레임의 .loc를 활용하여 결측치가 없는 데이터에서 평당분양가격만 가져옵니다.


```python

```


```python

```


```python
# 결측치가 없는 데이터에서 평당분양가격만 가져옵니다. 그리고 price라는 변수에 담습니다.
# .loc[행]
# .loc[행, 열]

h = df_last["평당분양가격"].hist(bins=10) 
#bins = 10 10개의 구간으로 나누어서 표현
```


    
![png](output_96_0.png)
    



```python
price = df_last.loc[df_last["평당분양가격"].notnull(), "평당분양가격"]
```


```python
# distplot으로 평당분양가격을 표현해 봅니다.

sns.distplot(price)
```




    <AxesSubplot:xlabel='평당분양가격', ylabel='Density'>




    
![png](output_98_1.png)
    



```python
# sns.distplot(price, hist=False, rug=True)


```

* distplot을 산마루 형태의 ridge plot으로 그리기
* https://seaborn.pydata.org/tutorial/axis_grids.html#conditional-small-multiples
* https://seaborn.pydata.org/examples/kde_ridgeplot.html


```python
# subplot 으로 표현해 봅니다.


```


```python
# pairplot


```


```python
# 규모구분(전용면적)별로 value_counts를 사용해서 데이터를 집계해 봅니다.


```

## 2015년 8월 이전 데이터 보기


```python
# 모든 컬럼이 출력되게 설정합니다.


```


```python
# head 로 미리보기를 합니다.


```


```python
# df_first 변수에 담겨있는 데이터프레임의 정보를 info를 통해 봅니다.


```


```python
# 결측치가 있는지 봅니다.


```

### melt로 Tidy data 만들기
pandas의 melt를 사용하면 데이터의 형태를 변경할 수 있습니다. 
df_first 변수에 담긴 데이터프레임은 df_last에 담겨있는 데이터프레임의 모습과 다릅니다. 
같은 형태로 만들어주어야 데이터를 합칠 수 있습니다. 
데이터를 병합하기 위해 melt를 사용해 열에 있는 데이터를 행으로 녹여봅니다.

* https://pandas.pydata.org/docs/user_guide/reshaping.html#reshaping-by-melt
* [Tidy Data 란?](https://vita.had.co.nz/papers/tidy-data.pdf)


```python
# head 로 미리보기 합니다.


```


```python
# pd.melt 를 사용하며, 녹인 데이터는 df_first_melt 변수에 담습니다. 



```


```python
# df_first_melt 변수에 담겨진 컬럼의 이름을 
# ["지역명", "기간", "평당분양가격"] 으로 변경합니다.


```

### 연도와 월을 분리하기
* pandas 의 string-handling 사용하기 : https://pandas.pydata.org/pandas-docs/stable/reference/series.html#string-handling


```python
date = "2013년12월"
date
```


```python
# split 을 통해 "년"을 기준으로 텍스트를 분리해 봅니다.


```


```python
# 리스트의 인덱싱을 사용해서 연도만 가져옵니다.


```


```python
# 리스트의 인덱싱과 replace를 사용해서 월을 제거합니다.


```


```python
# parse_year라는 함수를 만듭니다.
# 연도만 반환하도록 하며, 반환하는 데이터는 int 타입이 되도록 합니다.

```


```python
# 제대로 분리가 되었는지 parse_year 함수를 확인합니다.


```


```python
# parse_month 라는 함수를 만듭니다.
# 월만 반환하도록 하며, 반환하는 데이터는 int 타입이 되도록 합니다.

```


```python
# 제대로 분리가 되었는지 parse_month 함수를 확인합니다.


```


```python
# df_first_melt 변수에 담긴 데이터프레임에서 
# apply를 활용해 연도만 추출해서 새로운 컬럼에 담습니다.


```


```python
# df_first_melt 변수에 담긴 데이터프레임에서 
# apply를 활용해 월만 추출해서 새로운 컬럼에 담습니다.


```


```python
# 컬럼명을 리스트로 만들때 버전에 따라 tolist() 로 동작하기도 합니다.
# to_list() 가 동작하지 않는다면 tolist() 로 해보세요.


```


```python
# df_last와 병합을 하기 위해서는 컬럼의 이름이 같아야 합니다.
# sample을 활용해서 데이터를 미리보기 합니다.


```


```python
cols = ['지역명', '연도', '월', '평당분양가격']
cols
```


```python
# 최근 데이터가 담긴 df_last 에는 전용면적이 있습니다. 
# 이전 데이터에는 전용면적이 없기 때문에 "전체"만 사용하도록 합니다.
# loc를 사용해서 전체에 해당하는 면적만 copy로 복사해서 df_last_prepare 변수에 담습니다.


```


```python
# df_first_melt에서 공통된 컬럼만 가져온 뒤
# copy로 복사해서 df_first_prepare 변수에 담습니다.


```

### concat 으로 데이터 합치기
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.concat.html


```python
# df_first_prepare 와 df_last_prepare 를 합쳐줍니다.


```


```python
# 제대로 합쳐졌는지 미리보기를 합니다.


```


```python
# 연도별로 데이터가 몇개씩 있는지 value_counts를 통해 세어봅니다.


```

### pivot_table 사용하기
* https://pandas.pydata.org/docs/user_guide/reshaping.html#reshaping-and-pivot-tables


```python
# 연도를 인덱스로, 지역명을 컬럼으로 평당분양가격을 피봇테이블로 그려봅니다.

```


```python
# 위에서 그린 피봇테이블을 히트맵으로 표현해 봅니다.


```


```python
# transpose 를 사용하면 행과 열을 바꿔줄 수 있습니다.


```


```python
# 바뀐 행과 열을 히트맵으로 표현해 봅니다.


```


```python
# Groupby로 그려봅니다. 인덱스에 ["연도", "지역명"] 을 넣고 그려봅니다.


```


```python

```

## 2013년부터 최근 데이터까지 시각화하기
### 연도별 평당분양가격 보기


```python
# barplot 으로 연도별 평당분양가격 그리기

```


```python
# pointplot 으로 연도별 평당분양가격 그리기

```


```python
# 서울만 barplot 으로 그리기

```


```python
# 연도별 평당분양가격 boxplot 그리기


```


```python

```


```python
# 연도별 평당분양가격 violinplot 그리기


```


```python
# 연도별 평당분양가격 swarmplot 그리기


```

### 지역별 평당분양가격 보기


```python
# barplot 으로 지역별 평당분양가격을 그려봅니다.


```


```python
# boxplot 으로 지역별 평당분양가격을 그려봅니다.

```


```python

```


```python
# violinplot 으로 지역별 평당분양가격을 그려봅니다.

```


```python
# swarmplot 으로 지역별 평당분양가격을 그려봅니다.


```


```python

```


```python

```

<big> 고생 많으셨습니다! 다음 챕터도 화이팅 입니다!</big>


```python

```


```python

```
