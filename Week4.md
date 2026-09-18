# SQL_MASTER 4주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_4th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_4th_TIL

### 5장 사용자를 파악하기 위한 데이터 추출
#### 2. 시계열에 따른 사용자 전체의 상태 변화 찾기
#### 3. 시계열에 따른 사용자의 개별적인 행동 분석하기 


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.52~136   | ✅         |
| 2주차 | p.138~184  | ✅         |
| 3주차 | p.186~232 | ✅         |
| 4주차 | p.233~321 | ✅         |
| 5주차 | p.324~406 | 🍽️         |
| 6주차 | p.408~464 | 🍽️         |
| 7주차 | p.466~566 | 🍽️         |

<br>

<!-- 여기까진 그대로 둬 주세요-->


# 실습 

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **08_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 2. 시계열에 따른 사용자 전체의 상태 변화 찾기

### 2-1 등록 수의 추이와 경향 보기

등록자 수는 서비스의 **신규 유입 규모**를 확인하는 가장 기본적인 지표. 일별 등록 수만 보는 것보다 누적 등록자 수를 함께 확인하면 전체적인 성장 흐름을 파악하기 쉬움

- `COUNT()` : 기간별 등록자 수 집계
- 누적값 : 신규 유입이 시간에 따라 얼마나 쌓였는지 확인

```sql
WITH daily_register AS (
    SELECT
        register_date,
        COUNT(*) AS register_count
    FROM mst_users_registration
    GROUP BY register_date
)
SELECT
    register_date,
    register_count,
    SUM(register_count) OVER (
        ORDER BY register_date
    ) AS cumulative_count
FROM daily_register
ORDER BY register_date;
```

![2-1 실행 결과](/image/week4/2-1.png)

<br>
<br>

### 2-2 지속률과 정착률 산출하기

**지속률(Repeat Rate)** 과 **정착률(Retention Rate)** 은 모두 등록 이후에도 사용자가 서비스를 이용하는지 확인하는 지표지만 판정 방식이 다름

| 구분 | 판정 방식 |
| --- | --- |
| 지속률 | 등록 후 **특정 날짜**에 사용했는지 확인 |
| 정착률 | 등록 후 **특정 기간 안**에 한 번이라도 사용했는지 확인 |

⚠️ *아직 해당 판정 시점까지 시간이 지나지 않은 사용자는 집계 대상에서 제외!*

```sql
WITH interval_master AS (
    SELECT '01 day repeat' AS index_name, 1 AS begin_day, 1 AS end_day
    UNION ALL SELECT '03 day repeat', 3, 3
    UNION ALL SELECT '07 day repeat', 7, 7
    UNION ALL SELECT '07 day retention', 1, 7
    UNION ALL SELECT '14 day retention', 8, 14
    UNION ALL SELECT '28 day retention', 22, 28
),
latest_log AS (
    SELECT MAX(DATE(stamp)) AS latest_date
    FROM action_log_retention
),
user_index AS (
    SELECT
        u.user_id,
        i.index_name,
        i.begin_day,
        i.end_day,
        MAX(
            CASE
                WHEN DATE(a.stamp) BETWEEN
                    DATE_ADD(u.register_date, INTERVAL i.begin_day DAY)
                    AND DATE_ADD(u.register_date, INTERVAL i.end_day DAY)
                THEN 1
                ELSE 0
            END
        ) AS action_flag
    FROM mst_users_retention AS u
    CROSS JOIN interval_master AS i
    CROSS JOIN latest_log AS l
    LEFT JOIN action_log_retention AS a
        ON u.user_id = a.user_id
    WHERE l.latest_date >= DATE_ADD(
        u.register_date,
        INTERVAL i.end_day DAY
    )
    GROUP BY
        u.user_id,
        i.index_name,
        i.begin_day,
        i.end_day
)
SELECT
    index_name,
    COUNT(*) AS target_users,
    SUM(action_flag) AS active_users,
    ROUND(100.0 * AVG(action_flag), 2) AS rate
FROM user_index
GROUP BY
    index_name,
    begin_day,
    end_day
ORDER BY
    begin_day,
    end_day,
    index_name;
```

