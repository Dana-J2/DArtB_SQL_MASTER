# SQL_MASTER 5주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_5th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_5th_TIL

### 6장 웹사이트에서의 행동을 파악하는 데이터 추출하기
#### 1. 사이트 전체의 특징/경향 찾기
#### 2. 사이트 내의 사용자 행동 파악하기
#### 3. 입력 양식 최적화하기 


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.52~136   | ✅         |
| 2주차 | p.138~184  | ✅         |
| 3주차 | p.186~232 | ✅         |
| 4주차 | p.233~321 | ✅         |
| 5주차 | p.324~406 | ✅         |
| 6주차 | p.408~464 | 🍽️         |
| 7주차 | p.466~566 | 🍽️         |

<br>

<!-- 여기까진 그대로 둬 주세요-->


# 📚 6장 교재 정리

## 14장 사이트 전체의 특징/경향 찾기

### 핵심 지표

| 지표 | 의미 |
| --- | --- |
| 방문자 수 | 장기간 유지되는 쿠키 기준 유니크 사용자 수 |
| 방문 횟수 | 세션 쿠키 기준 유니크 방문 수 |
| 페이지 뷰 | 페이지 출력 로그 수 |
| 1회 방문당 페이지 뷰 | 페이지 뷰 ÷ 방문 횟수 |

📌 사이트 전체 지표는 한 번에 함께 추출해두면 이후 분석에 재사용하기 편리

### URL 집계

- URL 전체 기준 집계 : 요청 매개변수까지 서로 다른 페이지로 집계될 수 있음
- 경로 기준 집계 : `?`, `#` 이후를 제거해 같은 페이지를 하나로 묶는 방식
- 페이지 의미 기준 집계 : `/list/cd`, `/list/dvd`처럼 성격이 같은 경로를 하나의 페이지 유형으로 재분류

### 유입원 분석

| 유입원 | 대표 판정 기준 |
| --- | --- |
| 광고 · 제휴 | `utm_source`, `utm_medium` 등 URL 매개변수 |
| 검색 엔진 | referrer 도메인 |
| 소셜 미디어 | referrer 도메인 |
| 기타 사이트 | 위 조건에 해당하지 않는 외부 referrer |

- 유입원별 방문 수만 비교하는 것보다 `CVR`, 매출까지 함께 확인하는 방식이 효과적
- 유입량이 많아도 전환율이 낮다면 유입의 질이 낮을 가능성 존재

### 요일 · 시간대 분석

- 서비스 이용 패턴이 강한 요일과 시간대 파악
- 공지사항 발송, 메일 발송, 캠페인 시작 시점 결정 등에 활용 가능

---

## 15장 사이트 내의 사용자 행동 파악하기

### 입구 페이지와 출구 페이지

- 입구 페이지 : 한 세션에서 처음 접근한 페이지
- 출구 페이지 : 한 세션에서 마지막으로 접근한 페이지
- 세션 내부 순서를 다루므로 `FIRST_VALUE`, `LAST_VALUE`, `ROW_NUMBER` 등의 윈도 함수 활용

### 이탈률과 직귀율

| 지표 | 계산 기준 |
| --- | --- |
| 이탈률 | 해당 페이지에서 끝난 횟수 ÷ 해당 페이지의 페이지 뷰 |
| 직귀율 | 한 페이지만 보고 끝난 방문 수 ÷ 해당 페이지의 입구 수 |

⚠️ 완료 페이지처럼 목적을 달성한 뒤 끝나는 페이지는 이탈률이 높아도 문제로 해석하지 않음

### 성과와 페이지 가치

- 컨버전 : 구매 완료, 신청 완료 등 서비스가 목표로 하는 행동
- CVR : 방문 중 컨버전으로 이어진 비율
- 페이지 가치 : 컨버전까지 거친 페이지에 성과의 가치를 배분해 기여도를 평가하는 방식

교재의 대표적인 가치 배분 방식

