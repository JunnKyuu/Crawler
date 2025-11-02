# 뉴욕 식당 정보 및 리뷰 수집 사용 예시 (Grid-Based Pipeline)

## 빠른 시작

### 1. 테스트 실행 (첫 1개 그리드만)

```bash
python main.py --grid_file girdInfo.txt --limit 1 --max_restaurants 10 --max_reviews 20
```

이 명령은:
- 첫 번째 그리드(MN1: 트라이베카, 금융 지구)만 처리
- 그리드당 최대 10개 레스토랑 수집
- 레스토랑당 최대 20개 리뷰 수집
- 결과는 `restaurants/`와 `reviews/` 디렉토리에 자동 저장

**예상 출력:**
- `restaurants/restaurants_MN1.json` - 레스토랑 정보
- `reviews/MN1_Gramercy Tavern_reviews.json` - 각 레스토랑별 리뷰 파일
- `reviews/MN1_The Smith_reviews.json`
- ... (수집된 레스토랑 개수만큼)
- `restaurants/pipeline_log.json` - 전체 실행 로그

### 2. 전체 맨해튼 수집 (12개 그리드)

```bash
python main.py --grid_file girdInfo.txt --limit 12 --max_restaurants 30 --max_reviews 50 --headless
```

이 명령은:
- 맨해튼 12개 그리드 전체 처리 (MN1~MN12)
- 각 그리드당 최대 30개 레스토랑 수집
- 레스토랑당 최대 50개 리뷰 수집
- 백그라운드 모드로 실행 (브라우저 창 숨김)

**예상 소요 시간**: 약 2~4시간 (리뷰 수에 따라 다름)

### 3. 전체 뉴욕시 수집 (59개 그리드) - 권장하지 않음

```bash
python main.py --grid_file girdInfo.txt --max_restaurants 30 --max_reviews 50 --headless --delay 3.0
```

이 명령은:
- 뉴욕시 59개 그리드 전체 처리
- 각 그리드당 최대 30개 레스토랑 수집
- 레스토랑당 최대 50개 리뷰 수집
- 그리드 처리 사이 3초 대기

**주의**:
- 예상 소요 시간: 12~24시간
- API 비용이 많이 발생할 수 있습니다
- **팀원별 작업 분할을 권장합니다** (아래 참조)

## 팀원별 작업 분할 (권장)

59개 그리드를 3명이 나눠서 동시에 처리하는 방법:

### 팀원 1 - 그리드 0~19 (맨해튼 전체 + 브롱스 일부)

```bash
python main.py --grid_file girdInfo.txt --start_from 0 --limit 20 --max_restaurants 30 --max_reviews 50 --headless --delay 2.0
```

**처리 그리드**: MN1~MN12 (맨해튼 12개) + BX1~BX8 (브롱스 8개)
**예상 소요 시간**: 4~6시간

### 팀원 2 - 그리드 20~39 (브롱스 일부 + 브루클린 일부)

```bash
python main.py --grid_file girdInfo.txt --start_from 20 --limit 20 --max_restaurants 30 --max_reviews 50 --headless --delay 2.0
```

**처리 그리드**: BX9~BX12 (브롱스 4개) + BK1~BK16 (브루클린 16개)
**예상 소요 시간**: 4~6시간

### 팀원 3 - 그리드 40~58 (브루클린 일부 + 퀸즈 + 스태튼 아일랜드)

```bash
python main.py --grid_file girdInfo.txt --start_from 40 --limit 19 --max_restaurants 30 --max_reviews 50 --headless --delay 2.0
```

**처리 그리드**: BK17~BK18 (브루클린 2개) + QN1~QN14 (퀸즈 14개) + SI1~SI3 (스태튼 아일랜드 3개)
**예상 소요 시간**: 4~6시간

### 작업 분할 장점

- 전체 소요 시간: 24시간 → **6시간으로 단축**
- 한 팀원의 작업이 실패해도 다른 팀원 데이터는 안전
- 각자 다른 컴퓨터에서 동시 실행 가능

## 출력 예시

### 콘솔 출력 예시