![2-2 실행 결과](/image/week4/2-2.png)

<br>
<br>

### 2-3 지속과 정착에 영향을 주는 액션 집계하기 

지속률이나 정착률이 변한 이유를 찾으려면 단순한 비율보다 **어떤 행동을 한 사용자와 하지 않은 사용자의 차이**를 비교해야 함

➡️ 모든 사용자와 액션의 조합을 만든 뒤 액션 수행 여부를 `0/1`로 나타내면, 특정 행동이 이후의 지속 사용과 어떤 관계가 있는지 비교 가능

```sql
WITH action_master AS (
    SELECT 'view' AS action
    UNION ALL SELECT 'comment'
    UNION ALL SELECT 'follow'
),
user_action AS (
    SELECT
        u.user_id,
        m.action,
        MAX(
            CASE
                WHEN a.action = m.action
                 AND DATE(a.stamp) = u.register_date
                THEN 1
                ELSE 0
            END
        ) AS used_action,
        MAX(
            CASE
                WHEN DATE(a.stamp) = DATE_ADD(
                    u.register_date,
                    INTERVAL 1 DAY
                )
                THEN 1
                ELSE 0
            END
        ) AS next_day_repeat
    FROM mst_users_retention AS u
    CROSS JOIN action_master AS m
    LEFT JOIN action_log_retention AS a
        ON u.user_id = a.user_id
    GROUP BY
        u.user_id,
        m.action
)
SELECT
    action,
    used_action,
    COUNT(*) AS users,
    ROUND(
        100.0 * AVG(next_day_repeat),
        2
    ) AS next_day_repeat_rate
FROM user_action
GROUP BY
    action,
    used_action
ORDER BY
    action,
    used_action DESC;
```

![2-3 실행 결과](/image/week4/2-3.png)
 

<br>
<br>

### 2-4 액션 수에 따른 정착률 집계하기 

액션은 단순히 **했는지 여부**뿐 아니라 **얼마나 많이 했는지**도 중요하다. 액션 횟수를 구간으로 나누고 각 구간의 정착률을 비교하면 행동량과 정착 사이의 관계를 확인 가능

*ex) 액션 횟수를 `0회`, `1~5회`, `6~10회`, `11회 이상`과 같이 구간화*

```sql
WITH action_bucket AS (
    SELECT 'comment' AS action, 0 AS min_count, 0 AS max_count
    UNION ALL SELECT 'comment', 1, 5
    UNION ALL SELECT 'comment', 6, 10
    UNION ALL SELECT 'comment', 11, 9999
    UNION ALL SELECT 'follow', 0, 0
    UNION ALL SELECT 'follow', 1, 5
    UNION ALL SELECT 'follow', 6, 10
    UNION ALL SELECT 'follow', 11, 9999
),
user_action_count AS (
    SELECT
        u.user_id,
        b.action,
        b.min_count,
        b.max_count,
        COUNT(a.action) AS action_count
    FROM mst_users_retention AS u
    CROSS JOIN action_bucket AS b
    LEFT JOIN action_log_retention AS a
        ON u.user_id = a.user_id
       AND a.action = b.action
       AND DATE(a.stamp) BETWEEN
           u.register_date
           AND DATE_ADD(
               u.register_date,
               INTERVAL 7 DAY
           )
    GROUP BY
        u.user_id,
        b.action,
        b.min_count,
        b.max_count
),
user_retention AS (
    SELECT
        u.user_id,
        MAX(
            CASE
                WHEN DATE(a.stamp) BETWEEN
                    DATE_ADD(u.register_date, INTERVAL 8 DAY)
                    AND DATE_ADD(u.register_date, INTERVAL 14 DAY)
                THEN 1
                ELSE 0
            END
        ) AS retained_14
    FROM mst_users_retention AS u
    LEFT JOIN action_log_retention AS a
        ON u.user_id = a.user_id
    GROUP BY u.user_id
)
SELECT
    c.action,
    CONCAT(c.min_count, ' ~ ', c.max_count) AS count_range,
    COUNT(*) AS users,
    ROUND(
        100.0 * AVG(r.retained_14),
        2
    ) AS retention_rate
FROM user_action_count AS c
JOIN user_retention AS r
    ON c.user_id = r.user_id
WHERE c.action_count BETWEEN c.min_count AND c.max_count
GROUP BY
    c.action,
    c.min_count,
    c.max_count
ORDER BY
    c.action,
    c.min_count;
```