| 방식 | 특징 |
| --- | --- |
| 균등 배분 | 경유 페이지에 동일한 가치 부여 |
| 첫 페이지 중심 | 최초 접점에 전체 가치 부여 |
| 마지막 페이지 중심 | 전환 직전 접점에 전체 가치 부여 |
| 전환에 가까울수록 높은 배분 | 전환 직전 페이지의 기여도를 크게 평가 |
| 최초 접점에 가까울수록 높은 배분 | 초기 유입 페이지의 기여도를 크게 평가 |

### 검색 행동 분석

- 검색 조건별 검색 수만으로 기능의 품질 판단 어려움
- 검색 결과 → 상세 페이지 이동을 `CTR`
- 상세 페이지 → 컨버전 이동을 `CVR`로 확인
- 검색량이 적더라도 CTR과 CVR이 높으면 중요한 검색 조건일 수 있음

### 폴아웃 리포트

- 사용자가 정해진 단계를 얼마나 순서대로 통과하는지 확인하는 리포트
- 첫 단계 대비 이동률과 직전 단계 대비 이동률을 함께 확인
- 이탈이 급격하게 발생하는 단계가 개선 우선순위

### 사용자 흐름과 완독률

- `LAG` : 현재 페이지 직전 행동 확인
- `LEAD` : 현재 페이지 다음 행동 확인
- 완독률 : 페이지 조회자 중 20%, 40%, 60%, 80%, 100% 지점까지 읽은 비율
- 페이지 뷰만으로 알 수 없는 콘텐츠 소비 깊이 확인 가능

### 사용자 행동 전체 시각화

- 개별 지표만 확인하기보다 유입 → 탐색 → 상세 → 입력 → 완료까지 조감도로 연결
- 분석 목적 : 숫자 추출보다 서비스 전체의 병목과 개선 지점 발견

---

## 16장 입력 양식 최적화하기

### EFO

**EFO Entry Form Optimization** : 입력 양식의 이탈을 줄이고 완료율을 높이기 위한 최적화

대표적인 개선 방향

- 필수 입력과 선택 입력 구분
- 입력 항목 수 축소
- 자동 완성 기능 활용
- 입력 예시 제공
- 오류를 즉시 알려주는 방식 적용
- 불필요한 링크 제거
- 실수로 페이지를 벗어나는 상황 방지

### 주요 지표

| 지표 | 의미 |
| --- | --- |
| 오류율 | 확인 화면 접근 중 오류가 발생한 비율 |
| 확정률 | 입력 화면에서 확인 화면까지 이동한 비율 |
| CVR | 입력 화면에서 완료 화면까지 이동한 비율 |
| 이탈률 | 완료까지 이동하지 못한 비율 |
| 입력 양식 직귀율 | 입력 화면 방문 후 다음 단계로 이동하지 않은 비율 |

### 오류 로그 설계

- `form` : 입력 양식 종류
- `field` : 오류가 발생한 항목
- `error_type` : 필수값 누락, 형식 오류 등 오류 종류
- 오류 수와 구성비를 함께 집계해 우선 개선 항목 선정
- 개인정보가 포함될 수 있는 실제 입력값 저장은 주의 필요

---

# 실습

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **08_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 1. 사이트 전체의 특징/경향 찾기

### 1-1 날짜별 방문자 수 / 방문 횟수 / 페이지 뷰 집계하기

- 방문자 수 : `long_session`의 유니크 수
- 방문 횟수 : `short_session`의 유니크 수
- 페이지 뷰 : 접근 로그의 전체 행 수
- 같은 로그에서 세 지표를 함께 집계하면 사이트 규모와 이용 깊이를 동시에 확인 가능

```sql
SELECT
    DATE(stamp) AS dt,
    COUNT(DISTINCT long_session) AS visitors,
    COUNT(DISTINCT short_session) AS visits,
    COUNT(*) AS page_views,
    ROUND(
        1.0 * COUNT(*) / NULLIF(COUNT(DISTINCT short_session), 0),
        2
    ) AS pv_per_visit
FROM access_log
GROUP BY DATE(stamp)
ORDER BY dt;
```

