# Тестовое задание от СБЕРа
## Задание 1

### mysql

```mysql
SELECT name
FROM employees
ORDER BY age ASC
LIMIT 3;
```

### postgresql

```postgresql
SELECT name
FROM employees
ORDER BY age ASC
FETCH FIRST 3 ROWS ONLY;

SELECT name, age 
FROM test 
WHERE age IN (SELECT MIN(age) FROM test) 
ORDER BY age 
FETCH FIRST 3 ROWS ONLY;
```

### sql server

```sql
SELECT TOP 3 name
FROM employees
ORDER BY age ASC;
```

### oracle

```oracle
SELECT * 
FROM (SELECT * 
      FROM   employees
      ORDER  BY age ASC) 
WHERE ROWNUM <= 3;
```

## Задание 2 
### postgresql

```postgresql
SELECT abonent, region_id, DATE(dttm) AS day
FROM (
  SELECT abonent, region_id, dttm,
         ROW_NUMBER() OVER (PARTITION BY abonent, DATE(dttm) ORDER BY dttm DESC) AS rn
  FROM calls
) 
WHERE rn = 1;
```

## Задание 3 
### postgresql

```postgresql
CREATE TABLE dict_item_prices (
    item_id      INTEGER,
    item_name    VARCHAR(150),
    item_price   NUMERIC(12,2),
    valid_from_dt DATE,
    valid_to_dt   DATE,
    PRIMARY KEY (item_id)
);

INSERT INTO dict_item_prices
SELECT
    item_id,
    item_name,
    item_price,
    created_dttm AS valid_from_dt,
    LEAD(created_dttm, 1, '9999-12-31'::DATE) OVER (
        PARTITION BY item_id
        ORDER BY created_dttm
    ) - INTERVAL '1 DAY' AS valid_to_dt
FROM item_prices;
```

## Задание 4 
### postgresql

```postgresql
SELECT 
  td.customer_id,
  SUM(td.item_number * ip.item_price) AS amount_spent_1m,
  MAX(ip.item_name) AS top_item_1m
FROM 
  transaction_details td
  JOIN dict_item_prices ip ON td.item_id = ip.item_id
WHERE 
  td.transaction_dttm >= (SYSDATE - INTERVAL '30' DAY)
GROUP BY 
  td.customer_id
HAVING 
  SUM(td.item_number * ip.item_price) > 0;
```

## Задание 5 
### postgresql

```postgresql
SELECT 
  date_trunc('month', post_date)  AS month_year,
  COUNT(*) AS num_posts,
  CONCAT(
    ROUND(
      ((COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(post_date, '%Y-%m-01'))) / LAG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(post_date, '%Y-%m-01'))) * 100,
      '%'
    ) AS increase_fraction
FROM social_network_posts
GROUP BY month_year
ORDER BY month_year ASC;
```