![2-4 실행 결과](/image/week4/2-4.png)


<br>
<br>

### 2-5 사용 일수에 따른 정착률 집계하기 

같은 액션 수라도 여러 날에 걸쳐 사용한 사용자와 하루에 몰아서 사용한 사용자는 다를 수 있음 
➡️ **중복 날짜를 제거한 실제 사용 일수**를 기준으로 정착률을 비교할 수 있다.

- `COUNT(DISTINCT 날짜)` : 일정 기간 동안 사용자가 실제로 며칠 활동했는지 계산 가능

```sql
WITH user_usage AS (
    SELECT
        u.user_id,
        COUNT(
            DISTINCT CASE
                WHEN DATE(a.stamp) BETWEEN
                    DATE_ADD(u.register_date, INTERVAL 1 DAY)
                    AND DATE_ADD(u.register_date, INTERVAL 7 DAY)
                THEN DATE(a.stamp)
            END
        ) AS use_days,
        MAX(
            CASE
                WHEN DATE(a.stamp) BETWEEN
                    DATE_ADD(u.register_date, INTERVAL 22 DAY)
                    AND DATE_ADD(u.register_date, INTERVAL 28 DAY)
                THEN 1
                ELSE 0
            END
        ) AS retained_28
    FROM mst_users_retention AS u
    LEFT JOIN action_log_retention AS a
        ON u.user_id = a.user_id
    GROUP BY u.user_id
),
usage_summary AS (
    SELECT
        use_days,
        COUNT(*) AS users,
        SUM(retained_28) AS retained_users
    FROM user_usage
    GROUP BY use_days
)
SELECT
    use_days,
    users,
    ROUND(
        100.0 * users / SUM(users) OVER (),
        2
    ) AS user_ratio,
    retained_users,
    ROUND(
        100.0 * retained_users / NULLIF(users, 0),
        2
    ) AS retention_rate
FROM usage_summary
ORDER BY use_days;
```

![2-5 실행 결과](/image/week4/2-5.png)

<br>
<br>

### 2-6 사용자의 잔존율 집계하기 

✅ **잔존율** : 같은 시기에 등록한 사용자가 시간이 지난 뒤에도 얼마나 남아 있는지를 확인하는 코호트 지표

- `register_month` : 사용자가 처음 등록한 월
- `index_month` : 등록 후 잔존 여부를 확인할 월
- 잔존율 : 해당 월 활동 사용자 / 등록 코호트 사용자

등록 월별로 비교하면 사용자 집단마다 장기적인 유지 패턴이 어떻게 다른지 볼 수 있음