![1-1 실행 결과](/image/week5/1-1.png)

### 1-2 페이지별 쿠키 / 방문 횟수 / 페이지 뷰 집계하기

- URL 전체를 그대로 집계하면 요청 매개변수 때문에 같은 페이지가 여러 개로 분리될 수 있음
- 분석 목적에 따라 URL → 경로 → 페이지 의미의 순서로 집계 단위를 조정
- 집계 단위가 너무 세밀하면 전체 경향 파악이 어려워질 수 있음

```sql
WITH access_with_path AS (
    SELECT
        *,
        SUBSTRING(
            SUBSTRING_INDEX(
                SUBSTRING_INDEX(url, '?', 1),
                '#',
                1
            ),
            LOCATE(
                '/',
                SUBSTRING_INDEX(url, '://', -1)
            ) + LOCATE('://', url) + 2
        ) AS url_path
    FROM access_log
)
SELECT
    url_path,
    COUNT(DISTINCT long_session) AS cookies,
    COUNT(DISTINCT short_session) AS visits,
    COUNT(*) AS page_views
FROM access_with_path
GROUP BY url_path
ORDER BY page_views DESC;
```

![1-2 실행 결과](/image/week5/1-2.png)

### 1-3 유입원별로 방문 횟수 또는 CVR 집계하기

- 유입원 판정 : URL 매개변수와 referrer를 함께 활용
- `CVR` : 방문 중 성과로 이어진 비율
- 유입 수가 많은 채널과 실제 성과가 높은 채널은 다를 수 있으므로 방문 수 · 전환 수 · CVR을 함께 확인

```sql
WITH entry_log AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY short_session
            ORDER BY stamp
        ) AS rn
    FROM access_log
),
entry_with_via AS (
    SELECT
        short_session,
        CASE
            WHEN url LIKE '%utm_source=google%'
             AND url LIKE '%utm_medium=cpc%'
                THEN 'google-cpc'
            WHEN url LIKE '%utm_source=mynavi%'
             AND url LIKE '%utm_medium=affiliate%'
                THEN 'mynavi-affiliate'
            WHEN url LIKE '%utm_source=facebook%'
                THEN 'social'
            WHEN referrer LIKE '%twitter.com%'
              OR referrer LIKE '%facebook.com%'
                THEN 'social'
            WHEN referrer LIKE '%google.%'
              OR referrer LIKE '%search.naver.%'
                THEN 'search'
            ELSE 'other'
        END AS via
    FROM entry_log
    WHERE rn = 1
),
purchase_by_session AS (
    SELECT
        short_session,
        SUM(amount) AS amount
    FROM purchase_log
    GROUP BY short_session
)
SELECT
    e.via,
    COUNT(*) AS visits,
    SUM(CASE WHEN p.amount IS NOT NULL THEN 1 ELSE 0 END) AS conversions,
    ROUND(
        100.0 * AVG(
            CASE WHEN p.amount IS NOT NULL THEN 1 ELSE 0 END
        ),
        2
    ) AS cvr,
    SUM(COALESCE(p.amount, 0)) AS amount
FROM entry_with_via AS e
LEFT JOIN purchase_by_session AS p
    ON e.short_session = p.short_session
GROUP BY e.via
ORDER BY cvr DESC;
```

![1-3 실행 결과](/image/week5/1-3.png)
 
### 1-4 접근 요일,시간대 파악하기

- 요일과 시간대 : 서비스의 이용 주기를 파악하는 기본 축
- 캠페인 시작 시점, 공지 발송 시점, 메일 발송 시점 결정 등에 활용
- `DAYNAME`, `HOUR` 등 날짜 함수 활용

