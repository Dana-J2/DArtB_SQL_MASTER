# SQL_MASTER 7주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_7th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_7th_TIL

### 8장 데이터를 무기로 삼기 위한 분석 기술
#### 1. 검색 기능 평가하기
#### 2. 데이터마이닝
#### 3. 추천 (범위 제외)
#### 4. 점수 계산하기 (범위 제외)


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.52~136   | ✅         |
| 2주차 | p.138~184  | ✅         |
| 3주차 | p.186~232 | ✅         |
| 4주차 | p.233~321 | ✅         |
| 5주차 | p.324~406 | ✅         |
| 6주차 | p.408~464 | ✅         |
| 7주차 | p.466~566 | ✅         |

<br>



# 📚 교재 핵심 개념 요약

## 21장 검색 기능 평가하기

### ✅ 검색 품질 개선과 평가 데이터

- **동의어 사전** : 별칭·약칭·표기 차이를 동일한 검색 의도로 연결
- **사용자 사전** : 고유명사·복합어의 의도하지 않은 분해 방지
- **정렬 순서 조정** : 관련도와 서비스 특성을 반영한 검색 결과 순서 개선
- **행동 로그 평가** : 검색 결과 수·재검색·상세 페이지 이동·이탈 분석
- **정답 데이터 평가** : 검색 결과와 사전 정답 아이템을 대조한 재현율·정확률·순위 품질 평가
- **검색 결과 페이지 이동과 신규 검색의 구분 필요**
- **재검색·이탈 증가** : 개선 후보 신호, 사용자 불만의 확정 증거로 해석 주의

### ✅ 행동 지표

| 지표 | 판정 기준 | 계산식 | 단위 |
| :--- | :--- | :--- | :--- |
| No Match 비율 | 검색 결과 수 0 | 결과 수 0인 검색 수 / 전체 검색 수 | 0~1 |
| 재검색 비율 | 같은 세션의 바로 다음 행동이 검색 | 연속 검색 발생 수 / 전체 검색 수 | 0~1 |
| 검색 이탈 비율 | 검색 다음 행동 부재 | 검색으로 종료된 수 / 전체 검색 수 | 0~1 |

- 다음 행동 계산 순서 : 전체 행동 로그에 `LEAD()` 적용 → 검색 행동 필터링
- 검색만 남긴 뒤 `LEAD()` 적용 시 상세 조회를 사이에 둔 검색까지 재검색으로 오판 가능
- 동일 시각 로그 : `stamp, log_id` 기준으로 순서 고정
- 이탈 판정 : 완결된 세션 로그 가정, 수집 종료 직전의 진행 중 세션 별도 주의

| 재검색 유형 | 조건 | 활용 |
| :--- | :--- | :--- |
| No Match 조건 변경 | 기존 검색 결과 수 0 | 오탈자·동의어·사용자 사전 개선 후보 |
| 검색 필터링 | 다음 검색어에 기존 검색어 포함 | 연관 검색어·필터 후보 |
| 검색어 변경 | 다음 검색어에 기존 검색어 미포함 | 다른 표현·검색 의도 변화 후보 |

- 교재의 유형별 조건은 독립 조건으로 중복 가능
- 단순 문자열 포함 기준의 휴리스틱, 의미 기반 의도 분류와 차이

### ✅ 검색 결과 평가 지표

⭐️ **주요기호**   
- `R` = 전체 정답 수
- `H(n)` = 상위 n개 중 정답 수
- `K(n)` = 상위 n개 내 실제 반환 수
- `rel(k)` = k위 정답 여부
- `Q` = 평가 키워드 수

| 지표 | 계산식 | 의미 |
| :--- | :--- | :--- |
| 재현율 Recall@n | H(n) / R | 전체 정답 중 검색 성공 비율 |
| 정확률 Precision@n | H(n) / K(n) | 실제 반환 결과 중 정답 비율 |
| AP | Σ[P@k × rel(k)] / R | 정답 위치의 정확률 평균, 미검색 정답 기여도 0 |
| MAP | Σ AP / Q | 키워드별 AP의 단순 평균 |
| MRR | 첫 정답 순위 역수의 키워드별 평균 | 첫 유효 결과의 상위 배치 정도 |
| F1 | 2 × Precision × Recall / (Precision + Recall) | 정확률과 재현율의 조화평균 |
| NDCG | DCG / 이상적 순위의 DCG | 다단계 관련도와 순위 할인 반영 |
| BPREF | 평가된 정답·오답 사이의 상대 순서 기반 | 정답 평가가 일부에만 존재하는 경우의 평가 |


