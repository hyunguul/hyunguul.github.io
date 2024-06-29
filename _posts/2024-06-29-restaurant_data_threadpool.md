---
layout: single
title: "프로젝트 때 했던 내용입니다"
---

```python
import requests
import json
import pandas as pd
import time
from selenium import webdriver 
from selenium.webdriver.chrome.service import Service 
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By 
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import re


place_data = {}

# 카카오 api 추출
def fetch_restaurant_data(place):
    REST_API_KEY = 
    combined_data = [] 

    for i in range(1, 4):
            url = "https://dapi.kakao.com/v2/local/search/keyword" 
            REST_API_KEY = 
            place_query = f'{place} 맛집'
            code = "FD6"
            page = i

            headers = {
                "Authorization": f"KakaoAK {REST_API_KEY}"
            }
            params = {'query' : place_query, 
                      'category_group_code' : code,
                      'page' : page,
                      'sort' : "accuracy",
                      'size' : 15}

            response = requests.get(url, headers=headers, params=params)
            data = json.loads(response.text)

            restaurant_data_df = pd.DataFrame.from_dict(data["documents"])
            columns_to_drop = ["x", "y", "category_group_code", "category_group_name", "distance", "id"]
            restaurant_data_df = restaurant_data_df.drop(columns=columns_to_drop)
            restaurant_data_df = restaurant_data_df[['place_name', 'category_name','road_address_name', 'address_name', 'phone','place_url']]

            combined_data.append(restaurant_data_df)  

    place_data[place] = pd.concat(combined_data, ignore_index=True)
    place_data[place]['category_name'] = place_data[place]['category_name'].apply(lambda category: category.split(">")[-1].strip())
  

 # 크롤링 데이터 추출        
def fetch_additional_data(place):   
    combined_data_rating = []
    combined_data_review = []
    combined_data_blog_review = []
    combined_data_homepage_link = []
    combined_data_image_url = []

    with webdriver.Chrome(service=Service(ChromeDriverManager().install())) as driver:
        for i in range(0,45):
            driver.get(place_data[place]['place_url'][i])
            driver.implicitly_wait(5)

            try:
                rating = driver.find_element(By.CLASS_NAME, 'num_rate').text
            except:
                rating = ''

            try:
                review_count = driver.find_element(By.XPATH, '//*[@id="mArticle"]/div[7]/strong[1]/span').text
            except:
                try:
                    review_count = driver.find_element(By.XPATH, '//*[@id="mArticle"]/div[8]/strong[1]/span').text
                except:
                    try:
                        review_count = driver.find_element(By.XPATH, '//*[@id="mArticle"]/div[6]/strong[1]/span').text
                    except:
                        review_count = ''

            try:
                blog_review_count = driver.find_element(By.XPATH, '//*[@id="mArticle"]/div[1]/div[1]/div[2]/div/div/a[2]/span').text
            except:
                try:
                    blog_review_count = driver.find_element(By.XPATH, '//*[@id="mArticle"]/div[1]/div[1]/div[2]/div/div/a/span').text 
                except:
                    blog_review_count = ''


            try:
                homepage_link = driver.find_element(By.XPATH, '//*[@id="mArticle"]/div[1]/div[2]/div[3]/div/div/a').get_attribute('href')
            except:
                homepage_link = ''

            try:
                image_url = driver.find_element(By.CLASS_NAME, 'bg_present').get_attribute('style')
                url_pattern = r'url\("//([^"]+)"\)'
                match = re.search(url_pattern, image_url)
                if match:
                    partial_url = match.group(1)
                    complete_url = f"https://{partial_url}"
                else:
                    complete_url = ""
            except:
                complete_url = ""

            rating_without_dot = rating.replace("점", "")   
            combined_data_rating.append(rating_without_dot)
            combined_data_review.append(review_count)
            combined_data_blog_review.append(blog_review_count)
            combined_data_homepage_link.append(homepage_link)
            combined_data_image_url.append(complete_url) 

    place_data[place]['rating'] = combined_data_rating
    place_data[place]['review_count'] = combined_data_review
    place_data[place]['blog_review_count'] = combined_data_blog_review
    place_data[place]['homepage_link'] = combined_data_homepage_link
    place_data[place]['image_url'] = combined_data_image_url
    place_data[place] = place_data[place][['place_name', 'category_name', 'rating', 'review_count', 'blog_review_count', 'road_address_name', 'address_name', 'phone','place_url', 'homepage_link', 'image_url']]   


def concat_and_drop_duplicates():
    # 데이터프레임 합치기, 새로운 키워드로 딕셔너리에 저장, 중복 행 제거 
    place_data['동대문 관광특구'] = (pd.concat([place_data['광희동'], place_data['신당동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['명동 관광특구'] = (pd.concat([place_data['소공동'], place_data['회현동'], place_data['명동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['이태원 관광특구'] = (pd.concat([place_data['이태원1동'], place_data['한남동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['잠실 관광특구'] = (pd.concat([place_data['잠실3동'], place_data['잠실6동'], place_data['방이2동'], place_data['오륜동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['종로·청계 관광특구'] = (pd.concat([place_data['종로1.2.3.4가동'], place_data['종로5.6가동'], place_data['창신1동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['홍대 관광특구'] = (pd.concat([place_data['서교동'], place_data['동교동'], place_data['합정동'], place_data['상수동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['광화문·덕수궁'] = (pd.concat([place_data['광화문'], place_data['덕수궁']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['창덕궁·종묘'] = (pd.concat([place_data['창덕궁'], place_data['종묘']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['서울식물원·마곡나루역'] = (pd.concat([place_data['서울식물원'], place_data['마곡나루역']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['신논현역·논현역'] = (pd.concat([place_data['신논현역'], place_data['논현역']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['신촌·이대역'] = (pd.concat([place_data['신촌'], place_data['이대역']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['오목교역·목동운동장'] = (pd.concat([place_data['오목교역'], place_data['목동']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['낙산공원·이화마을'] = (pd.concat([place_data['낙산공원'], place_data['이화마을']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['덕수궁길·정동길'] = (pd.concat([place_data['덕수궁돌담길'], place_data['정동길']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['해방촌·경리단길'] = (pd.concat([place_data['해방촌'], place_data['경리단길']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['국립중앙박물관·용산가족공원'] = (pd.concat([place_data['국립중앙박물관'], place_data['용산가족공원']], ignore_index=True)).drop_duplicates(subset=['place_name'])
    place_data['서리풀공원·몽마르뜨공원'] = (pd.concat([place_data['서리풀공원'], place_data['몽마르뜨공원']], ignore_index=True)).drop_duplicates(subset=['place_name'])

    # 기존 플레이스 데이터프레임 삭제
    to_remove = [
        '광희동', '신당동', '소공동', '회현동', '명동', '이태원1동', '한남동',
        '잠실3동', '잠실6동', '방이2동', '오륜동', '종로1.2.3.4가동',
        '종로5.6가동', '창신1동', '서교동', '동교동', '합정동', '상수동',
        '광화문', '덕수궁', '창덕궁', '종묘', '서울식물원', '마곡나루역',
        '신논현역', '논현역', '신촌', '이대역', '오목교역', '목동',
        '낙산공원', '이화마을', '덕수궁돌담길', '정동길', '해방촌',
        '경리단길', '국립중앙박물관', '용산가족공원', '서리풀공원', '몽마르뜨공원']
    for place in to_remove:
        del place_data[place]

    # 저장할 장소명으로 이름 변경
    place_data['강남 MICE 관광특구'] = place_data.pop('삼성1동')
    place_data['서울 암사동 유적'] = place_data.pop('암사동')
    place_data['북한산우이역'] = place_data.pop('우이동')
    place_data['4·19 카페거리'] = place_data.pop('419로')
    place_data['광장(전통)시장'] = place_data.pop('광장시장')
    place_data['방배역 먹자골목'] = place_data.pop('방배역')
    place_data['성수카페거리'] = place_data.pop('성수동2가')
    place_data['수유리 먹자골목'] = place_data.pop('수유역')
    place_data['쌍문동 맛집거리'] = place_data.pop('쌍문동')
    place_data['창동 신경제 중심지'] = place_data.pop('창동')
    place_data['청량리 제기동 일대 전통시장'] = place_data.pop('제기동 시장')
    place_data['DDP(동대문디자인플라자)'] = place_data.pop('동대문디자인플라자')
    place_data['DMC(디지털미디어시티)'] = place_data.pop('디지털미디어시티')


def main():
    # 수정전 장소 리스트
    initial_place_list = [
    '이태원1동', '한남동', '소공동', '회현동', '명동', '광희동', '신당동', '종로1.2.3.4가동', 
    '종로5.6가동', '창신1동', '잠실3동', '잠실6동', '방이2동', '오륜동', '삼성1동', '서교동', 
    '동교동', '합정동', '상수동', '경복궁', '광화문', '덕수궁', '보신각', '암사동', '창덕궁', '종묘',
    '가산디지털단지역', '강남역', '건대입구역', '고덕역', '고속터미널역', '교대역', '구로디지털단지역',
    '구로역', '군자역', '남구로역', '대림역', '동대문역', '뚝섬역', '미아사거리역', '발산역', '우이동',
    '사당역', '삼각지역', '서울대입구역', '서울식물원', '마곡나루역', '서울역', '선릉역', '성신여대입구역',
    '수유역', '신논현역', '논현역', '신도림역', '신림역', '신촌', '이대역', '양재역', '역삼역', 
    '연신내역', '오목교역', '목동', '낙산공원', '이화마을', '덕수궁돌담길', '정동길', '해방촌',
    '경리단길', '국립중앙박물관', '용산가족공원', '서리풀공원', '몽마르뜨공원', '419로', '가락시장',
    '가로수길', '광장시장', '김포공항', '낙산공원', '이화마을', '노량진', '덕수궁돌담길', '정동길',
    '방배역', '북촌한옥마을', '서촌', '성수동2가', '수유역', '쌍문동', '압구정로데오거리', '여의도',
    '연남동', '영등포 타임스퀘어', '외대앞', '한강로2가', '이태원 앤틱가구거리', '인사동', '익선동',
    '창동', '청담동 명품거리', '제기동 시장', '해방촌', '경리단길', '동대문디자인플라자', '디지털미디어시티',
    '강서한강공원', '고척돔', '광나루한강공원', '광화문광장', '국립중앙박물관', '용산가족공원', '난지한강공원',
    '남산공원', '노들섬', '뚝섬한강공원', '망원한강공원', '반포한강공원', '북서울꿈의숲', '불광천', '서리풀공원',
    '몽마르뜨공원', '서울광장', '서울대공원', '서울숲공원', '아차산', '양화한강공원', '어린이대공원',
    '여의도한강공원', '월드컵공원', '응봉산', '이촌한강공원', '잠실종합운동장', '잠실한강공원', '잠원한강공원',
    '청계산', '청와대'
    ]
    for place in initial_place_list:
        fetch_restaurant_data(place)
        fetch_additional_data(place)
    concat_and_drop_duplicates() 
    
    # 최종 저장할 장소 리스트
    final_place_list = [
    '강남 MICE 관광특구', '동대문 관광특구', '명동 관광특구', '이태원 관광특구', '잠실 관광특구', '종로·청계 관광특구',
    '홍대 관광특구', '경복궁', '광화문·덕수궁', '보신각', '서울 암사동 유적', '창덕궁·종묘', '가산디지털단지역',
    '강남역', '건대입구역', '고덕역', '고속터미널역', '교대역', '구로디지털단지역', '구로역', '군자역', '남구로역',
    '대림역', '동대문역', '뚝섬역', '미아사거리역', '발산역', '북한산우이역', '사당역', '삼각지역', '서울대입구역',
    '서울식물원·마곡나루역', '서울역', '선릉역', '성신여대입구역', '수유역', '신논현역·논현역', '신도림역', '신림역',
    '신촌·이대역', '양재역', '역삼역', '연신내역', '오목교역·목동운동장', '왕십리역', '용산역', '이태원역',
    '장지역', '장한평역', '천호역', '총신대입구(이수)역', '충정로역', '합정역', '혜화역', '홍대입구역(2호선)',
    '회기역', '4·19 카페거리', '가락시장', '가로수길', '광장(전통)시장', '김포공항', '낙산공원·이화마을', '노량진',
    '덕수궁길·정동길', '방배역 먹자골목', '북촌한옥마을', '서촌', '성수카페거리', '수유리 먹자골목', '쌍문동 맛집거리',
    '압구정로데오거리', '여의도', '연남동', '영등포 타임스퀘어', '외대앞', '용리단길', '이태원 앤틱가구거리',
    '인사동·익선동', '창동 신경제 중심지', '청담동 명품거리', '청량리 제기동 일대 전통시장', '해방촌·경리단길',
    'DDP(동대문디자인플라자)', 'DMC(디지털미디어시티)', '강서한강공원', '고척돔', '광나루한강공원', '광화문광장',
    '국립중앙박물관·용산가족공원', '난지한강공원', '남산공원', '노들섬', '뚝섬한강공원', '망원한강공원', '반포한강공원',
    '북서울꿈의숲', '불광천', '서리풀공원·몽마르뜨공원', '서울광장', '서울대공원', '서울숲공원', '아차산',
    '양화한강공원', '어린이대공원', '여의도한강공원', '월드컵공원', '응봉산', '이촌한강공원', '잠실종합운동장',
    '잠실한강공원', '잠원한강공원', '청계산', '청와대'
    ]  
    for place in final_place_list:
        csv_filename = f"{place}_restaurant_data.csv"
        place_data[place].to_csv(csv_filename, index=False)
    
if __name__ == "__main__":
    main()
```


```python
with ThreadPoolExecutor(max_workers=50) as executor:  # max_workers는 동시에 호출할 Lambda의 최대 개수
        for success_result, failure_result in executor.map(extract_comment_from_lambda, urls, [lambda_hook]*len(urls)):
            try:
                if success_result:
                    results.append(success_result[0])  # 첫 번째 요소만 추가
                if failure_result:  # 실패한 경우에는 failed_urls에 추가
                    failed_urls.append(failure_result)
            except Exception as e:
                print(f"Error while processing results: {e}")
                continue
```
