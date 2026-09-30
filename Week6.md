# SQL_MASTER 6주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_6th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_6th_TIL

### 7장 데이터 활용의 정밀도를 높이는 분석 기술
#### 1. 데이터를 조합해서 새로운 데이터 만들기
#### 2. 이상값 검출하기 
#### 3. 데이터 중복 검출하기
#### 4. 여러 개의 데이터셋 비교하기 


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.52~136   | ✅         |
| 2주차 | p.138~184  | ✅         |
| 3주차 | p.186~232 | ✅         |
| 4주차 | p.233~321 | ✅         |
| 5주차 | p.324~406 | ✅         |
| 6주차 | p.408~464 | ✅         |
| 7주차 | p.466~566 | 🍽️         |

<br>

<!-- 여기까진 그대로 둬 주세요-->


# 📚 7장 교재 정리｜데이터 활용의 정밀도를 높이는 분석 기술

## 17장 데이터를 조합해서 새로운 데이터 만들기

### 외부 데이터 활용

- 서비스 내부 데이터만으로 부족한 정보를 외부 데이터와 결합
- 오픈 데이터 · 지역 정보 · 달력 정보 등을 활용해 분석 범위 확장 가능
- 기존 로그에 새로운 속성을 추가해 분석 정밀도 향상

### IP 주소 기반 지역 정보 보완

- IP 주소 → 국가 · 도시 · 타임존 등의 지역 정보로 변환
- PostgreSQL : `inet` 자료형 활용 가능
- MySQL : `INET_ATON()`으로 IPv4를 숫자로 변환해 범위 비교 가능

### 주말 · 공휴일 판정

| 구분 | 판정 방법 |
| --- | --- |
| 주말 | 요일 정보 활용 |
| 공휴일 | 별도의 달력 마스터와 결합 |
| 휴일 여부 | 주말 또는 공휴일이면 휴일로 판정 |

📌 서비스별로 평일과 휴일의 방문 · 전환 패턴이 다를 수 있어 목표 설정 시 활용 가능

### 하루 집계 범위 변경

- 기본 날짜 집계 : 자정 기준
- 서비스 이용이 자정 전후에 이어지는 경우 임의의 기준 시각 설정 가능
- 오전 4시 기준 : 타임스탬프를 4시간 앞으로 당긴 뒤 날짜 추출

---

## 18장 이상값 검출하기

### 데이터 분산

- 극단적으로 많은 접근 : 크롤러 · 부정 접근 가능성
- 극단적으로 적은 접근 : 잘못된 URL · 비정상 접근 가능성
- `PERCENT_RANK()` : 순위가 전체에서 어느 위치인지 비율로 표현

### 크롤러 제외

대표적인 판정 문자열

| 유형 | 예시 |
| --- | --- |
| 일반 키워드 | `bot`, `crawler`, `spider`, `archiver` |
| 크롤러 이름 | `Googlebot`, `Baiduspider`, `Yeti` |

- 크롤러 로그 : 사용자 행동 분석에서는 노이즈
- 새로운 크롤러가 계속 추가될 수 있어 주기적인 확인 필요

### 데이터 타당성

- 로그의 액션 종류에 따라 필수 컬럼이 달라질 수 있음
- `AVG(CASE ...)` : 유효한 레코드의 비율 계산에 활용
- 원본 데이터의 결손과 오류를 확인한 뒤 분석 진행

### 특정 IP 제외