```sql
WITH RECURSIVE month_interval AS (
    SELECT 0 AS month_no
    UNION ALL
    SELECT month_no + 1
    FROM month_interval
    WHERE month_no < 12
),
user_month AS (
    SELECT
        u.user_id,
        DATE_FORMAT(u.register_date, '%Y-%m') AS register_month,
        DATE_FORMAT(
            DATE_ADD(
                u.register_date,
                INTERVAL i.month_no MONTH
            ),
            '%Y-%m'
        ) AS index_month,
        i.month_no
    FROM mst_users_retention AS u
    CROSS JOIN month_interval AS i
),
active_month AS (
    SELECT DISTINCT
        user_id,
        DATE_FORMAT(stamp, '%Y-%m') AS action_month
    FROM action_log_retention
),
latest_month AS (
    SELECT DATE_FORMAT(MAX(stamp), '%Y-%m') AS latest_month
    FROM action_log_retention
)
SELECT
    u.register_month,
    u.index_month,
    COUNT(*) AS cohort_users,
    SUM(
        CASE
            WHEN a.action_month IS NOT NULL THEN 1
            ELSE 0
        END
    ) AS active_users,
    ROUND(
        100.0 * AVG(
            CASE
                WHEN a.action_month IS NOT NULL THEN 1
                ELSE 0
            END
        ),
        2
    ) AS retention_rate
FROM user_month AS u
CROSS JOIN latest_month AS l
LEFT JOIN active_month AS a
    ON u.user_id = a.user_id
   AND u.index_month = a.action_month
WHERE u.index_month <= l.latest_month
GROUP BY
    u.register_month,
    u.index_month,
    u.month_no
ORDER BY
    u.register_month,
    u.month_no;
```

![2-6 실행 결과](/image/week4/2-6.png)

<br>
<br>


### 2-7 방문 빈도를 기반으로 사용자 속성을 정의하고 집계하기

월별 방문 이력을 이용하면 사용자를 방문 형태에 따라 구분 가능 

| 사용자 유형 | 의미 |
| --- | --- |
| New | 해당 월에 처음 방문한 사용자 |
| Repeat | 이전 달에 이어 이번 달에도 방문한 사용자 |
| Come Back | 한동안 방문하지 않다가 다시 방문한 사용자 |

➡️ 이렇게 분류하면 단순 MAU뿐 아니라 **신규·지속·복귀 사용자 구성**까지 함께 파악 가능!!

```sql
WITH activity_month AS (
    SELECT DISTINCT
        user_id,
        CAST(
            DATE_FORMAT(stamp, '%Y-%m-01')
            AS DATE
        ) AS action_month
    FROM action_log_retention
),
user_month AS (
    SELECT
        user_id,
        action_month,
        MIN(action_month) OVER (
            PARTITION BY user_id
        ) AS first_month,
        LAG(action_month) OVER (
            PARTITION BY user_id
            ORDER BY action_month
        ) AS previous_month
    FROM activity_month
),
user_type AS (
    SELECT
        user_id,
        action_month,
        CASE
            WHEN action_month = first_month
                THEN 'new'
            WHEN previous_month = DATE_SUB(
                action_month,
                INTERVAL 1 MONTH
            )
                THEN 'repeat'
            ELSE 'come_back'
        END AS user_type
    FROM user_month
)
SELECT
    DATE_FORMAT(action_month, '%Y-%m') AS action_month,
    COUNT(*) AS mau,
    SUM(
        CASE WHEN user_type = 'new' THEN 1 ELSE 0 END
    ) AS new_users,
    SUM(
        CASE WHEN user_type = 'repeat' THEN 1 ELSE 0 END
    ) AS repeat_users,
    SUM(
        CASE WHEN user_type = 'come_back' THEN 1 ELSE 0 END
    ) AS come_back_users
FROM user_type
GROUP BY action_month
ORDER BY action_month;
```

![2-7 실행 결과](/image/week4/2-7.png)

<br>
<br>

### 2-8 방문 종류를 기반으로 성장지수 집계하기 

- **성장지수** : 사용자 상태의 변화를 +와 -로 환산해 서비스가 성장하고 있는지 확인하는 지표

| 상태 변화 | 의미 | 부호 |
| --- | --- | ---: |
| Signup | 신규 사용 시작 | +1 |
| Reactivation | 비활성 → 활성 | +1 |
| Deactivation | 활성 → 비활성 | -1 |
| Exit | 탈퇴 또는 사용 종료 | -1 |

- `Signup + Reactivation - Deactivation - Exit`의 흐름을 통해 서비스의 순성장을 볼 수 있다.

