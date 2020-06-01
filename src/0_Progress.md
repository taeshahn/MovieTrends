# **Crawling Progress**

### **1. Trends**
1. Naver Trends
    * 공식 API 제공 (ID/SECRET 발급)
    * 각 영화마다 키워드 설정하여 트랜드 조회 가능
    * 기간별, 성별, 연령별, 채널별 조회 가능
    * 동시 조회 검색어 5개로 제한
    * 정확한 검색 요청 수가 아닌 MaxAbsScaling된 Index 형태로 값 제공
2. Google Trends

### **2. Review**
1. 네이버 영화
    * 수집가능항목
        * 닉네임
        * 리뷰
        * 평점
        * 리뷰에 대한 like/dislike 수
        * 작성일자
        * 영화 시청 여부
2. Twitter
    * 공식 API `Tweepy` 제공
        * 무료 requests 검색 기간 제한 (최근 7일)
        * 유료 $149 for 500 requests / 99$ for 100 requests
    * 비공식 Library `GetOldTweets3` 이용
    * 수집 가능 항목
        * 작성 일자/시각
        * 작성자 ID
        * 트윗 내용
        * 트윗 주소
        * retweet count
        * favorite count
        * (+ user_created, user_tweets, user_followings, user_followers)
    * 특이사항: 수행 시간 매우 오래 걸림
3. Instagram
    - 특정 검색어로 검색 시, 조회된 HashTag 수집 가능
    - 가능 여부 확인, 구현 시 에러 (chromedriver issue)
4. Facebook
    - 미확인

### **3. Like / Dislike**
1. 네이버 영화
    - 댓글에 대한 Like / Dislike
2. Twitter
    -  트윗에 대한 Like / Retweet
