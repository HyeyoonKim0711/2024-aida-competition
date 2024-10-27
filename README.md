
# 대전 중구 지역경제 활성화를 위한 성심당 방문객 행선지 추천 시스템
소비패턴 분석과 크롤링을 바탕으로

## 1. 분석 배경 설명 및 분석 과정
### 1.1 분석 배경
- 성심당은 대전의 방문객 유입을 늘리며 대전 내 압도적인 인기 관광지로 자리잡음
- 상생 프로젝트 등을 통해 지역 경제 활성화에도 기여하고 있음
- 대전세종포럼(2024 가을 통권 제90호, 대전세종연구원)에 따르면 대전 관광객들의 **목적지의 약 60%가 성심당**이며, 관광유형은 **대부분이 당일치기 여행**임
- 현재 성심당의 인기는 대전 중구 전체 상권 및 관광 산업 활성화로 이어지지 않고 있음.

  ➡️ 이용 고객이 타 관광지도 방문하고 체류형 관광으로 발전하기 위해서는 **데이터 분석이 필요** 


### 1.2 분석 과정
![image](https://github.com/user-attachments/assets/08e866f2-87b0-44fb-9349-fffd4cb53cc2)



## 2. 성심당 방문객의 이동경로 

### 2.1 데이터를 이용한 성심당 방문객 추출

#### "Cell Id별/업종별 가맹점 실적정보.csv"로 부터 성심당 매장 정보 추출

- 업종 소분류명에 '빵' 혹은 '제과'라는 단어가 있는 가맹점만 추출 
- 다수의 ‘기준일자’에 대해 cell_id가 **‘다바93a14a’**, 업종소분류가 **‘제과점업(I56191)’** 인 가맹점의 이용건수가 가장 많음을 확인

#### "지역별 업종 유입고객 및 상권특성 정보.csv"로 부터 성심당 방문객 정보 추출 
- 행정동_상권 : 30140535(은행선화동)
- 업종소분류 : I56191
- 광역시_유입 : 30(대전광역시) 제외
- 업력 : 10년초과
- 구분_시간대 : 5(22시-06시) 제외
- 구분_개인법인 : 개인

두 카드매출 데이터로부터 성심당 방문객을 추출하였고, 이 정보를 담은 데이터프레임을 "sungsim_card.csv"파일로 저장


### 2.2 성심당 방문객들의 이동경로 분석을 위한 데이터 생성 과정

1. 가맹점 실적정보 데이터로부터 이용건수와 이용금액이 높은 **2024년 5월** 데이터만 시각화 대상으로 선택
    - 효과적으로 시각화하기 위해 1년 데이터 전부를 시각화하기보다 이용건수의 합과 이용금액의 합이 동시에 가장 높은 5월만 선택
      <table>
        <tr>
          <td align="center">
            <img src="https://github.com/user-attachments/assets/2b2f7d74-6043-4413-98ef-683b74a44082" alt="스크린샷 2024-10-25 214405" />
            <p align="center">
            <이용건수의 합>
            </p>
          </td>
          <td align="center">
            <img src="https://github.com/user-attachments/assets/fc7c902c-7bd4-4608-8608-a61ca2c5d19d" alt="스크린샷 2024-10-25 214451" />
            <p align="center">
            <이용금액의 합>
            </p>
          </td>
        </tr>
      </table>


2. 성심당에서 가장 많은 소비를 한 **20대 여성**을 시각화 대상으로 선택
    - 지역별 업종 유입고객 데이터로부터 5월에 20대 여성이 성심당에서 이용건수의 합과 이용금액의 합이 가장 높은 것을 확인
      <table>
        <tr>
          <td align="center">
            <img src="https://github.com/user-attachments/assets/a12828ce-5c28-42be-b31c-1ab13a95ceca" alt="스크린샷 2024-10-25 214813" />
            <p align="center">
            <5월 이용건수의 합 상위 3개월>
          </p>
        </td>
        <td align="center">
          <img src="https://github.com/user-attachments/assets/9481ee90-4f0b-4e4b-907e-5361fdf4a768" alt="스크린샷 2024-10-25 214840" />
          <p align="center">
          <5월 이용금액의 합 상위 3개월>
           </p>
          </td>
        </tr>
      </table>


3. 유동인구 대표값으로 **중앙값(Median)**사용
   - 5월 유동인구 데이터에서 모든 셀(cell)에 대하여 시간대별 휴일과 평일을 구분한 후 20대 여성의 유동인구를 살펴본 결과,0의 값이 많고 오른쪽으로 긴 꼬리를 가진 분포임을 확인
   - 이러한 분포 특성상 평균보다는 중앙값이 유동인구의 특징을 더 잘 나타내는 지표가 되기 때문에 중앙값을 사용 
  ![output](https://github.com/user-attachments/assets/886110bd-4ca5-4488-bf3a-ca0adcbbb8f8)


4. 유동인구 데이터 전처리
    - '성별_연령' 변수의 연령 구간을 10세 단위로 재범주화
    - 평일(월~금)과 휴일(토,일,공휴일)을 나타내는 'holiday' 변수 생성
    - 유동인구의 ''x좌표', 'y좌표'를 WGS84 좌표계로 변환하여 'latitude', 'longitude' 변수 생성
    - 유동인구 데이터의 변수 중에서 '셀번호', 'x좌표', 'y좌표', '시간대', '행정동코드', '일자', 'holiday', '여성_20대' 만 선택


=>이 과정을 거쳐 만들어진 최종 데이터를 **"final_pop_may_20.csv"** 파일로 저장하였고, 이 데이터를 **유동인구 시각화** 하는데 사용함


### 2.3 성심당 방문객들의 이동경로 시각화 및 분석

#### 시각화 
- final_pop_may_20의 데이터의 값을 평일과 휴일로 나누어 시간대별로 변화하는 히트맵을 지도 위에 나타냄
- 행정동 파악을 쉽게 하기 위해 대전 중구 지역의 행정구역도를 표시함
- 유동인구 변화하는 히트맵을 gif 파일로 저장하였으나 용량이 커 figures 폴더에 첨부함 
<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/03d266dc-5f2a-497b-ab75-8c48263ddbd6" alt="heatmap_평일_12" />
      <p align="center">
      <평일 12시 유동인구>
      </p>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/cd987530-9b72-4d09-b250-bbb6729d14a0" alt="heatmap_휴일_12" />
      <p align="center">
      <휴일 12시 유동인구>
      </p>
    </td>
  </tr>
</table>


- 시각화한 결과를 더 상세히 파악하기 위해
 **각 시간대마다 유동인구가 높은 5개의 cell**만을 추출하여 다시 시각화
<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/8e89349d-9ed8-419e-8cb7-786170d3ce1f" alt="heatmap_top5_평일_12" />
      <p align="center">
      <평일 12시, 상위 5개 cell의 유동인구>
      </p>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/7024a0e8-5c51-4d9b-903a-e0233da420b2" alt="heatmap_top5_휴일_12" />
      <p align="center">
      <휴일 12시, 상위 5개 cell의 유동인구>
      </p>
    </td>
  </tr>
</table>

#### 20대 여성의 유동인구 변화 분석
##### 평일
- 서대전 네거리가 위치한 '문화1동'은 대전 중구의 교통 중심으로 모든 시간대에 유동인구가 높음 
- 9시 ~ 19시 시간대에는 성심당 근처의 '은행선화동'에서 유동인구가 가장 높음
- 19시-21시 한화이글스파크가 위치한 '부사동'에서도 유동인구가 높음
- 20시부터는 '은행선화동' 중심의 유동인구가 '대흥동'으로 이동
##### 휴일
- 8시 ~ 14시, 19시 ~ 20시 시간대에 서대전역이 위치한 '류동'에 유동인구가 높음 
- 평일과 유사하게 서대전 네거리가 위치한 
'문화1동'에서 모든 시간대에 유동인구가 높음 
- 9시 ~ 22시 시간대에  성심당 본점이 위치한 '은행선화동'에서 유동인구가 계속 높음
- '대흥동'에서도 모든 시간대에 유동인구가 높음

## 3. 성심당 이용 고객 분석
### 3.1 성심당 이용 고객 파악을 위한 데이터 전처리
- 성심당 데이터셋(sungsim_card.csv)에 업종코드 및 행정동코드 데이터 연결
    ➡️ 유입 시도명과 업종 소분류명 파악하기 위함
- 방문객의 유형을 쉽게 파악하기 위해 '유형' 변수 생성
    = 유입 시도명 + 연령 + 성별 + 개인/법인 여부
- 관광객들의 소비만 알아보기 위해 특정 업종만 필터링
    = 중분류-'소매업; 자동차 제외', 대분류-'숙박 및 음식점업', '예술, 스포츠 및 여가관련 서비스업'


### 3.2 이용건수 상위의 출발 도시 파악
- 평일과 휴일을 나누어 성심당 이용 고객 시도별 비율 파악
![image](https://github.com/user-attachments/assets/2f52f3f4-0bde-4f77-b3d5-ad09eadde97d)
- 평일과 휴일 모두 경기도와 서울특별시가 압도적으로 큰 비중을 차지함을 확인.
- 그 다음으로는 인접 도시인 세종특별자치시
    즉 성심당 이용 고객들은 대부분 근교에서 방문함

**➡️ 당일치기 여행이 많다는 근거**



### 3.2 평일/휴일 시간대별 이용건수가 큰 상위 5개의 유형을 추출

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/43698d34-d037-477b-9385-7601557d6f04" alt="평일 시간대별 상위 유형 5개" width="100%" />
      <p align="center"
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/17025b0e-337c-4ec5-8da2-8a68a0fc4256" alt="휴일 시간대별 상위 유형 5개" width="100%" />
      <p align="center"
    </td>
  </tr>
</table>

- 평일/주말 모두 비슷한 유형 나타남
- **서울특별시&경기도에서 유입된 20대 여성이 주 이용 고객**
- 그 다음으로는 경기도 30대 여성, 경기도 30-40대 남성, 서울특별시 20대 남성
- 성심당 오픈 직후 ~ 저녁 전까지 가장 소비 多

### 3.3 성심당 주 이용 고객 유형의 대전 중구 전체 소비 이력
![image](https://github.com/user-attachments/assets/c794c304-848f-4a9a-a081-0615916b1722)
- 성심당 이용이 가장 많았던 경기도&서울특별시에서 유입된 20-30대 여성의 1년치 **대전 중구 전체의 소비 데이터** 파악
- 각 시간대별로 제과점업이 가장 높은 항목

    **➡️ 성심당이 대전 주 소비**



<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/5b3245ee-6057-442c-8200-2b8e4b6b21a5" alt="수도권 20-30대 남성 소비항목" width="100%" />
      <p align="center"
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/a5a93016-b2fd-4b50-9c06-f287335ef9c0" alt="수도권 40-50대 남성 소비항목" width="100%" />
      <p align="center"
    </td>
  </tr>
</table>  

- 그 다음으로 소비 많았던 경기도 & 서울특별시에서 유입된
    20-30대 남성과 40-50대 남성의 소비 이력도 파악 


    ➡️ 수도권 유입 20-30대 여성과 매우 유사한 패턴을 보임



> - 성심당과 식당, 카페 등의 소비는 많이 이루어지지만,
**숙박이나 관광지 관련 소비는 매우 적음**
>
> - 점심과 오후에 가장 이용 건수 높았다가 **18-22 시간대에는 감소**
>
>- 대부분이 **당일치기 여행**이며, **성심당과 식사 외의 관광 이루어지지 않음**




## 4. Pain Point & Solution
![image](https://github.com/user-attachments/assets/68a6d166-85eb-4259-9a6f-8e9a64b19c05)


**1. 관광지 데이터를 수집한 후 크롤링**  
**2. 전처리를 거쳐 tf-idf 행렬 생성**  
**3. 생성된 tf-idf 행렬로 관관지들을 군집화**  
**4. 사용자의 유동 패턴을 고려하여 관광지 우선 순위 추천**  

❓관광지가 이미 역사관광, 문화관광 등으로 구분되어 있는데 군집화를 하고자 하는 이유 :

기존의 구분은 관광지의 성격만 나타낼 뿐 관광객들이 느끼는 감정과 감성을 반영하고 있지 않음  
➡️ 크롤링을 통해 관광객들이 생각하는 관광지의 감상과 느낌 추출 & 더 세부적인 테마나 감성적인 카테고리 도출 가능
<img src="https://github.com/user-attachments/assets/fd9b0684-ac50-4e24-b450-c10c125179d6" width="1000"/>  
<br>

## 5. 관광지 추천 시스템 구현
### 5.1 관광지 데이터 수집
대전 중구에 위치한 관광지들을 파악하기 위해 한국관광 데이터랩 사이트에서 대전 중구의 중심 관광지명들을 파일로 수집 (대전_관광지_수정(36행).csv)

출처: https://datalab.visitkorea.or.kr/datalab/portal/loc/getAreaDataForm.do#

<img src="https://github.com/user-attachments/assets/23512b99-fec3-42c3-959b-0b86ed0b7784" width="600"/>

- 받아온 관광지명들 중 모텔과 호텔, 위치를 특정하기 어려운 관광지, 기간이 한정적인 축제를 제외한 34개의 관광지를 가지고 네이버 블로그 크롤링
- 보문산 공원, 보문산 숲속공연장, 대전오월드 버드랜드와 같은 부속시설들은 보문산, 대전오월드 같은 상위시설로 합쳐서 크롤링
- 우리들공원, 대전오월드, 한화생명이글스파크 ...
<br>

### 5.2 블로그 크롤링

```ruby
# 웹 드라이버 설정
options = webdriver.ChromeOptions()
options.add_experimental_option("excludeSwitches", ["enable-automation"])
options.add_experimental_option("useAutomationExtension", False)

# 버전에 상관 없이 os에 설치된 크롬 브라우저 사용
driver = webdriver.Chrome()
driver.implicitly_wait(3)

# Naver API key 입력
client_id = 'Xn3Yn**********ce2w3' 
client_secret = '*******_lM'
```
- naver api key를 발급받아 client_id와 client_secret에 입력
<br>

```ruby
naver_urls = []
postdate = []
titles = []
data = []
place = []

# 기준 날짜를 2020년 1월 1일로 설정
filter_date = '20200101'

for name in place_names:
    # 검색어 입력 (대전 반드시 포함 / 제외 단어 필터링)
    encText = urllib.parse.quote(f"{name} + 대전 -부동산 -월세 -전세 -분양 -주택 -공주 -주차장 -코스 -여행")

    # 검색을 끝낼 페이지 입력
    end = 6
    # 한번에 가져올 페이지 입력
    display = 100

    count = 0
    for start in range(end):
        url = "https://openapi.naver.com/v1/search/blog?query=" + encText + "&start=" + str(start+1) + "&display=" + str(display) # JSON 결과
        request = urllib.request.Request(url)
        request.add_header("X-Naver-Client-Id", client_id)
        request.add_header("X-Naver-Client-Secret", client_secret)
        response = urllib.request.urlopen(request)
        rescode = response.getcode()
    
        if rescode == 200:
            response_body = response.read()
            crawling = json.loads(response_body.decode('utf-8'))['items']
    
            for row in crawling:
                post_date = row['postdate']
                if post_date >= filter_date and 'blog.naver' in row['link']:
                    row['place'] = name
                    naver_urls.append(row['link'])
                    postdate.append(post_date)
                    place.append(name)
                
                    # html태그 제거
                    title = row['title']
                    pattern1 = '<[^>]*>'
                    title = re.sub(pattern=pattern1, repl='', string=title)
                    titles.append(title)
                
                    data.append(row)
                    count += 1  # 조건에 맞는 게시물 카운트 증가
                
                    if count >= 50:  # 50개를 넘기지 않도록 제한
                        break
            
            if count >= 50:
                break
            time.sleep(2)  # API 호출 간 딜레이
        else:
            print("Error Code:" + str(rescode))

    print(f"{name}, {count}개 크롤링 완료")
```

<img src="https://github.com/user-attachments/assets/c581bd01-0b0e-4153-8622-39f097c09f68" width="600"/>  

- 2020년 이후의 블로그 글만 크롤링
- 하나의 블로그에 여러 관광지들에 대한 리뷰가 같이 있는 경우가 많은 ‘여행’ 과 ‘코스’ 키워드를 제외
- 부동산 관련 홍보 블로그가 많이 검색되는 ‘부동산’, ‘월세’, ‘분양’, ‘주택’ 키워드를 제외
- 그 외에 불필요하게 같이 검색되었던 ‘공주’와 ‘주차장’ 키워드도 제외
- 관광지들 중 다른 지역에 같은 이름의 관광지가 존재하는 것이 있어 ‘대전’ 키워드를 반드시 포함해 검색
  
**⇒ 34개 관광지에 대해 50개씩, 총 1700개의 블로그 크롤링 (blog.csv)**  
<br>

### 5.3 블로그 글 전처리

<img src="https://github.com/user-attachments/assets/b1a4763a-451e-4547-8fd6-a9a3e867a1fd" width="1000"/>

- 블로그의 글을 가지고 군집화를 하기 전 단어의 출현빈도나 관계를 파악하기 위해 토큰화 진행
- 한국어는 조사와 어미가 붙기 때문에 영어와 달리 단순 띄어쓰기로는 토큰화가 어려움
- 따라서 Okt(Open Korea Text)로 형태소를 분리한 후 토큰화를 진행
- 각 관광지별로 50개의 블로그글을 하나의 문장으로 합친 후 관광지의 성격을 나타낼 수 있는 형태소인 명사와 동사, 형용사만 추출  
<br>

<img src="https://github.com/user-attachments/assets/d885fd57-a5ee-4ac5-ac0a-68044b66ea68" width="800"/>  

- 관광지명들을 불용어 사전에 넣어 제거
- 추출한 명사, 동사, 형용사 중에서 의미가 별로 없는 단어들을 불용어 사전에 넣어 제거  
<br>

### 5.4 군집분석

<img src="https://github.com/user-attachments/assets/932aea59-c0db-4c0d-9a85-5fbfaa5cacf9" width="900"/>

- 파이썬의 TfidfVectorizer함수를 이용해 전처리된 관광지별 블로그글을 가지고 **tf-idf행렬**을 생성
- 생성된 tf-idf행렬을 이용해 **K-Means 군집분석**을 수행  
<br> <br>

<img src="https://github.com/user-attachments/assets/1aa8daae-5355-4dab-86ab-19a49deccad1" width="800"/>

- 적절한 군집 수를 결정하기  위해 **Elbow Method**로 그래프 생성
- 군집 간 거리의 합을 나타내는 inertia 값이 급격히 떨어지며 꺾이는 5를 군집의 숫자로 결정  
<br> <br>

<img src="https://github.com/user-attachments/assets/03752f2d-4293-44cf-9690-6a59d353e580" width="800"/>

- 결정된 군집의 수를 토대로 K=5인 K-Means 군집화를 수행
- tf-idf행렬을 PCA(주성분분석)을 통해 2차원과 3차원으로 차원축소를 한 후 군집들을 시각화  
<br> <br>

**각 군집에 포함된 관광지 확인**

<img src="https://github.com/user-attachments/assets/d410b518-ea51-46ec-a2c7-984febe24cec" width="1000"/>  

<br> <br>

**각 군집의 속성을 파악하기 위해 군집별로 빈도수가 많은 단어들 확인**

<img src="https://github.com/user-attachments/assets/cf914dd9-3a2b-4bbf-aff5-04c7983ffc5c" width="1000"/>  

<br> <br>

**군집별 빈도수가 높은 단어를 기준으로 군집의 이름 결정**

<img src="https://github.com/user-attachments/assets/4eddee2c-df07-44b7-b05e-dd592b2f4a2b" width="1000"/>  

<br>

### 5.5 관광지 추천 시스템

<img src="https://github.com/user-attachments/assets/2962c452-856f-4cda-a145-9375eacb66a3" width="1000"/> 

- 각 관광지의 위치파악 후 관광지를 중심으로 반경 100m 내의 유동인구 데이터 셀들의 합을 집계
- 계절_성별_나이_평일/휴일_시간대 별로 각 관광지별 유동인구의 median 산출 (final_1023.csv)  
<br> <br>

![Animation](https://github.com/user-attachments/assets/5083c8c9-29c3-4f9e-b810-07efce30cb6a)

- 파이썬의 Dash 라이브러리를 이용
- 사용자에게서 성별, 연령, 시간대와 방문하려는 날짜를 입력받음
- 생성한 데이터(final_1023.csv)에서 군집별로 해당 성별, 연령, 시간대, 계절, 휴일평일 여부에 유동인구가 많은 관광지 2개씩 추천해주는 대시보드 생성  
<br>

## 6. 결론

> **"대전 관광객들이 성심당 말고 다른 관광지는 방문하지 않는다!"**  
<br>

**데이터 분석 및 문제 상황**

- 대전 중구의 관광객들은 성심당 위주의 방문과 소비 이루어짐
- 중구 내의 다른 매력적인 관광지들은 인지도 낮아 방문이 적음
- 관광객들에게 충분히 노출되지 않음
<br>

**해결 방안**

-  웹크롤링과 군집화를 통한 맞춤 관광지 추천 시스템 만들기
-  웹크롤링을 통해 수집한 블로그글을 바탕으로 관광지를 군집화
-  각 관광지를 테마별로 분류해 관광지의 특징을 반영한 군집 형성
<br>

**군집화 결과**
1. 연극, 공연 관련 관광지
2. 맛집, 먹거리 관련 관광지
3. 작품, 전시 관련 관광지
4. 아이와 가기 좋은 볼거리 많은 관광지
5. 역사 관련 관광지

:arrow_right: 사용자가 **성별, 연령대, 시간대, 방문날짜**를 입력하면, 이에 맞춰 적절한 관광지들을 군집별로 추천  
<br>

**활용 방안**  

<img src="https://github.com/user-attachments/assets/a574685d-defa-4539-9ce4-8850c998e04d" width="600"/> 

- 성심당 대기줄 사이에 추천시스템과 연결한 QR코드 포스터 부착
- 관광객들이 성심당 줄을 기다리면서 대전 중구 관광지에 흥미를 가질 수 있도록 함  
<br>

**기대 효과**

- 성심당에 방문한 관광객들을 주변의 다른 관광지도 같이 방문하게 하면서 중구 전역에 걸친 관광지의 활성화 효과 기대
- 개인화된 맞춤 추천을 통해 사용자의 방문 만족도 극대화
- 새로운 명소들을 경험하게 해 대전에 대한 긍정적인 인식을 높이고, 대전의 재방문율을 높일 수 있음
- 대전 중구가 다양한 테마의 매력을 지닌 지역으로 알려지면, 빵여행 뿐만 아니라 새롭고 다양한 이유로 여행객들이 대전을 여행지로 고려하고 선택할 가능성이 높아질 것  
<br>

## 7. 참고자료

**사용 데이터셋**  

- 유동인구 데이터
- 카드데이터 (cell id별, 지역별 업종 유입고객)
- 업종코드(통계청 표준산업분류10차)
- 빈격자(500).shp (국토지리정보원)
- 도로명주소 제공 행정구역 shp파일 (emd.shp)  
  (출처: http://www.gisdeveloper.co.kr/?p=2332)
- 한국관광 데이터랩 대전 관광지 데이터셋 (대전_관광지.csv)  
  (출처: https://datalab.visitkorea.or.kr/datalab/portal)  
<br>

**라이브러리**

<img src="https://github.com/user-attachments/assets/774e618f-8eb2-408a-8287-5c988e24ad35" width="500"/> 
