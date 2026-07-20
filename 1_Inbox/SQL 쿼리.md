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