```sql
SELECT
    DAYNAME(stamp) AS day_name,
    HOUR(stamp) AS hour_value,
    COUNT(DISTINCT short_session) AS visits,
    COUNT(*) AS page_views
FROM access_log
GROUP BY
    DAYOFWEEK(stamp),
    DAYNAME(stamp),
    HOUR(stamp)
ORDER BY
    DAYOFWEEK(stamp),
    hour_value;
```

![1-4 실행 결과](/image/week5/1-4.png)


## 2. 사이트 내의 사용자 행동 파악하기 

### 2-1 입구 페이지와 출구 페이지 파악하기

- 입구 페이지 : 세션의 첫 페이지
- 출구 페이지 : 세션의 마지막 페이지
- 세션 내부 로그 순서가 중요하므로 윈도 함수 활용
- `FIRST_VALUE`, `LAST_VALUE`로 첫 행동과 마지막 행동 추출 가능

```sql
WITH landing_exit AS (
    SELECT
        session_id,
        FIRST_VALUE(path) OVER (
            PARTITION BY session_id
            ORDER BY stamp
            ROWS BETWEEN UNBOUNDED PRECEDING
                     AND UNBOUNDED FOLLOWING
        ) AS landing,
        LAST_VALUE(path) OVER (
            PARTITION BY session_id
            ORDER BY stamp
            ROWS BETWEEN UNBOUNDED PRECEDING
                     AND UNBOUNDED FOLLOWING
        ) AS exit_page
    FROM activity_log
),
session_page AS (
    SELECT DISTINCT
        session_id,
        landing,
        exit_page
    FROM landing_exit
)
SELECT
    'landing' AS page_type,
    landing AS path,
    COUNT(*) AS visits
FROM session_page
GROUP BY landing

UNION ALL

SELECT
    'exit' AS page_type,
    exit_page AS path,
    COUNT(*) AS visits
FROM session_page
GROUP BY exit_page;
```

![2-1 실행 결과](/image/week5/2-1.png)

### 2-2 이탈률과 직귀율 계산하기

| 지표 | 의미 |
| --- | --- |
| 이탈률 | 해당 페이지에서 방문이 종료된 비율 |
| 직귀율 | 입구 페이지 한 개만 보고 종료된 비율 |

⚠️ 완료 페이지처럼 정상적으로 목적을 달성한 페이지는 높은 이탈률을 문제로 해석하지 않음

```sql
WITH page_flag AS (
    SELECT
        *,
        CASE
            WHEN ROW_NUMBER() OVER (
                PARTITION BY session_id
                ORDER BY stamp DESC
            ) = 1
            THEN 1 ELSE 0
        END AS is_exit,
        CASE
            WHEN ROW_NUMBER() OVER (
                PARTITION BY session_id
                ORDER BY stamp
            ) = 1
            THEN 1 ELSE 0
        END AS is_landing,
        CASE
            WHEN COUNT(*) OVER (
                PARTITION BY session_id
            ) = 1
            THEN 1 ELSE 0
        END AS is_bounce
    FROM activity_log
)
SELECT
    path,
    COUNT(*) AS page_views,
    SUM(is_exit) AS exit_count,
    ROUND(100.0 * AVG(is_exit), 2) AS exit_rate,
    SUM(is_landing) AS landing_count,
    SUM(is_bounce) AS bounce_count,
    ROUND(
        100.0
        * SUM(is_bounce)
        / NULLIF(SUM(is_landing), 0),
        2
    ) AS bounce_rate
FROM page_flag
GROUP BY path
ORDER BY page_views DESC;
```

![2-2 실행 결과](/image/week5/2-2.png)

### 2-3 성과로 이어지는 페이지 파악하기 

- 컨버전 페이지에 도달한 세션과 도달하지 않은 세션을 구분
- 페이지별 방문 수와 컨버전 수를 비교해 성과에 가까운 페이지 탐색
- 조회 수가 적어도 CVR이 높다면 중요한 페이지일 가능성 존재