```
================================================================================
  Google Maps 그리드 기반 식당 정보 및 리뷰 수집 파이프라인
================================================================================

그리드 정보 파싱 중: girdInfo.txt
✓ 총 59개 그리드 발견

설정:
  그리드 파일: girdInfo.txt
  처리할 그리드: 1개 (전체 59개 중 0~0)
  그리드당 최대 레스토랑: 10개
  레스토랑당 최대 리뷰: 20개
  헤드리스 모드: 아니오
  API 요청 간 대기 시간: 2.0초

################################################################################
Progress: 1/1 (100%)
Grid: [MN1] 트라이베카, 금융 지구 (Tribeca, Financial District)
################################################################################

[Step 1/2] 레스토랑 정보 수집

================================================================================
[MN1] 트라이베카, 금융 지구 (Tribeca, Financial District)
Query: restaurants in Tribeca, Financial District New York
Output: restaurants/restaurants_MN1.json
================================================================================

Query: restaurants in Tribeca, Financial District New York  |  Max: 10
총 10개 장소를 가져와 'restaurants/restaurants_MN1.json' 파일에 저장했습니다.

✓ 레스토랑 정보 수집 [MN1] 완료
   수집된 레스토랑: 10개

[Step 2/2] 리뷰 수집

================================================================================
구글 맵 리뷰 크롤러 시작
================================================================================
입력 파일: restaurants/restaurants_MN1.json
출력 디렉토리: reviews
최대 리뷰 개수: 20
헤드리스 모드: 아니오
================================================================================

크롤링 시작: Gramercy Tavern
URL: https://www.google.com/maps/place/?q=place_id:ChIJxxxxxx
...
✓ 저장 완료: reviews/MN1_Gramercy Tavern_reviews.json (리뷰 20개)

크롤링 시작: The Smith
...
✓ 저장 완료: reviews/MN1_The Smith_reviews.json (리뷰 20개)

...

✓ 리뷰 수집 [MN1] 완료
   수집된 리뷰: 200개

================================================================================
  실행 결과 요약
================================================================================

처리된 그리드: 1개
  ✓ 성공: 1개
  ✗ 실패: 0개

수집 통계:
  레스토랑: 10개
  리뷰: 200개

소요 시간: 450.5초 (7.5분)

출력 디렉토리:
  레스토랑: restaurants/
  리뷰: reviews/

로그 저장: restaurants/pipeline_log.json

================================================================================
  모든 작업이 성공적으로 완료되었습니다!
================================================================================
```

### pipeline_log.json 예시

```json
{
    "timestamp": "2025-01-15 14:30:45",
    "total_grids": 1,
    "success_count": 1,
    "total_restaurants": 10,
    "total_reviews": 200,
    "elapsed_seconds": 450.5,
    "results": [
        {
            "code": "MN1",
            "restaurants_success": true,
            "reviews_success": true,
            "restaurant_count": 10,
            "review_count": 200
        }
    ]
}
```

## 중단 및 재개

### 중단된 경우

작업이 중간에 중단되어도 걱정하지 마세요! 이미 수집된 데이터는 모두 저장되어 있습니다.

**예시**: 팀원 1이 그리드 0~19를 처리 중 5번째 그리드에서 중단된 경우

**확인**:
```bash
ls restaurants/  # restaurants_MN1.json ~ restaurants_BX5.json 까지 존재
ls reviews/      # 해당 그리드들의 리뷰 파일들 존재
```

**재개**:
```bash
# 6번째 그리드부터 재개 (인덱스 5부터)
python main.py --grid_file girdInfo.txt --start_from 5 --limit 15 --max_restaurants 30 --max_reviews 50 --headless
```

**중요**: `--start_from 5`는 6번째 그리드부터 시작 (인덱스는 0부터 시작)

## 팁

1. **테스트 먼저**: `--limit 1` 옵션으로 먼저 테스트해보세요
2. **팀원별 분할**: 59개 그리드 전체 대신 팀원별로 나눠서 처리하세요
3. **API 비용 주의**: Google Places API는 유료입니다. 비용을 미리 계산하세요
4. **대기 시간 조정**: API 제한에 걸리면 `--delay` 값을 늘리세요 (2초 → 3~5초)
5. **헤드리스 모드**: 장시간 실행 시 `--headless` 옵션으로 브라우저 숨김 권장
6. **로그 확인**: 문제 발생 시 `restaurants/pipeline_log.json` 확인
7. **중간 저장 확인**: 작업 중 `restaurants/`, `reviews/` 디렉토리를 주기적으로 확인하여 파일이 잘 생성되는지 체크
8. **저장 공간**: 리뷰가 많을 경우 수 GB 필요할 수 있으니 저장 공간 미리 확인