```sql
WITH RECURSIVE calendar AS (
    SELECT MIN(register_date) AS target_date
    FROM mst_users_growth

    UNION ALL

    SELECT DATE_ADD(target_date, INTERVAL 1 DAY)
    FROM calendar
    WHERE target_date < (
        SELECT MAX(DATE(stamp))
        FROM action_log_growth
    )
),
user_daily AS (
    SELECT
        c.target_date,
        u.user_id,
        u.register_date,
        u.withdraw_date,
        MAX(
            CASE
                WHEN DATE(a.stamp) = c.target_date THEN 1
                ELSE 0
            END
        ) AS active_flag
    FROM calendar AS c
    CROSS JOIN mst_users_growth AS u
    LEFT JOIN action_log_growth AS a
        ON u.user_id = a.user_id
       AND DATE(a.stamp) = c.target_date
    WHERE c.target_date >= u.register_date
      AND (
          u.withdraw_date IS NULL
          OR c.target_date <= u.withdraw_date
      )
    GROUP BY
        c.target_date,
        u.user_id,
        u.register_date,
        u.withdraw_date
),
user_daily_with_prev AS (
    SELECT
        *,
        LAG(active_flag, 1, 0) OVER (
            PARTITION BY user_id
            ORDER BY target_date
        ) AS previous_active
    FROM user_daily
),
growth_type AS (
    SELECT
        target_date,
        user_id,
        CASE
            WHEN target_date = register_date
                THEN 'signup'
            WHEN target_date = withdraw_date
                THEN 'exit'
            WHEN active_flag = 1
             AND previous_active = 0
                THEN 'reactivation'
            WHEN active_flag = 0
             AND previous_active = 1
                THEN 'deactivation'
        END AS growth_type
    FROM user_daily_with_prev
)
SELECT
    target_date,
    SUM(
        CASE WHEN growth_type = 'signup' THEN 1 ELSE 0 END
    ) AS signup,
    SUM(
        CASE WHEN growth_type = 'reactivation' THEN 1 ELSE 0 END
    ) AS reactivation,
    SUM(
        CASE WHEN growth_type = 'deactivation' THEN -1 ELSE 0 END
    ) AS deactivation,
    SUM(
        CASE WHEN growth_type = 'exit' THEN -1 ELSE 0 END
    ) AS exit_count,
    SUM(
        CASE
            WHEN growth_type IN ('signup', 'reactivation') THEN 1
            WHEN growth_type IN ('deactivation', 'exit') THEN -1
            ELSE 0
        END
    ) AS growth_index
FROM growth_type
GROUP BY target_date
ORDER BY target_date;
```

![2-8 실행 결과](/image/week4/2-8.png)

<br>
<br>

### 2-9 지표 개선 방법 익히기 

⚠️ 지표를 만드는 것보다 중요한 것은 **지표를 실제 개선 행동으로 연결하는 것**

1. 개선하고 싶은 지표를 정한다.
2. 지표에 영향을 줄 것으로 보이는 행동을 정한다.
3. 해당 행동의 수행 여부·횟수에 따라 지표가 얼마나 달라지는지 비교한다.

즉, 분석의 목적은 단순한 현황 파악이 아니라 **어떤 행동을 유도해야 지표가 개선되는지 찾는 것!!**

```sql
WITH user_metric AS (
    SELECT
        u.user_id,
        MAX(
            CASE
                WHEN a.action = 'follow'
                 AND DATE(a.stamp) = u.register_date
                THEN 1
                ELSE 0
            END
        ) AS did_follow,
        MAX(
            CASE
                WHEN DATE(a.stamp) = DATE_ADD(
                    u.register_date,
                    INTERVAL 1 DAY
                )
                THEN 1
                ELSE 0
            END
        ) AS next_day_repeat
    FROM mst_users_retention AS u
    LEFT JOIN action_log_retention AS a
        ON u.user_id = a.user_id
    GROUP BY u.user_id
)
SELECT
    did_follow,
    COUNT(*) AS users,
    ROUND(
        100.0 * AVG(next_day_repeat),
        2
    ) AS next_day_repeat_rate
FROM user_metric
GROUP BY did_follow
ORDER BY did_follow DESC;
```

