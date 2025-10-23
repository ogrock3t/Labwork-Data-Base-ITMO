# **Лабораторная работа №3**

### Задание 1

Найти название самого продаваемого продукта.

Мое решение, которое я считаю верным, но не совпадает ответ.
```sql
SELECT name
FROM production.product

WHERE product_id = (
	SELECT product_id 
	FROM sales.sales_order_detail

	GROUP BY product_id
	ORDER BY SUM(order_qty) DESC
	LIMIT 1
)
```

Решение, где ответ совпадает.
```sql
SELECT name
FROM production.product

WHERE product_id = (
	SELECT product_id
	FROM sales.sales_order_detail

	GROUP BY product_id
	ORDER BY COUNT(*) DESC
	LIMIT 1
)
```

### Задание 2

Найти покупателя, совершившего покупку на самую большую сумму, считая сумму покупки исходя из цены товара без скидки (unit_price).

```sql
SELECT customer_id
FROM sales.customer

WHERE customer_id = (
	SELECT customer_id 
	FROM sales.sales_order_header

	WHERE sales_order_id = (
		SELECT sales_order_id
		FROM sales.sales_order_detail

		GROUP BY sales_order_id
		ORDER BY SUM(unit_price * order_qty) DESC
		LIMIT 1
	)
)
```

### Задание 3

Найти такие продукты, которые покупал только один покупатель.

```sql
SELECT name 
FROM production.product

WHERE product_id IN (
	SELECT SOD.product_id 
	FROM sales.sales_order_detail AS SOD

	GROUP BY product_id
	HAVING COUNT(DISTINCT (
		SELECT customer_id 
		FROM sales.sales_order_header as SOH

		WHERE SOD.sales_order_id = SOH.sales_order_id
	)) = 1
)
```

### Задание 4

Вывести список продуктов, цена которых выше средней цены товаров в подкатегории, к которой относится товар.

```sql
SELECT name
FROM production.product AS a

WHERE list_price > (
	SELECT AVG(list_price) 
	FROM production.product AS b

	WHERE a.product_subcategory_id = b.product_subcategory_id
)
```

### Задание 5

Найти такие товары, которые были куплены более чем одним покупателем, при этом все покупатели этих товаров покупали товары только одного цвета, и эти товары не входят в список покупок покупателей, купивших товары только двух цветов.

```sql

```

### Задание 6

Найти такие товары, которые были куплены такими покупателями, у которых они присутствовали в каждой их покупке.

```sql
SELECT name FROM production.product
WHERE product_id IN (
    SELECT product_id FROM (
        SELECT sod.product_id,
  (
   SELECT customer_id FROM sales.sales_order_header
   WHERE sales_order_id = sod.sales_order_id
  ) as customer_id,
        COUNT(DISTINCT sod.sales_order_id) as order_count_with_product
        FROM sales.sales_order_detail sod
        GROUP BY sod.product_id, customer_id
    ) product_customers
    WHERE order_count_with_product = (
        SELECT COUNT(*)
        FROM sales.sales_order_header soh
        WHERE soh.customer_id = product_customers.customer_id
    )
)
```

### Задание 7

Найти покупателей, у которых есть товар, присутствующий в каждой покупке/чеке.

```sql
WITH customer_total_orders AS (
  SELECT customer_id,
         COUNT(DISTINCT sales_order_id) AS total_orders
  FROM sales.sales_order_header
  GROUP BY customer_id
),
cust_prod_orders AS (
  SELECT h.customer_id,
         d.product_id,
         COUNT(DISTINCT d.sales_order_id) AS orders_with_product
  FROM sales.sales_order_detail d,
       sales.sales_order_header h
  WHERE d.sales_order_id = h.sales_order_id
  GROUP BY h.customer_id, d.product_id
)
SELECT DISTINCT cpo.customer_id
FROM cust_prod_orders cpo
WHERE cpo.orders_with_product = (
  SELECT cto.total_orders
  FROM customer_total_orders cto
  WHERE cto.customer_id = cpo.customer_id
)
```

### Задание 8