## 22장 데이터마이닝 

### ✅ 데이터마이닝과 어소시에이션 분석

- 데이터마이닝 : 대량 데이터에서 유용한 패턴·규칙·지식 추출
- 대표 방법 : 연관 규칙 추출·클러스터링·상관분석
- SQL 실습 범위 : 동일 거래에 포함된 두 상품 사이의 연관 규칙
- 분석 단위 : `purchase_id` 거래, 상품 행 수·사용자 수와 구분
- 거래별 동일 상품 중복 제거 후 자기 결합
- `A → B`와 `B → A` : 방향별 확신도 차이, 지지도와 리프트 동일
- 동일 거래의 동시 구매 : 시간적 선후·인과관계 입증 불가

⭐️ **주요기호**   
- `N` = 전체 거래 수
- `N(A)` = A 포함 거래 수
- `N(B)` = B 포함 거래 수
- `N(A∩B)` = A·B 동시 포함 거래 수

| 지표 | 공식 | 해석 |
| :--- | :--- | :--- |
| 지지도 Support | N(A∩B) / N | 전체 거래에서 규칙 발생 빈도 |
| 확신도 Confidence A→B | N(A∩B) / N(A) | A 구매 거래에서 B 동시 구매 비율 |
| 리프트 Lift | Confidence(A→B) / [N(B) / N] | B 기본 구매 확률 대비 조건부 구매 확률의 배수 |

| 리프트 값 | 해석 |
| :--- | :--- |
| 1 초과 | 양의 연관성 |
| 1 | 독립 수준 |
| 1 미만 | 음의 연관성 |

<br>
<br>
<br>

# 실습

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **08_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 1. 검색 기능 평가하기

### 1-1 No Match 비율과 키워드 집계하기 

- `No Match` : 검색 결과 수 0인 검색
- 날짜별 전체 검색 대비 No Match 비율과 발생 키워드 집계
- 결과 수 NULL과 0의 구분, 상세 조회 로그 제외

```sql
SELECT
    DATE(stamp) AS dt,
    COUNT(*) AS search_count,
    SUM(CASE WHEN result_num = 0 THEN 1 ELSE 0 END) AS no_match_count,
    ROUND(AVG(CASE WHEN result_num = 0 THEN 1.0 ELSE 0.0 END), 4) AS no_match_rate
FROM w7_access_log
WHERE action_type = 'search'
GROUP BY DATE(stamp)
ORDER BY dt;

SELECT keyword, COUNT(*) AS no_match_count
FROM w7_access_log
WHERE action_type = 'search' AND result_num = 0
GROUP BY keyword
ORDER BY no_match_count DESC, keyword;
```

![1-1 실행 결과](/image/week7/1-1.png)

### 1-2 재검색 비율과 키워드 집계하기 

- `재검색` : 같은 세션에서 상세 조회 없이 연속 발생한 검색
- 전체 로그의 다음 행동·다음 키워드·다음 결과 수 확인 후 검색 로그 추출
- 일별 재검색 비율과 기존 검색어 → 다음 검색어의 전환 건수 집계

```sql
WITH next_log AS (
    SELECT
        log_id, stamp, session_id, action_type, keyword, result_num,
        LEAD(action_type) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_action,
        LEAD(keyword) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_keyword,
        LEAD(result_num) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_result_num
    FROM w7_access_log
)
SELECT
    DATE(stamp) AS dt,
    COUNT(*) AS search_count,
    SUM(CASE WHEN next_action = 'search' THEN 1 ELSE 0 END) AS retry_count,
    ROUND(AVG(CASE WHEN next_action = 'search' THEN 1.0 ELSE 0.0 END), 4) AS retry_rate
FROM next_log
WHERE action_type = 'search'
GROUP BY DATE(stamp)
ORDER BY dt;

WITH next_log AS (
    SELECT
        log_id, stamp, session_id, action_type, keyword, result_num,
        LEAD(action_type) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_action,
        LEAD(keyword) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_keyword,
        LEAD(result_num) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_result_num
    FROM w7_access_log
)
SELECT keyword, result_num, next_keyword, next_result_num,
       COUNT(*) AS retry_count
FROM next_log
WHERE action_type = 'search' AND next_action = 'search'
GROUP BY keyword, result_num, next_keyword, next_result_num
ORDER BY retry_count DESC, keyword, next_keyword;
```