![2-9 실행 결과](/image/week4/2-9.png)

<br>
<br>

## 3. 시계열에 따른 사용자의 개별적인 행동 분석하기 

### 3-1 사용자의 액션 간격 집계하기

사용자의 두 행동 사이에 걸린 시간을 **리드 타임(Lead Time)** 으로 계산하면 행동 간격을 정량적으로 비교 가능

*+) 날짜 간격은 `DATEDIFF()`로 계산할 수 있으며, 구매·예약·신청처럼 여러 단계가 있는 서비스에서 각 단계까지 얼마나 시간이 걸리는지 파악하는 데 활용가능함*

```sql
WITH reservations AS (
    SELECT
        1 AS reservation_id,
        CAST('2016-09-01' AS DATE) AS register_date,
        CAST('2016-10-01' AS DATE) AS visit_date,
        3 AS days

    UNION ALL
    SELECT 2, '2016-09-20', '2016-10-01', 2

    UNION ALL
    SELECT 3, '2016-09-30', '2016-11-20', 2

    UNION ALL
    SELECT 4, '2016-10-01', '2017-01-03', 2

    UNION ALL
    SELECT 5, '2016-11-01', '2016-12-28', 3
)
SELECT
    reservation_id,
    register_date,
    visit_date,
    DATEDIFF(
        visit_date,
        register_date
    ) AS lead_time
FROM reservations
ORDER BY reservation_id;
```

![3-1 실행 결과](/image/week4/3-1.png)

<br>
<br>

### 3-2 카트 추가 후에 구매했는지 파악하기 

- **카트 탈락(Cart Abandonment)** : 카트에 상품을 넣은 뒤 구매하지 않는 현상
- 단순 구매 여부뿐 아니라 카트 추가 후 `1시간`, `6시간`, `24시간`, `48시간` 등 시간 구간별 구매 전환을 보면 구매까지 걸리는 시간을 함께 파악 가능
- 상품 단위로 카트와 구매 로그를 연결해야 같은 사용자의 다른 상품 구매가 잘못 매칭되는 것을 막을 수 있음

```sql
WITH cart_product AS (
    SELECT
        l.dt,
        l.user_id,
        l.`session`,
        jt.product_id,
        l.stamp AS cart_time
    FROM action_log_cart AS l
    CROSS JOIN JSON_TABLE(
        CONCAT(
            '["',
            REPLACE(l.products, ',', '","'),
            '"]'
        ),
        '$[*]' COLUMNS(
            product_id VARCHAR(50) PATH '$'
        )
    ) AS jt
    WHERE l.action = 'add_cart'
),
purchase_product AS (
    SELECT
        l.user_id,
        jt.product_id,
        l.stamp AS purchase_time
    FROM action_log_cart AS l
    CROSS JOIN JSON_TABLE(
        CONCAT(
            '["',
            REPLACE(l.products, ',', '","'),
            '"]'
        ),
        '$[*]' COLUMNS(
            product_id VARCHAR(50) PATH '$'
        )
    ) AS jt
    WHERE l.action = 'purchase'
),
cart_to_purchase AS (
    SELECT
        c.dt,
        c.user_id,
        c.`session`,
        c.product_id,
        c.cart_time,
        MIN(p.purchase_time) AS purchase_time
    FROM cart_product AS c
    LEFT JOIN purchase_product AS p
        ON c.user_id = p.user_id
       AND c.product_id = p.product_id
       AND p.purchase_time >= c.cart_time
    GROUP BY
        c.dt,
        c.user_id,
        c.`session`,
        c.product_id,
        c.cart_time
)
SELECT
    dt,
    COUNT(*) AS add_cart,
    SUM(
        CASE
            WHEN TIMESTAMPDIFF(
                MINUTE,
                cart_time,
                purchase_time
            ) <= 60
            THEN 1
            ELSE 0
        END
    ) AS purchase_1_hour,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN TIMESTAMPDIFF(
                    MINUTE,
                    cart_time,
                    purchase_time
                ) <= 60
                THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS purchase_1_hour_rate,
    SUM(
        CASE
            WHEN TIMESTAMPDIFF(
                HOUR,
                cart_time,
                purchase_time
            ) <= 6
            THEN 1
            ELSE 0
        END
    ) AS purchase_6_hours,
    SUM(
        CASE
            WHEN TIMESTAMPDIFF(
                HOUR,
                cart_time,
                purchase_time
            ) <= 24
            THEN 1
            ELSE 0
        END
    ) AS purchase_24_hours,
    SUM(
        CASE
            WHEN TIMESTAMPDIFF(
                HOUR,
                cart_time,
                purchase_time
            ) <= 48
            THEN 1
            ELSE 0
        END
    ) AS purchase_48_hours,
    SUM(
        CASE
            WHEN purchase_time IS NULL
              OR TIMESTAMPDIFF(
                    HOUR,
                    cart_time,
                    purchase_time
                 ) > 48
            THEN 1
            ELSE 0
        END
    ) AS not_purchase
FROM cart_to_purchase
GROUP BY dt
ORDER BY dt;
```

