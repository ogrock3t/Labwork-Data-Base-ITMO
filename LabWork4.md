# **Лабораторная работа №4**

### Задание 1

Найти долю продаж каждого продукта (цена продукта × количество продукта) по каждому чеку, в денежном выражении.

```sql
SELECT sales_order_id,
	product_id,
	(unit_price * order_qty) * 1.0 / SUM(unit_price * order_qty) 
	OVER (PARTITION BY sales_order_id) AS sales_share
FROM sales.sales_order_detail
```

### Задание 2

Вывести список продуктов, их стоимость, а также разницу между стоимостью этого продукта и стоимостью самого дешевого продукта в той же подкатегории, к которой относится продукт 53.

```sql
SELECT
	product_id,
	list_price,
	list_price - (
		SELECT FIRST_VALUE(list_price) OVER(PARTITION BY product_subcategory_id ORDER BY list_price ASC)
		FROM production.product
		WHERE product_subcategory_id = (
			SELECT product_subcategory_id FROM production.product
			WHERE product_id = 53
		)
	) AS price_difference
FROM production.product
```

### Задание 3

Вывести три колонки: номер покупателя, номер чека покупателя (отсортированный по возрастанию даты чека) и искусственно введенный порядковый номер текущего чека, начиная с 1, для каждого покупателя.

```sql
SELECT customer_id,
	sales_order_id,
	ROW_NUMBER () 
	OVER (PARTITION BY customer_id ORDER BY order_date) AS sequence_number
FROM sales.sales_order_header
ORDER BY customer_id, order_date
```

### Задание 4

Вывести номера продуктов, таких что их цена выше средней цены продукта в подкатегории, к которой относится продукт. Запрос реализовать двумя способами. В одном из решений допускается использование обобщенного табличного выражения (WITH).

```sql
SELECT product_id
FROM (
	SELECT product_id,
		list_price,
		product_subcategory_id,
		AVG(list_price) OVER (PARTITION BY product_subcategory_id) AS avg_price
	FROM production.product
)
WHERE list_price > avg_price
```

```sql
WITH product_info AS (
	SELECT product_id,
		list_price,
		product_subcategory_id,
		AVG(list_price) OVER (PARTITION BY product_subcategory_id) AS avg_price
	FROM production.product
)

SELECT product_id
FROM product_info
WHERE list_price > avg_price
```

### Задание 5

Вывести номер продукта, название продукта, а также среднее количество этого продукта по трём последним по дате чекам, в которых был этот продукт. Использовать оконные рамки (ROWS BETWEEN).

```sql
SELECT
	p.product_id,
	p.name,
	(
		SELECT AVG(sod.order_qty)
		OVER(
			PARTITION BY sod.product_id
			ORDER BY soh.order_date DESC
			ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
		)
		FROM sales.sales_order_detail AS sod
		JOIN sales.sales_order_header AS soh
		ON sod.sales_order_id = soh.sales_order_id
		WHERE p.product_id = sod.product_id
		LIMIT 1
	) AS recent_average_quantity
FROM production.product AS p
```

## Таски, которые были на защите

Найти 5 самых продаваемых товаров

```sql
SELECT product_id
FROM (
	SELECT DISTINCT product_id,
		COUNT(product_id) OVER (PARTITION BY product_id) as sales_count
	FROM sales.sales_order_detail
)
ORDER BY sales_count DESC
LIMIT 5
```

Вывести продуктовое имя, имя категории, листовой прайс, и сумму продаж всех товаров этой категории до выхода этого товара в продажу

```sql
SELECT
    p.name AS product_name,
    pc.name AS category_name,
    p.list_price,
    SUM(sod.line_total) OVER (
        PARTITION BY pc.product_category_id
        ORDER BY p.sell_start_date
        ROWS UNBOUNDED PRECEDING
    ) AS category_sales_before_product
FROM production.product p
JOIN production.product_subcategory ps ON p.product_subcategory_id = ps.product_subcategory_id
JOIN production.product_category pc ON ps.product_category_id = pc.product_category_id
JOIN sales.sales_order_detail sod ON p.product_id = sod.product_id
JOIN sales.sales_order_header soh ON sod.sales_order_id = soh.sales_order_id
WHERE soh.order_date <= p.sell_start_date
```