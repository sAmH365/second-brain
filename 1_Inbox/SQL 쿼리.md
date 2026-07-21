## 중고급 서브쿼리
### 1. 간단 서브쿼리
```sql
SELECT first_name, last_name  
FROM customer  
WHERE customer_id IN (  
    SELECT customer_id  
    FROM payment  
    WHERE amount > (SELECT AVG(amount) FROM payment)  
    );
```
### 2. GROUP BY 서브쿼리
```sql
SELECT  
    first_name, last_name  
FROM customer  
WHERE customer_id IN (  
 SELECT customer_id  
 FROM payment  
 GROUP BY customer_id  
 HAVING COUNT(*) > (  
  SELECT AVG(payment_count)  
  FROM (  
   SELECT COUNT(*) AS payment_count  
   FROM payment  
   GROUP BY customer_id  
   ) AS payment_counts  
 )  
)
```
* `FROM` 절 서브 쿼리는 `AS` 로 이름 지어줘야 함
### 3. 최대값 가진 행 찾기
```sql
SELECT first_name, last_name  
FROM customer  
WHERE customer_id = (  
  SELECt customer_id  
  FROM (  
   SELECT  
       customer_id,  
       COUNT(*) AS payment_count  
   FROM payment  
   GROUP BY customer_id  
  ) AS payment_counts  
  ORDER BY payment_count DESC  
  LIMIT 1  
);
```
### 4. 상관 서브쿼리
```sql
SELECT  
  p.customer_id, p.amount, p.payment_date  
  FROM payment p 
  WHERE p.amount > (  
	  SELECT AVG(amount)  
	  FROM payment  
	  WHERE customer_id = p.customer_id  
);
```
* 밖에 있는 컬럼을 내부 쿼리에서 사용
## 집합 SQL
### 1. UNION
* 두 개 이상의 SELECT 문 결과 집합을 결합, 중복된 행은 제거
* 각 SELECT 문의 열은 같은 순서를 가져야하고, 유사한 데이터 유형을 가져야한다.
```sql
SELECT film_id FROM film  
UNION  
SELECT film_id FROM inventory
```
### 2. UNION ALL
* UNION 결과에 중복을 포함 시킬 때 사용
```sql
SELECT film_id FROM film  
UNION  
SELECT film_id FROM inventory
```
### 3. INTERSECT
* 교집합 -> 모든 SELECT 문에 공통적으로 있는 행 반환
```sql
SELECT film_id FROM film  
INTERSECT  
SELECT film_id FROM inventory
```
### 4.EXCEPT
* 차집합 -> A - B (A에만 있는거)
```sql
SELECT film_id FROM film  
EXCEPT  
SELECT film_id FROM inventory;
```

## SQL VIEW
* 실제 테이블을 기반으로 한 가상 테이블, 복잡한 쿼리를 단순화하고, 데이터의 특정 부분에만 접근하게 할 수 있다.
```sql
CREATE OR REPLACE VIEW ActorInfo AS  
SELECT first_name, last_name  
FROM actor  
WHERE actor_id < 100;
```
```sql
DROP VIEW ActorInfo;
```