Найти такой товар или товары, которые были куплены не более чем тремя различными покупателями.

```sql
SELECT name
FROM production.product

WHERE product_id IN (
	SELECT SOD.product_id 
	FROM sales.sales_order_detail AS SOD

	GROUP BY product_id
	HAVING COUNT(DISTINCT (
		SELECT SOH.customer_id
		FROM sales.sales_order_header AS SOH

		WHERE SOD.sales_order_id = SOH.sales_order_id
	)) <= 3
)
```

### Задание 9

Найти все товары, такие что их покупали всегда с товаром, цена которого максимальна в своей категории.

```sql

```

### Задание 10

Найти номера тех покупателей, у которых есть как минимум два чека, и каждый из этих чеков содержит как минимум три товара, каждый из которых был куплен другими покупателями как минимум три раза.

```sql

```

### Задание 11

Найти все чеки, в которых каждый товар был куплен дважды этим же покупателем.

```sql
SELECT sales_order_id FROM sales.sales_order_header AS soh
WHERE NOT EXISTS(
 SELECT 1 FROM sales.sales_order_detail AS sod
 WHERE soh.sales_order_id = sod.sales_order_id 
 AND 2 != (
  SELECT COUNT(*) FROM sales.sales_order_detail AS sod2
  WHERE sod.product_id = sod2.product_id
  AND EXISTS(
   SELECT 1 FROM sales.sales_order_header AS soh2
   WHERE soh2.sales_order_id = sod2.sales_order_id
   AND soh2.customer_id = soh.customer_id
  )
 )
)
```

### Задание 12

Найти товары, которые были куплены минимум три раза различными покупателями.

```sql
SELECT name
FROM production.product

WHERE product_id IN (
	SELECT SOD.product_id 
	FROM sales.sales_order_detail AS SOD

	GROUP BY product_id
	HAVING COUNT(DISTINCT (
		SELECT SOH.customer_id
		FROM sales.sales_order_header AS SOH

		WHERE SOD.sales_order_id = SOH.sales_order_id
	)) >= 3
)
```

### Задание 13

Найти такую подкатегорию или подкатегории товаров, которые содержат более трёх товаров, купленных более трёх раз.

```sql
SELECT product_subcategory_id
FROM production.product

WHERE product_id IN (
	SELECT product_id
	FROM sales.sales_order_detail

	GROUP BY product_id
	HAVING COUNT(*) > 3
)

GROUP BY product_subcategory_id
HAVING COUNT(*) > 3
```

### Задание 14

Найти те товары, которые не были куплены более трёх раз, и как минимум дважды одним и тем же покупателем.

```sql
SELECT name
FROM production.product
WHERE product_id IN (
	SELECT product_id 
	FROM sales.sales_order_detail
	GROUP BY product_id
	HAVING COUNT(DISTINCT sales_order_id) <= 3
	AND EXISTS (
		SELECT 1
		FROM sales.sales_order_detail as SOD
		WHERE SOD.product_id = sales_order_detail.product_id
		GROUP BY (
			SELECT customer_id
			FROM sales.sales_order_header as SOH
			WHERE SOH.sales_order_id = SOD.sales_order_id
		)
		HAVING COUNT(*) >= 2
	)
)
```

### Задание 15

Найти среднее количество покупок на чек для каждого покупателя (2 способа).

```sql
WITH customer_info AS (
	SELECT customer_id,
		(SELECT COUNT(DISTINCT SOD.product_id) 
		FROM sales.sales_order_detail as SOD
		WHERE SOD.sales_order_id = sales_order_id) AS count
	FROM sales.sales_order_header
)

SELECT customer_id, AVG(count) FROM customer_info
GROUP by customer_id
```

### Задание 16

Найти для каждого продукта и каждого покупателя соотношение количества фактов покупки данного товара данным покупателем к общему количеству фактов покупки товаров данным покупателем.


```sql

```

### Задание 17

Вывести информацию: название продукта, общее количество фактов покупки этого продукта, общее количество покупателей этого продукта.