![1-2 실행 결과](/image/week7/1-2.png)

### 1-3 재검색 키워드를 분류해서 집계하기 

- No Match 조건 변경·검색 필터링·검색어 변경의 세 유형별 집계
- `UNION ALL`로 교재의 독립 조건 유지, 유형 간 중복 가능
- `LOCATE()`로 포함 여부 판정, `%`·`_` 와일드카드 오해 방지

```sql
WITH next_log AS (
    SELECT
        log_id, stamp, session_id, action_type, keyword, result_num,
        LEAD(action_type) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_action,
        LEAD(keyword) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_keyword,
        LEAD(result_num) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_result_num
    FROM w7_access_log
), retry_patterns AS (
    SELECT 'no_match_change' AS retry_type, keyword, result_num,
           next_keyword, next_result_num
    FROM next_log
    WHERE action_type = 'search' AND next_action = 'search'
      AND result_num = 0
    UNION ALL
    SELECT 'filtering', keyword, result_num, next_keyword, next_result_num
    FROM next_log
    WHERE action_type = 'search' AND next_action = 'search'
      AND LOCATE(keyword, next_keyword) > 0
    UNION ALL
    SELECT 'keyword_change', keyword, result_num, next_keyword, next_result_num
    FROM next_log
    WHERE action_type = 'search' AND next_action = 'search'
      AND LOCATE(keyword, next_keyword) = 0
)
SELECT retry_type, keyword, result_num, next_keyword, next_result_num,
       COUNT(*) AS retry_count
FROM retry_patterns
GROUP BY retry_type, keyword, result_num, next_keyword, next_result_num
ORDER BY retry_type, retry_count DESC, keyword, next_keyword;
```

![1-3 실행 결과](/image/week7/1-3.png)

### 1-4 검색 이탈 비율과 키워드 집계하기

- `검색 이탈` : 검색 행동 뒤에 다음 행동이 없는 세션 종료
- 날짜별 이탈 비율과 이탈 발생 키워드별 검색 수·이탈 수·이탈률 집계
- 키워드별 분모 : 해당 키워드·결과 수 조합의 전체 검색 수

```sql
WITH next_log AS (
    SELECT
        log_id, stamp, session_id, action_type, keyword, result_num,
        LEAD(action_type) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_action,
        LEAD(keyword) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_keyword,
        LEAD(result_num) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_result_num
    FROM w7_access_log
)
SELECT
    DATE(stamp) AS dt,
    COUNT(*) AS search_count,
    SUM(CASE WHEN next_action IS NULL THEN 1 ELSE 0 END) AS exit_count,
    ROUND(AVG(CASE WHEN next_action IS NULL THEN 1.0 ELSE 0.0 END), 4) AS exit_rate
FROM next_log
WHERE action_type = 'search'
GROUP BY DATE(stamp)
ORDER BY dt;

WITH next_log AS (
    SELECT
        log_id, stamp, session_id, action_type, keyword, result_num,
        LEAD(action_type) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_action,
        LEAD(keyword) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_keyword,
        LEAD(result_num) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_result_num
    FROM w7_access_log
)
SELECT keyword, result_num, COUNT(*) AS search_count,
       SUM(CASE WHEN next_action IS NULL THEN 1 ELSE 0 END) AS exit_count,
       ROUND(AVG(CASE WHEN next_action IS NULL THEN 1.0 ELSE 0.0 END), 4) AS exit_rate
FROM next_log
WHERE action_type = 'search'
GROUP BY keyword, result_num
HAVING SUM(CASE WHEN next_action IS NULL THEN 1 ELSE 0 END) > 0
ORDER BY exit_count DESC, keyword, result_num;
```

![1-4 실행 결과](/image/week7/1-4.png)