- 사내 접근 · 테스트 사용자 · 사설 네트워크 등 분석 대상이 아닌 접근 제거
- IP 주소를 비교 가능한 값으로 변환한 뒤 네트워크 범위와 비교
- 대표 사설 네트워크 : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`

---

## 19장 데이터 중복 검출하기

### 마스터 데이터 중복

중복 발생 예시

- 같은 데이터가 여러 번 로드
- 갱신 과정에서 이전 데이터와 신규 데이터가 함께 존재
- 같은 ID를 다른 데이터에 재사용

중복 확인 방법

- 전체 레코드 수와 `COUNT(DISTINCT key)` 비교
- `GROUP BY` + `HAVING COUNT(*) > 1`
- 윈도 함수로 각 키의 중복 개수 부여

### 로그 중복

- 동일 사용자 · 동일 액션 · 동일 상품의 로그가 짧은 시간 안에 반복되면 중복 가능성 존재
- `LAG()` : 직전 로그 시각 확인
- 직전 로그와의 시간 차이가 기준보다 작으면 중복으로 판정 가능

---

## 20장 여러 개의 데이터셋 비교하기

### 데이터 변경 유형

| 상태 | 의미 |
| --- | --- |
| added | 새로운 데이터셋에만 존재 |
| deleted | 이전 데이터셋에만 존재 |
| updated | 양쪽에 존재하지만 값이 변경 |

- PostgreSQL : `FULL OUTER JOIN` 활용 가능
- MySQL : `LEFT JOIN` 결과를 `UNION ALL`로 결합해 같은 결과 구현 가능

### 순위 유사도

**스피어만 상관계수** : 두 순위의 유사성을 수치화하는 지표

```math
\rho = 1 - \frac{6\sum d_i^2}{n^3-n}
```

| 값 | 해석 |
| --- | --- |
| 1에 가까움 | 두 순위가 비슷한 방향 |
| 0에 가까움 | 순위 사이의 관계가 약함 |
| -1에 가까움 | 두 순위가 반대 방향 |

- 순위를 눈으로 비교하는 대신 정량적으로 평가 가능
- 집계 기준이나 랭킹 로직 변경 전후 비교에 활용 가능

---

# 실습

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **08_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 1. 데이터를 조합해서 새로운 데이터 만들기

### 1-1 IP 주소를 기반으로 국가와 지역 보완하기 

- IP 주소 자체만으로는 지역 정보 확인 어려움
- IP 범위 마스터와 지역 마스터를 결합하면 국가 · 도시 · 타임존 등의 정보 보완 가능
- MySQL에서는 `INET_ATON()`을 활용해 IPv4 범위 비교 가능

```sql
SELECT
    a.ip,
    l.continent_name,
    l.country_name,
    l.city_name,
    l.time_zone
FROM action_log_with_ip AS a
LEFT JOIN mst_city_ip AS i
    ON INET_ATON(a.ip)
       BETWEEN INET_ATON(i.network_start_ip)
           AND INET_ATON(i.network_last_ip)
LEFT JOIN mst_locations AS l
    ON i.geoname_id = l.geoname_id
ORDER BY a.stamp;
```

![1-1 실행 결과](/image/week6/1-1.png)

### 1-2 주말과 공휴일 판단하기 

- 주말 : 요일 정보만으로 판정 가능
- 공휴일 : 날짜별 휴일 정보를 가진 별도 마스터 필요
- 로그 날짜와 달력 마스터를 결합하면 평일 · 휴일 기준 분석 가능

```sql
SELECT
    a.action,
    a.stamp,
    c.dow,
    c.holiday_name,
    CASE
        WHEN c.dow_num IN (0, 6)
          OR c.holiday_name IS NOT NULL
        THEN 1
        ELSE 0
    END AS is_day_off
FROM access_log_calendar AS a
JOIN mst_calendar AS c
    ON DATE(a.stamp) = c.calendar_date
ORDER BY a.stamp;
```

![1-2 실행 결과](/image/week6/1-2.png)

### 1-3 하루 집계 범위 변경하기 

- 일반적인 날짜 집계 기준 : 자정
- 서비스 특성에 따라 하루의 시작 시각을 변경 가능
- 오전 4시 기준 집계 : 원본 시각에서 4시간을 뺀 날짜를 집계일로 사용

```sql
SELECT
    session_id,
    user_id,
    action,
    stamp,
    DATE(stamp) AS raw_date,
    DATE(
        DATE_SUB(
            stamp,
            INTERVAL 4 HOUR
        )
    ) AS mod_date
FROM action_log_day
ORDER BY stamp;
```

![1-3 실행 결과](/image/week6/1-3.png)


## 2. 이상값 검출하기 

### 2-1 데이터 분산 계산하기 

- 이상값 탐색의 기본 : 데이터 분포에서 극단적인 값 확인
- `PERCENT_RANK()` : 현재 순위가 전체 순위에서 차지하는 위치를 0~1로 표현
- 상위 · 하위 일정 비율을 필터링해 비정상 데이터 후보 탐색 가능

```sql
WITH session_count AS (
    SELECT
        session_id,
        COUNT(*) AS access_count
    FROM action_log_with_noise
    GROUP BY session_id
)
SELECT
    session_id,
    access_count,
    RANK() OVER (
        ORDER BY access_count DESC
    ) AS rank_value,
    ROUND(
        PERCENT_RANK() OVER (
            ORDER BY access_count DESC
        ),
        4
    ) AS percent_rank