## `WITH`, `CASE` `WHEN`
### `WITH`절
* Common Table Expression (CTEs)
* 일시적인 ResultSet을 만들어 `SELECT`, `INSERT`, `UPDEATE`, `DELETE` 문에서 참조하게 해줌
* 단일 SQL 쿼리 내에서만 사용 가능 -> 쿼리 종료되면 삭제
```sql
WITH FilmInventory AS (
  SELECT DISTINCT film_id FROM inventory
)
SELECT f.film_id, f.title
FROM film f
JOIN FilmInventory fi ON f.film_id = fi.film_id;
```
### `CASE` `WHEN`
```sql
SELECT title,
CASE
  WHEN rental_rate < 1 THEN 'Cheap'
  WHEN rental_rate BETWEEN 1 AND 3 THEN 'Moderate'
  ELSE 'Expensive'
END AS PriceCategory
FROM film;
```
## `GROUP_CONCAT`
```sql
SELECT  
    c.customer_id,  
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,  
    GROUP_CONCAT(f.title ORDER BY f.title ASC) AS rented_movies  
FROM customer c  
JOIN rental r ON c.customer_id = r.customer_id  
JOIN inventory i ON r.inventory_id = i.inventory_id  
JOIN film f ON i.film_id = f.film_id  
GROUP BY c.customer_id  
LIMIT 5;

SELECT  
    c.customer_id,  
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,  
    GROUP_CONCAT(f.title ORDER BY f.title ASC SEPARATOR '::') AS rented_movies  
FROM customer c  
JOIN rental r ON c.customer_id = r.customer_id  
JOIN inventory i ON r.inventory_id = i.inventory_id  
JOIN film f ON i.film_id = f.film_id  
GROUP BY c.customer_id  
LIMIT 5;
```
## MySQL 윈도우함수
* MySQL 8버전부터 윈도우 함수 지원
* 윈도우 함수
	* SQL 쿼리 내에서 데이터 집합을 세분화하여 각 부분에 대한 계산을 수행하는 함수
		* 데이터 순위 매기기, 집계, 이동 평균, 누적 합 등 계산 가능 -> 기본 집계함수보다 유연한 데이터 분석을 가능하게함
	* 이 함수들은 특정 '윈도우'(데이터의 부분 집합) 내에서 작동하며, 각각의 행에 대해 결과를 반환하되, 전체 쿼리 결과의 컨텍스트 내에서 실행됨
	* **행 기준 연산**: 각 행에 대해 연산을 수행하면서, 원본 행의 구조를 유지
	* **부분 데이터 집합 사용** : 특정 윈도우 내에서 연산을 수행하고, 이는 `PARTITION BY` 절을 통해 더 세분화 가능
*  행과 행 사이의 관계를 정의하고 계산하기 위해 사용하는 함수. `GROUP BY`처럼 데이터를 그룹별로 묶어서 집계하지만, **기존 행의 개수를 줄이지 않고 유지하며 집계 값을 각 행에 그대로 붙여준다**는 결정적인 차이점
	* ```sql
1. GROUP BY
	SELECT 부서, SUM(급여) FROM 사원 GROUP BY 부서;
	
	부서    | SUM(급여)
	개발900 | 기획400
	
	2. 윈도우 함수
	SELECT 이름, 부서, 급여, SUM(급여) OVER(PARTITION BY 부서) AS 부서총합 FROM 사원;
	|이름|부서|급여|부서총합|
    |김철수|개발|500|900|
    |이영희|개발|400|900|
    |박민수|기획|400|400|
	  ```
## RANK(), DENSE_RANK(), ROW_NUMBER() 문법
### 1. RANK()
* 순위를 매기되, 동일한 값이 있을 경우 같은 순위를 부여하고 다음 순위는 건너 뜀
```sql
RANK() OVER (ORDER BY column_name [ASC | DESC])
```

### 2. DENSE_RANK()
* 순위를 매기되, 동일한 값이 있을 경우 같은 순위를 부여하지만 다음 순위는 건너 뛰지 않음
```sql
DENSE_RANK() OVER (ORDER BY column_name [ASC | DESC])  
```

### 3. ROW_NUMBER()
* 순위와 상관없이 각 행에 고유한 번호를 부여
```sql
ROW_NUMBER() OVER (ORDER BY column_name [ASC | DESC])  
```

### EX
```sql
SELECT  
    title,  
    length,  
    RANK() OVER (ORDER BY length DESC) AS ranking,  
    DENSE_RANK() OVER (ORDER BY length DESC) AS dense_ranking,  
    ROW_NUMBER() OVER (ORDER BY length DESC) AS row_numbers  
FROM film  
ORDER BY length DESC;
```