### 1-5 검색 키워드 관련 지표의 집계 효율화 하기

- 공통 전처리 CTE : 다음 행동 계산 → 검색 필터링 → 지표 플래그 생성
- 일자·키워드별 No Match·재검색·이탈 수와 비율의 일괄 집계
- CTE 재사용에 따른 집계 로직 표준화, 물리적 실행 성능 향상 보장과 구분

```sql
WITH next_log AS (
    SELECT
        log_id, stamp, session_id, action_type, keyword, result_num,
        LEAD(action_type) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_action,
        LEAD(keyword) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_keyword,
        LEAD(result_num) OVER (
            PARTITION BY session_id ORDER BY stamp, log_id
        ) AS next_result_num
    FROM w7_access_log
), search_flags AS (
    SELECT *,
        CASE WHEN result_num = 0 THEN 1 ELSE 0 END AS no_match_flag,
        CASE WHEN next_action = 'search' THEN 1 ELSE 0 END AS retry_flag,
        CASE WHEN next_action IS NULL THEN 1 ELSE 0 END AS exit_flag
    FROM next_log
    WHERE action_type = 'search'
)
SELECT
    DATE(stamp) AS dt, keyword, COUNT(*) AS search_count,
    SUM(no_match_flag) AS no_match_count,
    ROUND(AVG(no_match_flag), 4) AS no_match_rate,
    SUM(retry_flag) AS retry_count,
    ROUND(AVG(retry_flag), 4) AS retry_rate,
    SUM(exit_flag) AS exit_count,
    ROUND(AVG(exit_flag), 4) AS exit_rate
FROM search_flags
GROUP BY DATE(stamp), keyword
ORDER BY dt, keyword;
```

![1-5 실행 결과](/image/week7/1-5.png)

### 1-6 검색 결과의 포괄성을 지표화 하기

- `재현율` : 전체 정답 중 검색된 정답의 비율
- 검색 결과·정답의 전체 외부 결합 효과 구현, 미검색 정답 유지
- 순위별 누계 재현율과 키워드별 Recall@5·단순 평균 출력

```sql
WITH evaluated_items AS (
    SELECT r.keyword, r.rank_value, r.item,
           CASE WHEN c.item IS NOT NULL THEN 1 ELSE 0 END AS correct_flag
    FROM w7_search_result AS r
    LEFT JOIN w7_correct_result AS c
      ON r.keyword = c.keyword AND r.item = c.item
    UNION ALL
    SELECT c.keyword, NULL AS rank_value, c.item, 1 AS correct_flag
    FROM w7_correct_result AS c
    LEFT JOIN w7_search_result AS r
      ON c.keyword = r.keyword AND c.item = r.item
    WHERE r.item IS NULL
), cumulative_stats AS (
    SELECT *,
        SUM(correct_flag) OVER (PARTITION BY keyword) AS correct_total,
        SUM(correct_flag) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_correct,
        COUNT(*) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_items
    FROM evaluated_items
)
SELECT keyword, rank_value, item, correct_flag, correct_total,
       cumulative_correct,
       CASE WHEN rank_value IS NULL THEN 0.0
            ELSE ROUND(100.0 * cumulative_correct / NULLIF(correct_total, 0), 2)
       END AS recall_pct
FROM cumulative_stats
ORDER BY keyword, (rank_value IS NULL), rank_value, item;

# 상위 5개 재현율 및 키워드 단순 평균
WITH evaluated_items AS (
    SELECT r.keyword, r.rank_value, r.item,
           CASE WHEN c.item IS NOT NULL THEN 1 ELSE 0 END AS correct_flag
    FROM w7_search_result AS r
    LEFT JOIN w7_correct_result AS c
      ON r.keyword = c.keyword AND r.item = c.item
    UNION ALL
    SELECT c.keyword, NULL AS rank_value, c.item, 1 AS correct_flag
    FROM w7_correct_result AS c
    LEFT JOIN w7_search_result AS r
      ON c.keyword = r.keyword AND c.item = r.item
    WHERE r.item IS NULL
), cumulative_stats AS (
    SELECT *,
        SUM(correct_flag) OVER (PARTITION BY keyword) AS correct_total,
        SUM(correct_flag) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_correct,
        COUNT(*) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_items
    FROM evaluated_items
), per_keyword AS (
    SELECT keyword,
        100.0 * SUM(CASE WHEN rank_value <= 5 THEN correct_flag ELSE 0 END)
        / NULLIF(MAX(correct_total), 0) AS recall_at_5_pct
    FROM cumulative_stats
    GROUP BY keyword
)
SELECT keyword, ROUND(recall_at_5_pct, 2) AS recall_at_5_pct,
       ROUND(AVG(recall_at_5_pct) OVER (), 2) AS mean_recall_at_5_pct
FROM per_keyword
ORDER BY keyword;
```