FROM session_count
ORDER BY access_count DESC, session_id;
```

![2-1 실행 결과](/image/week6/2-1.png)

### 2-2 크롤러 제외하기 

- 크롤러 접근 : 일반 사용자 행동 분석에서는 노이즈
- 사용자 에이전트에 포함된 `bot`, `crawler`, `spider` 등의 문자열로 1차 판정 가능
- 신규 크롤러가 계속 발생하므로 주기적인 로그 확인 필요

```sql
SELECT
    stamp,
    session_id,
    action,
    url,
    user_agent
FROM action_log_with_noise
WHERE LOWER(user_agent) NOT LIKE '%bot%'
  AND LOWER(user_agent) NOT LIKE '%crawler%'
  AND LOWER(user_agent) NOT LIKE '%spider%'
  AND LOWER(user_agent) NOT LIKE '%archiver%'
ORDER BY stamp;
```

![2-2 실행 결과](/image/week6/2-2.png)

### 2-3 데이터 타당성 확인하기 

- 로그 분석 전 결손값과 비정상값 확인 필요
- 액션마다 필수 컬럼이 다르므로 액션별 조건을 적용해 타당성 검증
- `AVG(CASE ...)`로 조건을 만족하는 데이터 비율 계산 가능

```sql
SELECT
    action,
    COUNT(*) AS records,
    ROUND(
        100.0 * AVG(
            CASE
                WHEN session_id IS NOT NULL
                 AND user_id IS NOT NULL
                 AND stamp IS NOT NULL
                THEN 1 ELSE 0
            END
        ),
        2
    ) AS basic_valid_rate,
    ROUND(
        100.0 * AVG(
            CASE
                WHEN action IN ('favorite', 'add_cart', 'purchase')
                    THEN category IS NOT NULL
                ELSE 1
            END
        ),
        2
    ) AS category_valid_rate,
    ROUND(
        100.0 * AVG(
            CASE
                WHEN action IN ('favorite', 'add_cart', 'purchase')
                    THEN products IS NOT NULL
                ELSE 1
            END
        ),
        2
    ) AS products_valid_rate,
    ROUND(
        100.0 * AVG(
            CASE
                WHEN action = 'purchase'
                    THEN amount IS NOT NULL
                ELSE 1
            END
        ),
        2
    ) AS amount_valid_rate
FROM invalid_action_log
GROUP BY action
ORDER BY action;
```

![2-3 실행 결과](/image/week6/2-3.png)

### 2-4 특정 IP 주소에서의 접근 제외하기 

- 사내 · 테스트 · 사설 IP 접근은 일반 사용자 분석에서 제외 가능
- IP를 숫자로 변환하면 시작 IP와 종료 IP 사이의 범위 비교 가능
- 제외 대상 네트워크를 별도 마스터로 관리하면 유지보수 용이

```sql
SELECT
    a.user_id,
    a.ip,
    a.action,
    a.stamp
FROM action_log_with_ip AS a
WHERE NOT EXISTS (
    SELECT 1
    FROM mst_reserved_ip_range AS r
    WHERE INET_ATON(a.ip)
          BETWEEN INET_ATON(r.network_start_ip)
              AND INET_ATON(r.network_last_ip)
)
ORDER BY a.stamp;
```

![2-4 실행 결과](/image/week6/2-4.png)


## 3. 데이터 중복 검출하기

### 3-1 마스터 데이터의 중복 검출하기 

- 마스터 데이터 중복 → JOIN 이후 레코드가 증가해 집계값 왜곡 가능
- 전체 레코드 수와 유니크 키 수가 다르면 중복 존재
- 중복 키별 레코드 수를 구하면 실제 중복 데이터 확인 가능

```sql
WITH category_with_dup_count AS (
    SELECT
        id,
        name,
        stamp,
        COUNT(*) OVER (
            PARTITION BY id
        ) AS duplicate_count
    FROM mst_categories
)
SELECT
    id,
    name,
    stamp,
    duplicate_count