```sql
WITH conversion_flag AS (
    SELECT
        *,
        MAX(
            CASE
                WHEN path = '/complete' THEN 1
                ELSE 0
            END
        ) OVER (
            PARTITION BY session_id
        ) AS has_conversion
    FROM activity_log
)
SELECT
    path,
    COUNT(DISTINCT session_id) AS visits,
    COUNT(
        DISTINCT CASE
            WHEN has_conversion = 1 THEN session_id
        END
    ) AS conversions,
    ROUND(
        100.0
        * COUNT(
            DISTINCT CASE
                WHEN has_conversion = 1 THEN session_id
            END
        )
        / NULLIF(COUNT(DISTINCT session_id), 0),
        2
    ) AS cvr
FROM conversion_flag
WHERE path <> '/complete'
GROUP BY path
ORDER BY cvr DESC, visits DESC;
```

![2-3 실행 결과](/image/week5/2-3.png)

### 2-4 페이지 가치 산출하기 

- 페이지 가치 : 컨버전의 가치를 경유 페이지에 배분한 지표
- 단순 페이지 뷰보다 각 페이지가 성과에 얼마나 기여했는지 평가 가능
- 균등 배분 · 최초 접점 · 마지막 접점 등 다양한 배분 기준 사용 가능

```sql
WITH conversion_flag AS (
    SELECT
        *,
        MAX(
            CASE
                WHEN path = '/complete' THEN 1
                ELSE 0
            END
        ) OVER (
            PARTITION BY session_id
        ) AS has_conversion
    FROM activity_log
),
target_log AS (
    SELECT
        session_id,
        stamp,
        path,
        ROW_NUMBER() OVER (
            PARTITION BY session_id
            ORDER BY stamp
        ) AS asc_order,
        ROW_NUMBER() OVER (
            PARTITION BY session_id
            ORDER BY stamp DESC
        ) AS desc_order,
        COUNT(*) OVER (
            PARTITION BY session_id
        ) AS page_count
    FROM conversion_flag
    WHERE has_conversion = 1
      AND path NOT IN ('/input', '/confirm', '/complete')
),
page_value AS (
    SELECT
        *,
        1000.0 / page_count AS fair_value,
        CASE
            WHEN asc_order = 1 THEN 1000.0
            ELSE 0
        END AS first_value,
        CASE
            WHEN desc_order = 1 THEN 1000.0
            ELSE 0
        END AS last_value
    FROM target_log
)
SELECT
    path,
    COUNT(*) AS page_views,
    ROUND(SUM(fair_value), 2) AS fair_value,
    ROUND(SUM(first_value), 2) AS first_value,
    ROUND(SUM(last_value), 2) AS last_value,
    ROUND(AVG(fair_value), 2) AS avg_fair_value
FROM page_value
GROUP BY path
ORDER BY fair_value DESC;
```

![2-4 실행 결과](/image/week5/2-4.png)

### 2-5 검색 조건들의 사용자 행동 가시화하기 

- 검색 기능 평가는 검색 수만으로 충분하지 않음
- `CTR` : 검색 결과에서 상세 페이지로 이동한 비율
- `CVR` : 상세 페이지 이동 후 성과까지 이어진 비율
- 검색 조건별 CTR과 CVR을 비교해 유효한 검색 기능 파악

```sql
WITH session_flag AS (
    SELECT
        session_id,
        MAX(
            CASE WHEN path = '/detail' THEN 1 ELSE 0 END
        ) AS has_detail,
        MAX(
            CASE WHEN path = '/complete' THEN 1 ELSE 0 END
        ) AS has_conversion
    FROM activity_log
    GROUP BY session_id
),
search_log AS (
    SELECT
        a.session_id,
        a.search_type,
        f.has_detail,
        f.has_conversion
    FROM activity_log AS a
    JOIN session_flag AS f
        ON a.session_id = f.session_id
    WHERE a.path = '/search_list'
)
SELECT
    search_type,
    COUNT(*) AS searches,
    SUM(has_detail) AS detail_clicks,
    ROUND(100.0 * AVG(has_detail), 2) AS ctr,
    SUM(
        CASE
            WHEN has_detail = 1 THEN has_conversion
        END
    ) AS conversions,
    ROUND(
        100.0 * AVG(
            CASE
                WHEN has_detail = 1 THEN has_conversion
            END
        ),
        2
    ) AS cvr
FROM search_log
GROUP BY search_type
ORDER BY searches DESC;
```