![1-6 실행 결과](/image/week7/1-6.png)

### 1-7 검색 결과의 타당성을 지표화 하기

- `정확률` : 실제 반환된 결과 중 정답의 비율
- 순위별 누계 정답 수 / 해당 순위까지의 실제 반환 수
- 키워드별 P@5와 평균 P@5 출력, 5개 미만 반환 시 교재 기준 적용

```sql
WITH evaluated_items AS (
    SELECT r.keyword, r.rank_value, r.item,
           CASE WHEN c.item IS NOT NULL THEN 1 ELSE 0 END AS correct_flag
    FROM w7_search_result AS r
    LEFT JOIN w7_correct_result AS c
      ON r.keyword = c.keyword AND r.item = c.item
    UNION ALL
    SELECT c.keyword, NULL AS rank_value, c.item, 1 AS correct_flag
    FROM w7_correct_result AS c
    LEFT JOIN w7_search_result AS r
      ON c.keyword = r.keyword AND c.item = r.item
    WHERE r.item IS NULL
), cumulative_stats AS (
    SELECT *,
        SUM(correct_flag) OVER (PARTITION BY keyword) AS correct_total,
        SUM(correct_flag) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_correct,
        COUNT(*) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_items
    FROM evaluated_items
)
SELECT keyword, rank_value, item, correct_flag, cumulative_correct,
       CASE WHEN rank_value IS NULL THEN 0.0
            ELSE ROUND(100.0 * cumulative_correct / NULLIF(cumulative_items, 0), 2)
       END AS precision_pct
FROM cumulative_stats
ORDER BY keyword, (rank_value IS NULL), rank_value, item;

# 교재 기준 P@5: 결과가 5개 미만이면 실제 반환 수를 분모로 사용
WITH evaluated_items AS (
    SELECT r.keyword, r.rank_value, r.item,
           CASE WHEN c.item IS NOT NULL THEN 1 ELSE 0 END AS correct_flag
    FROM w7_search_result AS r
    LEFT JOIN w7_correct_result AS c
      ON r.keyword = c.keyword AND r.item = c.item
    UNION ALL
    SELECT c.keyword, NULL AS rank_value, c.item, 1 AS correct_flag
    FROM w7_correct_result AS c
    LEFT JOIN w7_search_result AS r
      ON c.keyword = r.keyword AND c.item = r.item
    WHERE r.item IS NULL
), cumulative_stats AS (
    SELECT *,
        SUM(correct_flag) OVER (PARTITION BY keyword) AS correct_total,
        SUM(correct_flag) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_correct,
        COUNT(*) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_items
    FROM evaluated_items
), per_keyword AS (
    SELECT keyword,
        COALESCE(
            100.0 * SUM(CASE WHEN rank_value <= 5 THEN correct_flag ELSE 0 END)
            / NULLIF(SUM(CASE WHEN rank_value <= 5 THEN 1 ELSE 0 END), 0),
            0.0
        ) AS precision_at_5_pct
    FROM cumulative_stats
    GROUP BY keyword
)
SELECT keyword, ROUND(precision_at_5_pct, 2) AS precision_at_5_pct,
       ROUND(AVG(precision_at_5_pct) OVER (), 2) AS mean_precision_at_5_pct
FROM per_keyword
ORDER BY keyword;
```

![1-7 실행 결과](/image/week7/1-7.png)

### 1-8 검색 결과 순위와 관련된 지표 계산하기

- `AP` : 정답 위치에서의 정확률 합 / 전체 정답 수
- 미검색 정답의 기여도 0, 정답의 상위 배치에 높은 평가
- `MAP` : 키워드별 AP의 단순 평균, 중간값 반올림 없이 최종 출력에서 반올림

