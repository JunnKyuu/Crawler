# Google Maps Restaurant & Review Crawler (Grid-Based)

뉴욕시 59개 커뮤니티 그리드별로 식당 정보와 리뷰를 자동 수집하는 크롤러입니다.

## 목차

- [주요 특징](#주요-특징)
- [프로젝트 구조](#프로젝트-구조)
- [설치](#설치)
- [사용법](#사용법)
  - [빠른 시작](#빠른-시작)
  - [팀원별 작업 분할](#팀원별-작업-분할)
  - [개별 스크립트 실행](#개별-스크립트-실행)
- [파일 형식](#파일-형식)
- [주의사항](#주의사항)

## 주요 특징

### 그리드 기반 자동화 파이프라인 (main.py)
- **gridInfo.txt 기반 자동 수집**: 뉴욕시 59개 커뮤니티 그리드별 자동 처리
- **그리드별 완전 수집**: 각 그리드마다 레스토랑 정보 수집 → 즉시 리뷰 수집
- **팀원별 작업 분할**: `--start_from`, `--limit` 옵션으로 작업을 나눠서 병렬 실행 가능
- **중간 저장**: 레스토랑 수집 완료, 식당별 리뷰 수집 완료 시마다 즉시 저장 (오류 발생 시에도 데이터 보존)
- **개별 파일 저장**: 각 레스토랑의 리뷰를 개별 JSON 파일로 저장
- **자동 로그 기록**: 전체 실행 결과를 pipeline_log.json에 자동 저장

### 식당 정보 수집 (getRestaurantsInfo.py)
- Google Places API를 사용하여 식당 정보 수집
- 이름, 주소, place_id, 평점, 리뷰 수, 전화번호 수집
- 검색 쿼리 기반 식당 검색
- 그리드별 자동 쿼리 생성 및 저장

### 리뷰 수집 (getReviews.py)
- Selenium을 이용한 동적 크롤링
- 리뷰 별점, 작성일자, 텍스트, 언어 정보 수집
- 자동으로 "자세히", "원문보기" 버튼 클릭하여 전체 리뷰 내용 수집
- 스크롤을 통한 동적 로딩으로 모든 리뷰 수집
- 최신순 정렬로 리뷰 수집
- **식당별 개별 파일 저장**: {그리드코드}_{식당명}_reviews.json 형식
- 오류 발생 시에도 오류 정보 저장

## 프로젝트 구조

```
Crawler/
├── main.py                           # 그리드 기반 전체 파이프라인 (메인 스크립트)
├── getRestaurantsInfo.py             # 식당 정보 수집 유틸리티
├── getReviews.py                     # 리뷰 수집 유틸리티
├── collect_restaurants_by_grid.py    # (사용 안 함, main.py가 대체)
├── config.py                         # API 키 설정
├── requirements.txt                  # 필수 패키지 목록
├── .env                              # 환경 변수 (API 키 저장)
├── girdInfo.txt                      # 뉴욕시 59개 그리드 정보 (입력)
│
├── restaurants/                      # 그리드별 레스토랑 정보 (출력)
│   ├── restaurants_MN1.json          # 맨해튼 1지구 레스토랑 정보
│   ├── restaurants_MN2.json          # 맨해튼 2지구 레스토랑 정보
│   ├── restaurants_BX1.json          # 브롱스 1지구 레스토랑 정보
│   ├── ...                           # (총 59개 파일)
│   └── pipeline_log.json             # 전체 실행 로그
│
└── reviews/                          # 레스토랑별 리뷰 정보 (출력)
    ├── MN1_Gramercy Tavern_reviews.json
    ├── MN1_The Smith_reviews.json
    ├── MN2_Balthazar_reviews.json
    └── ...                           # (레스토랑 개수만큼 파일 생성)
```

## 설치

### 1. 필수 패키지 설치

```bash
pip install -r requirements.txt
```

### 2. Google Maps API 키 설정

식당 정보 수집을 위해 Google Maps API 키가 필요합니다.

1. [Google Cloud Console](https://console.cloud.google.com/)에서 프로젝트 생성
2. Places API 활성화
3. API 키 생성
4. 프로젝트 루트 디렉토리에 `.env` 파일 생성:

```env
GOOGLE_MAPS_API_KEY=your_api_key_here
```

### 3. Chrome WebDriver 설치

Selenium은 Chrome 브라우저를 제어하기 위해 ChromeDriver가 필요합니다.

**방법 1: 자동 설치 (권장)**
- Selenium 4.6 이상에서는 자동으로 ChromeDriver를 관리합니다.
- 별도 설치가 필요 없습니다.

**방법 2: 수동 설치**
1. [ChromeDriver 다운로드](https://chromedriver.chromium.org/downloads)
2. Chrome 브라우저 버전에 맞는 ChromeDriver 다운로드
3. PATH에 추가하거나 프로젝트 폴더에 저장

## 사용법

### 빠른 시작

#### 테스트 실행 (첫 1개 그리드만)

```bash
python main.py --grid_file girdInfo.txt --limit 1 --max_restaurants 10 --max_reviews 20
```

이 명령은:
- 첫 번째 그리드(MN1)만 처리
- 그리드당 최대 10개 레스토랑 수집
- 레스토랑당 최대 20개 리뷰 수집
- 결과는 `restaurants/`와 `reviews/` 디렉토리에 저장

#### 전체 그리드 실행 (59개 그리드)

```bash
python main.py --grid_file girdInfo.txt --max_restaurants 30 --max_reviews 50 --headless
```

이 명령은:
- 뉴욕시 59개 그리드 전체 처리
- 그리드당 최대 30개 레스토랑 수집
- 레스토랑당 최대 50개 리뷰 수집
- 백그라운드 모드로 실행 (브라우저 창 숨김)

### 팀원별 작업 분할

59개 그리드를 3명의 팀원이 나눠서 동시에 처리하는 예시:

#### 팀원 1: 그리드 0~19 (20개)

```bash
python main.py --grid_file girdInfo.txt --start_from 0 --limit 20 --max_restaurants 30 --max_reviews 50 --headless
```

#### 팀원 2: 그리드 20~39 (20개)

```bash
python main.py --grid_file girdInfo.txt --start_from 20 --limit 20 --max_restaurants 30 --max_reviews 50 --headless
```

#### 팀원 3: 그리드 40~58 (19개)

```bash
python main.py --grid_file girdInfo.txt --start_from 40 --limit 19 --max_restaurants 30 --max_reviews 50 --headless
```

### main.py 명령어 파라미터

| 파라미터 | 설명 | 기본값 | 예시 |
|---------|------|--------|------|
| `--grid_file` | Grid 정보 파일 경로 | girdInfo.txt | `--grid_file my_grid.txt` |
| `--start_from` | 시작 그리드 인덱스 | 0 | `--start_from 20` |
| `--limit` | 처리할 그리드 수 | 전체 | `--limit 20` |
| `--max_restaurants` | 그리드당 최대 레스토랑 수 | 30 | `--max_restaurants 50` |
| `--max_reviews` | 레스토랑당 최대 리뷰 수 | 제한 없음 | `--max_reviews 100` |
| `--headless` | 백그라운드 실행 | False | `--headless` |
| `--restaurants_dir` | 레스토랑 정보 출력 디렉토리 | restaurants | `--restaurants_dir ./data/rest` |
| `--reviews_dir` | 리뷰 출력 디렉토리 | reviews | `--reviews_dir ./data/rev` |
| `--delay` | 그리드 처리 간 대기 시간(초) | 2.0 | `--delay 3.0` |

### 개별 스크립트 실행

일반적으로는 main.py를 사용하는 것을 권장하지만, 필요한 경우 개별 스크립트를 직접 실행할 수도 있습니다.

#### 2-1. 식당 정보 수집 (getRestaurantsInfo.py)

```bash
# 기본 사용
python getRestaurantsInfo.py --query "restaurants in Seoul"

# 최대 결과 수 지정
python getRestaurantsInfo.py --query "sushi in Gangnam" --max_results 50

# 출력 파일 지정
python getRestaurantsInfo.py --query "korean food" --output my_restaurants.json
```

**파라미터:**
- `--query`: 검색 쿼리 (필수)
- `--max_results`: 최대 식당 수 (기본값: 30)
- `--output`: 출력 파일 경로 (기본값: restaurants.json)

#### 2-2. 리뷰 수집 (getReviews.py)

```bash
# 특정 그리드의 레스토랑 리뷰 수집
python getReviews.py --input restaurants/restaurants_MN1.json --output_dir reviews --max_reviews 50 --headless

# 백그라운드 실행
python getReviews.py --input restaurants/restaurants_BX1.json --output_dir reviews --headless
```

**파라미터:**
- `--input`: 입력 파일 경로 (기본값: restaurants.json)
- `--output_dir`: 출력 디렉토리 (기본값: reviews)
- `--max_reviews`: 레스토랑당 최대 리뷰 수 (기본값: 제한 없음)
- `--headless`: 백그라운드 실행

**참고**: getReviews.py는 각 레스토랑의 리뷰를 `{그리드코드}_{레스토랑명}_reviews.json` 형식의 개별 파일로 저장합니다.

## 파일 형식

### 입력 파일: girdInfo.txt

뉴욕시 59개 커뮤니티 그리드 정보:

```
MN 1,"트라이베카, 금융 지구 (Tribeca, Financial District)"
MN 2,"그리니치 빌리지, 소호, 차이나타운 (Greenwich Village, SoHo, Chinatown)"
BX 1,"모트 헤이븐, 멜로즈 (Mott Haven, Melrose)"
...
```

형식: `지구코드,"한국어 지역명 (영문 지역명)"`

### 출력 파일 1: restaurants/{CODE}.json

각 그리드별 레스토랑 정보 (예: `restaurants/restaurants_MN1.json`):

```json
[
    {
        "name": "Gramercy Tavern",
        "address": "42 E 20th St, New York, NY 10003",
        "place_id": "ChIJxxxxxx",
        "rating": 4.6,
        "user_ratings_total": 3245,
        "phone_number": "(212) 477-0777"
    },
    ...
]
```

### 출력 파일 2: reviews/{CODE}_{name}_reviews.json

각 레스토랑별 리뷰 정보 (예: `reviews/MN1_Gramercy Tavern_reviews.json`):

```json
{
    "name": "Gramercy Tavern",
    "place_id": "ChIJxxxxxx",
    "grid": "MN1",
    "address": "42 E 20th St, New York, NY 10003",
    "rating": 4.6,
    "user_ratings_total": 3245,
    "phone_number": "(212) 477-0777",
    "reviews_count": 50,
    "reviews": [
        {
            "rating": 5,
            "date": "3주 전",
            "text": "Amazing food and service...",
            "language": "en"
        },
        {
            "rating": 4,
            "date": "1개월 전",
            "text": "정말 맛있었어요...",
            "language": "ko"
        }
    ]
}
```

### 출력 파일 3: restaurants/pipeline_log.json

전체 파이프라인 실행 로그:

```json
{
    "timestamp": "2025-01-15 14:30:45",
    "total_grids": 20,
    "success_count": 19,
    "total_restaurants": 570,
    "total_reviews": 28500,
    "elapsed_seconds": 7200.5,
    "results": [
        {
            "code": "MN1",
            "restaurants_success": true,
            "reviews_success": true,
            "restaurant_count": 30,
            "review_count": 1500
        }
    ]
}
```

## 작동 방식

### 그리드 기반 파이프라인 (main.py)

1. **gridInfo 파싱**
   - `girdInfo.txt` 파일을 읽어 59개 그리드 정보 파싱
   - `--start_from`, `--limit` 옵션에 따라 처리할 그리드 범위 결정

2. **각 그리드별 처리 (순차적)**
   - **Step 1: 레스토랑 정보 수집**
     - Google Places API로 그리드별 검색 쿼리 생성 (예: "restaurants in Tribeca New York")
     - 그리드별 레스토랑 정보를 `restaurants/restaurants_{CODE}.json`에 **즉시 저장**

   - **Step 2: 리뷰 수집**
     - 방금 수집한 레스토랑 파일을 읽어 각 레스토랑의 리뷰 크롤링
     - Selenium으로 브라우저 자동화:
       - 리뷰 탭 클릭 → 최신순 정렬 → 스크롤 → "자세히", "원문보기" 클릭
     - 각 레스토랑의 리뷰를 `reviews/{CODE}_{name}_reviews.json`에 **즉시 저장**

3. **오류 처리**
   - 레스토랑 수집 실패 시: 해당 그리드의 리뷰 수집 건너뜀
   - 리뷰 수집 실패 시: 오류 정보를 JSON 파일로 저장하고 다음 레스토랑 진행

4. **최종 요약**
   - 처리된 그리드 수, 성공/실패 통계
   - 총 레스토랑 수, 총 리뷰 수
   - 실행 로그를 `restaurants/pipeline_log.json`에 저장

## 주의사항

### 비용 및 제한
- **Google Maps API 비용**: Places API 사용량에 따라 비용이 발생합니다. [가격 정책](https://mapsplatform.google.com/pricing/) 확인 필요
- **API 요청 제한**: Google Places API는 요청 횟수 제한이 있습니다. `--delay` 옵션으로 대기 시간 조정 가능
- **수집 시간**: 59개 그리드 전체 수집 시 수 시간 소요될 수 있습니다

### 기술적 요구사항
- **안정적인 인터넷 연결**: 크롤링 중 연결이 끊기지 않도록 주의
- **Chrome 브라우저**: Chrome 브라우저가 설치되어 있어야 합니다
- **충분한 저장 공간**: 리뷰가 많을 경우 수 GB의 저장 공간 필요

### 데이터 안정성
- **중간 저장**: 레스토랑 수집 완료 시, 각 식당의 리뷰 수집 완료 시마다 즉시 저장되므로 중간에 오류가 발생해도 데이터 손실 최소화
- **오류 처리**: 오류 발생 시에도 오류 정보가 JSON 파일로 저장됩니다
- **팀원별 작업**: 여러 팀원이 동시에 다른 그리드를 처리할 수 있어 시간 절약 가능

### 크롤링 주의사항
- **크롤링 속도**: 너무 빠르면 구글에서 차단할 수 있습니다
- **헤드리스 모드**: 백그라운드 실행 시 `--headless` 옵션 사용 권장
- **대기 시간**: API 제한에 걸리면 `--delay` 값을 늘리세요 (기본 2초 → 3~5초)

## 문제 해결

### API 키 오류
```
RuntimeError: API_KEY가 설정되어 있지 않습니다.
```
- `.env` 파일이 프로젝트 루트에 있는지 확인
- `GOOGLE_MAPS_API_KEY` 값이 올바른지 확인
- API 키에 Places API가 활성화되어 있는지 확인

### ChromeDriver 오류
```
selenium.common.exceptions.WebDriverException: 'chromedriver' executable needs to be in PATH
```
- Chrome과 ChromeDriver 버전이 일치하는지 확인
- ChromeDriver가 PATH에 있는지 확인
- Selenium 4.6 이상 버전 사용 권장 (자동 ChromeDriver 관리)

### 요소를 찾을 수 없음
```
TimeoutException: Message:
```
- 인터넷 연결 확인
- 페이지 로딩 시간이 충분한지 확인
- 구글 맵 UI가 변경되었을 수 있음 (value.txt의 HTML 구조 확인)

### API 사용량 초과
```
RuntimeError: Text Search API error: OVER_QUERY_LIMIT
```
- API 사용량이 일일 한도를 초과했습니다
- Google Cloud Console에서 사용량 확인
- 필요시 결제 정보 등록 또는 한도 증가 요청

## 라이선스

MIT