![2-5 실행 결과](/image/week5/2-5.png)

### 2-6 폴아웃 리포트를 사용해 사용자 회유를 가시화하기 

- 폴아웃 리포트 : 정해진 사용자 이동 단계를 순서대로 얼마나 통과했는지 확인
- 첫 단계 대비 이동률과 직전 단계 대비 이동률을 함께 비교
- 이동률이 급격하게 낮아지는 단계 : 주요 개선 후보

```sql
WITH session_step AS (
    SELECT
        session_id,
        MIN(CASE WHEN path = '/' THEN stamp END) AS step1,
        MIN(CASE WHEN path = '/search_list' THEN stamp END) AS step2,
        MIN(CASE WHEN path = '/detail' THEN stamp END) AS step3,
        MIN(CASE WHEN path = '/input' THEN stamp END) AS step4,
        MIN(CASE WHEN path = '/complete' THEN stamp END) AS step5
    FROM activity_log
    GROUP BY session_id
),
step_count AS (
    SELECT 1 AS step, '/' AS path,
           SUM(step1 IS NOT NULL) AS users
    FROM session_step

    UNION ALL

    SELECT 2, '/search_list',
           SUM(step1 IS NOT NULL
               AND step2 > step1)
    FROM session_step

    UNION ALL

    SELECT 3, '/detail',
           SUM(step1 IS NOT NULL
               AND step2 > step1
               AND step3 > step2)
    FROM session_step

    UNION ALL

    SELECT 4, '/input',
           SUM(step1 IS NOT NULL
               AND step2 > step1
               AND step3 > step2
               AND step4 > step3)
    FROM session_step

    UNION ALL

    SELECT 5, '/complete',
           SUM(step1 IS NOT NULL
               AND step2 > step1
               AND step3 > step2
               AND step4 > step3
               AND step5 > step4)
    FROM session_step
)
SELECT
    step,
    path,
    users,
    ROUND(
        100.0 * users
        / FIRST_VALUE(users) OVER (
            ORDER BY step
        ),
        2
    ) AS first_trans_rate,
    ROUND(
        100.0 * users
        / NULLIF(
            LAG(users) OVER (
                ORDER BY step
            ),
            0
        ),
        2
    ) AS step_trans_rate
FROM step_count
ORDER BY step;
```

![2-6 실행 결과](/image/week5/2-6.png)

### 2-7 사이트 내부에서 사용자 흐름 파악하기 

- 사용자 흐름 : 한 페이지의 이전 행동과 다음 행동을 연결해 확인
- `LAG` : 이전 페이지
- `LEAD` : 다음 페이지
- 예상한 동선과 실제 동선의 차이를 확인해 메뉴와 페이지 배치 개선에 활용

```sql
WITH user_flow AS (
    SELECT
        session_id,
        stamp,
        path,
        LAG(path) OVER (
            PARTITION BY session_id
            ORDER BY stamp
        ) AS previous_path,
        LEAD(path) OVER (
            PARTITION BY session_id
            ORDER BY stamp
        ) AS next_path
    FROM activity_log
)
SELECT
    path,
    previous_path,
    next_path,
    COUNT(*) AS flow_count
FROM user_flow
GROUP BY
    path,
    previous_path,
    next_path
ORDER BY
    flow_count DESC,
    path;
```

![2-7 실행 결과](/image/week5/2-7.png)

### 2-8 페이지 완독률 집계하기 

- 완독률 : 페이지 조회자가 콘텐츠의 어느 지점까지 읽었는지 나타내는 지표
- `view`를 기준 100%로 두고 `read-20%`, `read-40%` 등의 도달 비율 계산
- 페이지 뷰만으로 확인하기 어려운 콘텐츠 소비 깊이 측정 가능