```sql
WITH evaluated_items AS (
    SELECT r.keyword, r.rank_value, r.item,
           CASE WHEN c.item IS NOT NULL THEN 1 ELSE 0 END AS correct_flag
    FROM w7_search_result AS r
    LEFT JOIN w7_correct_result AS c
      ON r.keyword = c.keyword AND r.item = c.item
    UNION ALL
    SELECT c.keyword, NULL AS rank_value, c.item, 1 AS correct_flag
    FROM w7_correct_result AS c
    LEFT JOIN w7_search_result AS r
      ON c.keyword = r.keyword AND c.item = r.item
    WHERE r.item IS NULL
), cumulative_stats AS (
    SELECT *,
        SUM(correct_flag) OVER (PARTITION BY keyword) AS correct_total,
        SUM(correct_flag) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_correct,
        COUNT(*) OVER (
            PARTITION BY keyword
            ORDER BY (rank_value IS NULL), rank_value, item
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_items
    FROM evaluated_items
), per_keyword AS (
    SELECT keyword,
        SUM(CASE WHEN rank_value IS NOT NULL AND correct_flag = 1
                 THEN 100.0 * cumulative_correct / NULLIF(cumulative_items, 0)
                 ELSE 0.0 END)
        / NULLIF(MAX(correct_total), 0) AS average_precision_pct
    FROM cumulative_stats
    GROUP BY keyword
)
SELECT keyword, ROUND(average_precision_pct, 2) AS average_precision_pct,
       ROUND(AVG(average_precision_pct) OVER (), 2) AS mean_average_precision_pct
FROM per_keyword
ORDER BY keyword;
```

![1-8 실행 결과](/image/week7/1-8.png)


## 2. 데이터마이닝 
### 2-1 Associaion Rule

- 거래별 상품 중복 제거 → 자기 결합 → 방향성 있는 두 상품 페어 집계
- `support` : 동시 구매 / 전체 거래, 확신도 : 동시 구매 / 선행 상품 구매
- `lift` : 확신도 / 후행 상품 기본 구매 확률, 동시 구매의 인과관계 단정 주의

```sql
WITH basket_items AS (
    SELECT DISTINCT purchase_id, product_id
    FROM w7_purchase_detail_log
), transaction_total AS (
    SELECT COUNT(DISTINCT purchase_id) AS purchase_count
    FROM basket_items
), product_counts AS (
    SELECT product_id, COUNT(*) AS product_count
    FROM basket_items
    GROUP BY product_id
), pair_counts AS (
    SELECT a.product_id AS product_a, b.product_id AS product_b,
           COUNT(*) AS pair_count
    FROM basket_items AS a
    JOIN basket_items AS b ON a.purchase_id = b.purchase_id
      AND a.product_id <> b.product_id
    GROUP BY a.product_id, b.product_id
)
SELECT
    p.product_a, p.product_b, p.pair_count,
    a.product_count AS product_a_count,
    b.product_count AS product_b_count,
    t.purchase_count,
    ROUND(100.0 * p.pair_count / NULLIF(t.purchase_count, 0), 4) AS support_pct,
    ROUND(100.0 * p.pair_count / NULLIF(a.product_count, 0), 4) AS confidence_pct,
    ROUND(
        1.0 * p.pair_count * t.purchase_count
        / NULLIF(a.product_count * b.product_count, 0),
        4
    ) AS lift_value
FROM pair_counts AS p
JOIN product_counts AS a ON p.product_a = a.product_id
JOIN product_counts AS b ON p.product_b = b.product_id
CROSS JOIN transaction_total AS t
ORDER BY p.product_a, p.product_b;
```

![2-1 실행 결과](/image/week7/2-1.png)


한 학기 동안 다소 부족한 커리큘럼이었음에도 불구하고 성실하게, 그리고 열심히 과제를 수행해주셔서 진심으로 감사드립니다.

이번 과정을 통해 배운 내용이 앞으로 여러분이 SQL을 다루는 데에 조금이나마 도움이 되기를 바랍니다.

여러분의 앞날을 진심으로 응원합니다. 😊


### 🎉 수고하셨습니다.