![3-2 실행 결과](/image/week4/3-2.png)

<br>
<br>

### 3-3 등록으로부터의 매출을 날짜별로 집계하기 

사용자 획득에는 광고·제휴 등의 비용이 들기 때문에 등록 이후 일정 기간 동안 발생한 **1인당 매출**을 함께 살펴보는 것이 중요함

➡️ 등록일을 기준으로 `30일`, `45일`, `60일`과 같은 동일한 관찰 기간을 두고 매출을 비교하면 코호트별 수익성을 공정하게 비교할 수 있으며, 장기적으로는 ARPU나 LTV 분석으로 확장할 수 있다

```sql
WITH interval_master AS (
    SELECT '30 day sales amount' AS index_name, 30 AS end_day
    UNION ALL SELECT '45 day sales amount', 45
    UNION ALL SELECT '60 day sales amount', 60
),
latest_log AS (
    SELECT MAX(DATE(stamp)) AS latest_date
    FROM action_log_sales
),
user_sales AS (
    SELECT
        DATE_FORMAT(
            u.register_date,
            '%Y-%m'
        ) AS register_month,
        i.index_name,
        i.end_day,
        u.user_id,
        SUM(
            CASE
                WHEN a.action = 'purchase'
                 AND DATE(a.stamp) BETWEEN
                     u.register_date
                     AND DATE_ADD(
                         u.register_date,
                         INTERVAL i.end_day DAY
                     )
                THEN COALESCE(a.amount, 0)
                ELSE 0
            END
        ) AS sales_amount
    FROM mst_users_sales AS u
    CROSS JOIN interval_master AS i
    CROSS JOIN latest_log AS l
    LEFT JOIN action_log_sales AS a
        ON u.user_id = a.user_id
    WHERE l.latest_date >= DATE_ADD(
        u.register_date,
        INTERVAL i.end_day DAY
    )
    GROUP BY
        DATE_FORMAT(u.register_date, '%Y-%m'),
        i.index_name,
        i.end_day,
        u.user_id
)
SELECT
    register_month,
    index_name,
    COUNT(*) AS register_users,
    SUM(sales_amount) AS total_sales,
    ROUND(
        AVG(sales_amount),
        2
    ) AS sales_per_user
FROM user_sales
GROUP BY
    register_month,
    index_name,
    end_day
ORDER BY
    register_month,
    end_day;
```

![3-3 실행 결과](/image/week4/3-3.png)



### 🎉 수고하셨습니다.