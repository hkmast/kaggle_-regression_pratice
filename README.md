# 🙌<u>목적</u>
- 주어진 데이터로 캘리포니아 집값 예측(회귀문제)

# ❓<u>데이터 수집</u>
- 캐글 경쟁대회 놀이터2 데이터
- fetch_california_housing() 데이터

# 🙋‍♀️<u>데이터 분석 결과</u>

|히스토그램|회귀|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931885-dbdea2bc-16ea-41b5-9c44-af6bdb86a729.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931884-0a388a77-2af8-4ab2-8dd5-f459150c6866.PNG" width="700" height="300"/>| 

|상관관계|위,경도|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931886-6325f25a-7836-4b0b-8e19-aaf6faf78906.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931887-aff4d8b7-aee6-4ec4-9ba7-1c72d52983d2.PNG" width="700" height="300"/>| 

1. 이상치 제거 항목
  - MedInc, AveRooms, AveBedrms, Population, AveOccup
2. 차원 정리
  - Latitude, Longitude 
3. 강한 상관관계
  - AveRooms & MedInc 간의 강한 양의 상관관계
    - 하나만 사용
    - 둘다 사용
  - Latitude & Longitude 간의 강한 음의 상관관계
    - 하나만 사용
    - 둘다 사용

# 👆<u>피쳐 엔지니어링</u>
## 이상치 제거
  1. 이상치 제거할 피처를 시각화
  2. 시각화 결과를 바탕으로, 이상치 제거 대상 피처마다 범위를 지정한 후 IQR(분위수) 방식을 이용하여 이상치 제거.

|이상치제거 히스토그램|이상치제거 회귀|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931878-b1047385-739b-4f0e-8f73-09855d1cbc06.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931880-84036f70-0dad-4fb1-82dd-98d001f3c347.PNG" width="700" height="300"/>| 

## 상관관계 상쇄
- MedInc & AveRooms 
- Latitude & Longitude

|df|MedInc|AveRooms|Latitude|Longitude|
|--|--|--|--|--|
|tmp1|o|x|o|x|
|tmp2|o|x|x|o|
|tmp3|x|o|o|x|
|tmp4|x|o|x|o|
|tmp_D_1|o|x|o|o|
|tmp_D_2|x|o|o|o|

- tmp 1 ~ 4 : 차원 축소 할 필요 없음
- tmp_D_1 ~ 2 : 차원축소 필요

## 위경도를 이용하여 새로운 피쳐 생성
  - 방법1: 지역 분할
  - 방법2: K-Means
  - 방법3: 차원축소(PCA, PCA을 군집화, UMAP, UMAP을 군집화
  - 방법4: 해안가와의 거리
  - 방법5: 주요 도시와의 거리
  - 방법6: 위도 경도 회전
  - 방법7: 지역들의 메타 데이터 이용(주별 인구밀도, 미국과 멕시코의 실질 부동산 가격지수)
  - 방법8: 연속형 변수의 범위를 정해서 범주화

|PCA 군집화|UMAP 군집화|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931882-e552afa0-4814-4f5e-aa4c-db4121735297.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931883-1a129ff5-296f-4669-af63-ef9ef9ce0bb7.PNG" width="700" height="300"/>| 

    
# 🛠<u>베이스라인 모델 구축 및 기본학습</u>
- 학습 시키기 전, 스케일링 함수와 점수처리 함수 생성
- 어떤 피쳐가 의미가 있는지 기본모델로 검증
## 모델 학습
1. LinearRegression
2. Lasso
3. RandomForestRegressor
## 기본모델 학습 결과, 성능 저하시키는 피쳐 정리
1. 연속형 변수의 범위를 정해서 범주화 시킨 피처.
2. 상관관계가 높은 피처를 상쇄한 피처.
3. 주별 인구밀도 피처.
4. 미국과 멕시코의 실질 부동산 가격지수 피처.

- 위의 파생피처를 생성해서 돌렸을때, 성능저하가 되었으므로,위의 피처를 제거하여서 최종 피처를 생성하였다.

# 🙋‍♂️<u>모델 성능 향상</u>
## 피처제거 방법
- 위에서 생성한 최종 피처에서, 상대적으로 피처 중요도가 낮거나, 경험적으로 학습에 방해될 만한 피처 제거.
### 제거 피처 선정
1. pca_cluster제거
2. umap_cluster제거
3. cluster_3제거
4. cluster_20제거

- 위의 항목에 있는 피처를 각각제거하여, 성능평가
- 성능평가 결과가 의미있는 예측값 간의 블렌딩

## 최적 모델 선정
- 모델은 varstack 이라는 lgbm 튜너를 사용
- https://github.com/conversis/varstack
 
# 🔎<u>결론</u>
- umap_cluster피처 제거와, cluster_3피처를 제거하여 lgbm튜너를 돌려서 나온 각각의 예측값을 블렌딩하였다.
- 최종값이 '0.55546'으로 실험 중 가장 높은 결과 값이 나왔다.

<img src="https://user-images.githubusercontent.com/67897827/219930166-6ef32a7d-6a6f-4ad9-9ee5-bf1924bf0e3c.PNG" width="700" height="300"/>