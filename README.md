# youtubeData

인하공업전문대학 빅데이터처리 프로젝트
유튜브 데이터 분석

유튜브 동영상 데이터
>데이터 필터링 (날짜, 제목, 채널명, 조회수, 태그)
>필터링 데이터 조회수 순으로 정렬
>23년 데이터 조회수 추출
>23년 데이터 중 각 채널 총 조회수 순으로 채널명 정렬

참고자료 
> 블로그 : https://coding-kindergarten.tistory.com/108

> kaggle : https://www.kaggle.com/datasets/rsrishav/youtube-trending-video-dataset

유튜브의 23년, 22년, 21년 한국 동영상의 변화와 인기있는 컨텐츠, 채널 분석
(사용 데이터 값 20-08-05 ~ 23-11-10)

# 주피터 노트북을 활용하여 유튜브 동영상의 데이터 추출
```
import pandas as pd
import numpy as numpy
import seaborn as sns
import matplotlib.pyplot as plt
import os

#20-08-05부터 23-11-10 까지의 데이터 사용
KRvideo = pd.read_csv("KR_youtube_trending_data.csv", engine="python")

#데이터 정보 요약
KRvideo.info()
```
> 결과
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/bde440b3-9c3d-4994-9f52-4cb1bf0552fa)


# 데이터 정제를 위해 데이터를 요약하였다
```
# 필요한 데이터인 날짜, 제목, 채널명, 조회수, 테그만 불러오기
df = KRvideo[["publishedAt", "title", "channelTitle", "view_count", "tags"]]

df
```
> 결과
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/e2551392-8e8c-4088-a690-12ff14de5205)



# 정제된 데이터 중 2023년 조회수 순으로 정렬했다
```
# 조회수 순으로 정렬
df_sorted = df.sort_values(by='view_count', ascending=False)

# 체널명과 영상 제목이 중복 데이터를 제거
df_sorted_latest = df_sorted.drop_duplicates(['title','channelTitle'], keep='first')

# 원하는 년도 (ex : 2023)의 데이터만 추출
df_sorted_y = df_sorted_latest['publishedAt'].str.contains("2023")
df_23 = df_sorted_latest[df_sorted_y]
df_23
```
> 결과
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/0f4c5480-e3e3-4481-9ac4-9539df45fb05)


# 영상 별로 나눠져있는 데이터를 채널명 기준으로 합쳐서 정렬했다
```
# 채널별 조회수 합계 계산
df_channel_view_sum = df_23.groupby(['channelTitle']).sum()

# 채널별 조회수 내림차순 정렬
df_channel_view = df_channel_view_sum.sort_values(by='view_count', ascending=False)

# 출력
df_channel_view
```
> 결과
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/cf4dc6f8-87b1-4a83-8988-41a6649f1003)


# 차트를 만들기 위해 상위 20개의 채널만 인덱싱하였다
```
# 데이터 중에 tags가 없는 체널을 제거
df_23_filter = df_channel_view[~df_channel_view['tags'].str.contains('None')]

# 필터링된 데이터의 상위 20개의 채널만 추출
df_23_filter_top20 = df_23_filter[:20]

# 상위 20개의 데이터 인덱싱
df_23_filter_top20_index = df_23_filter_top20.reset_index()
df_23_filter_top20_index
```
> 결과
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/60a91a95-d47f-47bf-8056-bb8c8773e43f)
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/e09f4372-8569-4448-9cd5-e2f5020b93ae)




# 차트 결과
```
# 그래프 출력 시 이상한 에러들 무시
import warnings
warnings.filterwarnings("ignore")

# 그래프 그릴 때 한글 깨짐 방지 설정
import os

# Mac OS의 경우와 그 외 OS의 경우로 나누어 설정
if os.name == 'posix':
    plt.rc("font", family="AppleGothic")

else:
    plt.rc("font", family="Malgun Gothic")

# 그래프 사이즈 설정
plt.figure(figsize=(10,10))

# seaborn 패키지로 수평막대 그래프 그리기
sns.barplot(x='view_count', y='channelTitle', data=df_23_filter_top20_index)
```
> 결과
> ![image](https://github.com/dlrkd/youtubeData/assets/35716755/c5982ec6-9b0f-4e6a-8410-2fd7d3200280)


---
고민사항
1. 데이터를 인기순(조회수)으로 정렬하게 된다면 상위권이 kpop mv 가 많아지기 때문에 유튜브 컨텐츠의 정확한 인기 컨텐츠를 알기 어렵기 때문에 조회수를 기준 이하의 데이터를 사용하려 했으나 그렇게 하면 kpop이 아닌 상위권 컨텐츠 데이터를 사용할 수 없는 문제가 발생하게된다
2. 이를 해결하기 위해 태그 데이터에서 kpop, bts, sm 과 같은 단어를 필터링하여서 다시 데이터 추출을 할 예정이다