```sql
WITH read_count AS (
    SELECT
        url,
        action,
        COUNT(*) AS action_count
    FROM read_log
    GROUP BY
        url,
        action
),
read_with_view AS (
    SELECT
        *,
        MAX(
            CASE
                WHEN action = 'view' THEN action_count
            END
        ) OVER (
            PARTITION BY url
        ) AS view_count
    FROM read_count
)
SELECT
    url,
    action,
    action_count,
    ROUND(
        100.0 * action_count
        / NULLIF(view_count, 0),
        2
    ) AS action_per_view
FROM read_with_view
ORDER BY
    url,
    CASE action
        WHEN 'view' THEN 0
        WHEN 'read-20%' THEN 1
        WHEN 'read-40%' THEN 2
        WHEN 'read-60%' THEN 3
        WHEN 'read-80%' THEN 4
        WHEN 'read-100%' THEN 5
        ELSE 6
    END;
```

![2-8 실행 결과](/image/week5/2-8.png)

### 2-9 사용자 행동 전체를 시각화하기 

- 개별 리포트를 하나의 조감도로 연결하면 사이트 전체의 사용자 흐름 파악 가능
- 유입 → 탐색 → 상세 → 성과의 단계별 규모와 이동률을 함께 확인
- 분석 결과를 실제 서비스 개선과 조직의 의사결정으로 연결하는 것이 핵심

```sql
WITH page_metric AS (
    SELECT
        *,
        CASE
            WHEN ROW_NUMBER() OVER (
                PARTITION BY session_id
                ORDER BY stamp DESC
            ) = 1
            THEN 1 ELSE 0
        END AS is_exit,
        MAX(
            CASE
                WHEN path = '/complete' THEN 1
                ELSE 0
            END
        ) OVER (
            PARTITION BY session_id
        ) AS has_conversion
    FROM activity_log
)
SELECT
    path,
    COUNT(*) AS page_views,
    COUNT(DISTINCT session_id) AS visits,
    SUM(is_exit) AS exits,
    ROUND(100.0 * AVG(is_exit), 2) AS exit_rate,
    COUNT(
        DISTINCT CASE
            WHEN has_conversion = 1 THEN session_id
        END
    ) AS conversion_sessions
FROM page_metric
GROUP BY path
ORDER BY page_views DESC;
```

![2-9 실행 결과](/image/week5/2-9.png)

## 3. 입력 양식 최적화하기 

### 3-1 오류율 집계하기 

- 같은 확인 화면이라도 정상 출력과 오류 출력이 함께 존재할 수 있음
- 오류 상태를 별도 컬럼으로 로그에 남겨야 정확한 오류율 계산 가능
- 오류율뿐 아니라 사용자 1명당 오류 횟수도 함께 확인 가능

```sql
SELECT
    COUNT(*) AS confirm_count,
    SUM(
        CASE WHEN status = 'error' THEN 1 ELSE 0 END
    ) AS error_count,
    ROUND(
        100.0 * AVG(
            CASE WHEN status = 'error' THEN 1 ELSE 0 END
        ),
        2
    ) AS error_rate,
    ROUND(
        1.0 * SUM(
            CASE WHEN status = 'error' THEN 1 ELSE 0 END
        )
        / NULLIF(COUNT(DISTINCT session_id), 0),
        2
    ) AS error_per_user
FROM form_log
WHERE path = '/regist/confirm';
```

![3-1 실행 결과](/image/week5/3-1.png)

### 3-2 입력 ~ 확인 ~ 완료까지의 이동률 집계하기 

- 입력 → 확인 → 완료의 단계별 이동률을 폴아웃 형태로 집계
- 확정률 : 입력 화면 → 확인 화면 이동 비율
- CVR : 입력 화면 → 완료 화면 이동 비율
- 단계 사이에서 이동률이 급격히 낮아지는 구간이 개선 대상