```sql
SELECT name, (
	SELECT COUNT(*)
	FROM sales.sales_order_detail AS SOD

	WHERE P.product_id = SOD.product_id
) AS sale_count,
(
	SELECT COUNT(DISTINCT customer_id)
	FROM sales.sales_order_header as SOH

	WHERE SOH.sales_order_id IN (
		SELECT SOD2.sales_order_id
		FROM sales.sales_order_detail AS SOD2

		WHERE SOD2.product_id = P.product_id
	)
) as customer_count
FROM production.product as P
```

### Задание 18

Вывести для каждого покупателя информацию о максимальной и минимальной стоимости одной покупки (чека): номер покупателя, максимальная сумма, минимальная сумма.

```sql
SELECT SOH.customer_id, 
	(SELECT MIN(total_due) FROM sales.sales_order_header WHERE customer_id = SOH.customer_id),
	(SELECT MAX(total_due) FROM sales.sales_order_header WHERE customer_id = SOH.customer_id)
FROM sales.sales_order_header as SOH
GROUP BY SOH.customer_id
```

### Задание 19

Найти номера покупателей, у которых нет ни одной пары чеков с одинаковым количеством наименований товаров.

```sql
SELECT t.customer_id FROM (
	SELECT SOH.customer_id AS customer_id,
		SOH.sales_order_id as sales_order_id,
		(
			SELECT COUNT(DISTINCT SOD.product_id)
			FROM sales.sales_order_detail AS SOD
			WHERE SOH.sales_order_id = SOD.sales_order_id
		) as product_count
	FROM sales.sales_order_header AS SOH
) as t
GROUP BY t.customer_id
HAVING COUNT(*) = COUNT(DISTINCT t.product_count)
```

### Задание 20

Найти номера покупателей, у которых все купленные ими товары были куплены как минимум дважды (на два разных чека).

```sql
WITH customer_info AS (
	SELECT SOH2.customer_id, (
		SELECT COUNT(DISTINCT SOD.sales_order_id)
		FROM sales.sales_order_detail AS SOD

		JOIN sales.sales_order_header as SOH
		ON SOD.sales_order_id = SOH.sales_order_id

		WHERE SOD2.product_id = SOD.product_id
		AND SOH2.customer_id = SOH.customer_id
	) as orders_count
	FROM sales.sales_order_header as SOH2

	JOIN sales.sales_order_detail as SOD2
	ON SOH2.sales_order_id = SOD2.sales_order_id
)

SELECT customer_id 
FROM customer_info
GROUP BY customer_id
HAVING MIN(orders_count) >= 2
```

## Таски, которые были на защите

Для каждого customer вывести количество его кол во чеков и продуктов для этих чеков без join без ОТВ

```sql
SELECT SOH.customer_id, 
COUNT(SOH.sales_order_id), 
SUM((
 SELECT COUNT(DISTINCT product_id)
 FROM sales.sales_order_detail as SOD
 WHERE SOD.sales_order_id = SOH.sales_order_id
))
FROM sales.sales_order_header AS SOH
GROUP BY customer_id
```

Айди чеков у которых есть товары которые относится к двум разным подкатегориям

```sql
WITH sales_info (sales_order_id, subcategory_count) AS (
 SELECT SOD.sales_order_id, 
  COUNT(DISTINCT P.product_subcategory_id) AS subcategory_count
 FROM sales.sales_order_detail as SOD

 JOIN production.product as P
 ON P.product_id = SOD.product_id

 GROUP BY SOD.sales_order_id
)

SELECT sales_order_id, subcategory_count
FROM sales_info
WHERE subcategory_count >= 2
```

Найти все продукты которые продавались больше 3 раз различным покупателям без JOIN и WITH (12 таска)

```sql
SELECT name
FROM production.product

WHERE product_id IN (
 SELECT SOD.product_id 
 FROM sales.sales_order_detail AS SOD

 GROUP BY product_id
 HAVING COUNT(DISTINCT (
  SELECT SOH.customer_id
  FROM sales.sales_order_header AS SOH

  WHERE SOD.sales_order_id = SOH.sales_order_id
 )) >= 3
)
```