FROM category_with_dup_count
WHERE duplicate_count > 1
ORDER BY id, stamp;
```

![3-1 실행 결과](/image/week6/3-1.png)

### 3-2 로그 중복 검출하기 

- 로그 중복은 동일 행동이 짧은 시간 안에 반복 기록되는 형태로 발생 가능
- `LAG()`로 같은 사용자 · 액션 · 상품의 직전 시각 확인
- 교재 예시 : 30분 이내의 같은 액션을 중복으로 판단 가능

```sql
WITH log_with_previous AS (
    SELECT
        user_id,
        action,
        products,
        stamp,
        LAG(stamp) OVER (
            PARTITION BY
                user_id,
                action,
                products
            ORDER BY stamp
        ) AS previous_stamp
    FROM dup_action_log
)
SELECT
    user_id,
    action,
    products,
    stamp,
    previous_stamp,
    TIMESTAMPDIFF(
        SECOND,
        previous_stamp,
        stamp
    ) AS lag_seconds
FROM log_with_previous
WHERE previous_stamp IS NOT NULL
  AND TIMESTAMPDIFF(
        SECOND,
        previous_stamp,
        stamp
      ) < 30 * 60
ORDER BY stamp;
```

![3-2 실행 결과](/image/week6/3-2.png)


## 4. 여러 개의 데이터셋 비교하기 

### 4-1 데이터의 차이 추출하기 

- 서로 다른 시점의 마스터 비교 → 추가 · 삭제 · 갱신 데이터 추출 가능
- MySQL은 `FULL OUTER JOIN` 미지원
- `LEFT JOIN`을 방향별로 수행한 뒤 `UNION ALL`로 결합 가능

```sql
SELECT
    n.product_id,
    n.name,
    n.price,
    n.updated_at,
    'added' AS status
FROM mst_products_20170101 AS n
LEFT JOIN mst_products_20161201 AS o
    ON n.product_id = o.product_id
WHERE o.product_id IS NULL

UNION ALL

SELECT
    o.product_id,
    o.name,
    o.price,
    o.updated_at,
    'deleted' AS status
FROM mst_products_20161201 AS o
LEFT JOIN mst_products_20170101 AS n
    ON o.product_id = n.product_id
WHERE n.product_id IS NULL

UNION ALL

SELECT
    n.product_id,
    n.name,
    n.price,
    n.updated_at,
    'updated' AS status
FROM mst_products_20170101 AS n
JOIN mst_products_20161201 AS o
    ON n.product_id = o.product_id
WHERE n.name <> o.name
   OR n.price <> o.price
   OR n.updated_at <> o.updated_at
ORDER BY product_id;
```

![4-1 실행 결과](/image/week6/4-1.png)

### 4-2 두 순위의 유사도 계산하기 

- 순위 변화는 눈으로만 비교하면 판단이 모호해질 수 있음
- 스피어만 상관계수 : 두 순위의 유사도를 -1~1 범위로 수치화
- `1` : 같은 방향의 순위
- `-1` : 반대 방향의 순위

```sql
WITH path_stat AS (
    SELECT
        path,
        COUNT(DISTINCT long_session) AS access_users,
        COUNT(DISTINCT short_session) AS access_count,
        COUNT(*) AS page_view
    FROM access_log_rank
    GROUP BY path
),
path_ranking AS (
    SELECT
        'access_users' AS metric_type,
        path,
        RANK() OVER (
            ORDER BY access_users DESC
        ) AS rank_value
    FROM path_stat

    UNION ALL

    SELECT
        'access_count',
        path,
        RANK() OVER (
            ORDER BY access_count DESC
        )
    FROM path_stat

    UNION ALL

    SELECT
        'page_view',
        path,
        RANK() OVER (
            ORDER BY page_view DESC
        )
    FROM path_stat
),
pair_ranking AS (
    SELECT
        r1.metric_type AS metric1,
        r2.metric_type AS metric2,
        r1.path,
        POWER(
            r1.rank_value - r2.rank_value,
            2
        ) AS diff
    FROM path_ranking AS r1
    JOIN path_ranking AS r2
        ON r1.path = r2.path
    WHERE r1.metric_type <= r2.metric_type
)
SELECT
    metric1,
    metric2,
    ROUND(
        1 - (
            6.0 * SUM(diff)
            / (
                POWER(COUNT(*), 3)
                - COUNT(*)
              )
        ),
        4
    ) AS spearman
FROM pair_ranking
GROUP BY
    metric1,
    metric2
ORDER BY
    metric1,
    spearman DESC;
```

![4-2 실행 결과](/image/week6/4-2.png)

---

### 🎉 수고하셨습니다.