```sql
WITH session_step AS (
    SELECT
        session_id,
        MIN(
            CASE
                WHEN path = '/regist/input'
                THEN stamp
            END
        ) AS input_time,
        MIN(
            CASE
                WHEN path = '/regist/confirm'
                 AND status = ''
                THEN stamp
            END
        ) AS confirm_time,
        MIN(
            CASE
                WHEN path = '/regist/complete'
                THEN stamp
            END
        ) AS complete_time
    FROM form_log
    GROUP BY session_id
),
step_count AS (
    SELECT 1 AS step, '/regist/input' AS path,
           SUM(input_time IS NOT NULL) AS users
    FROM session_step

    UNION ALL

    SELECT 2, '/regist/confirm',
           SUM(
               input_time IS NOT NULL
               AND confirm_time > input_time
           )
    FROM session_step

    UNION ALL

    SELECT 3, '/regist/complete',
           SUM(
               input_time IS NOT NULL
               AND confirm_time > input_time
               AND complete_time > confirm_time
           )
    FROM session_step
)
SELECT
    step,
    path,
    users,
    ROUND(
        100.0 * users
        / FIRST_VALUE(users) OVER (
            ORDER BY step
        ),
        2
    ) AS first_trans_rate,
    ROUND(
        100.0 * users
        / NULLIF(
            LAG(users) OVER (
                ORDER BY step
            ),
            0
        ),
        2
    ) AS step_trans_rate
FROM step_count
ORDER BY step;
```

![3-2 실행 결과](/image/week5/3-2.png)

### 3-3 입력 양식 직귀율 집계하기 

- 입력 양식 직귀 : 입력 화면을 본 뒤 확인 또는 완료 단계로 이동하지 않은 세션
- 직귀율이 높으면 입력 항목 수 · 화면 구성 · 사용자의 입력 동기 등을 점검할 필요
- 입력 시작 자체를 정확히 알고 싶다면 별도의 입력 시작 로그가 필요

```sql
WITH form_progress AS (
    SELECT
        DATE(stamp) AS dt,
        session_id,
        MAX(
            CASE
                WHEN path = '/regist/input' THEN 1
                ELSE 0
            END
        ) AS has_input,
        MAX(
            CASE
                WHEN path IN (
                    '/regist/confirm',
                    '/regist/complete'
                )
                THEN 1
                ELSE 0
            END
        ) AS has_progress
    FROM form_log
    GROUP BY
        DATE(stamp),
        session_id
)
SELECT
    dt,
    COUNT(*) AS input_count,
    SUM(
        CASE WHEN has_progress = 0 THEN 1 ELSE 0 END
    ) AS bounce_count,
    ROUND(
        100.0 * AVG(
            CASE WHEN has_progress = 0 THEN 1 ELSE 0 END
        ),
        2
    ) AS bounce_rate
FROM form_progress
WHERE has_input = 1
GROUP BY dt
ORDER BY dt;
```

![3-3 실행 결과](/image/week5/3-3.png)

### 3-4 오류가 발생하는 항목과 내용 집계하기 

- 오류를 입력 양식 · 항목 · 오류 종류 단위로 세분화
- 전체 오류 수보다 구성비까지 함께 보면 개선 우선순위 결정이 쉬움
- 개인정보 입력값을 그대로 로그에 저장하지 않도록 주의

```sql
WITH error_count AS (
    SELECT
        form_name,
        field_name,
        error_type,
        COUNT(*) AS error_count
    FROM form_error_log
    GROUP BY
        form_name,
        field_name,
        error_type
)
SELECT
    form_name,
    field_name,
    error_type,
    error_count,
    ROUND(
        100.0 * error_count
        / SUM(error_count) OVER (
            PARTITION BY form_name
        ),
        2
    ) AS share
FROM error_count
ORDER BY
    form_name,
    error_count DESC;
```

![3-4 실행 결과](/image/week5/3-4.png)


### 🎉 수고하셨습니다.
