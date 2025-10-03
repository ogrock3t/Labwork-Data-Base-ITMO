# **Лабораторная работа №2**

### Задание 1

Найти и вывести на экран название продуктов и название категорий товаров, к которым относится этот продукт, с учетом того, что в выборку попадут только товары с цветом Red и ценой не менее 100.

```sql
SELECT P.name, PC.name
FROM production.product AS P

JOIN production.product_subcategory AS PSC
ON P.product_subcategory_id = PSC.product_subcategory_id

JOIN production.product_category AS PC
ON PSC.product_category_id = PC.product_category_id

WHERE P.color = 'Red' AND P.list_price >= 100
```

### Задание 2

Вывести на экран названия подкатегорий с совпадающими именами.

```sql
SELECT A.name
FROM production.product_subcategory as A

JOIN production.product_subcategory as B
ON A.product_subcategory_id != B.product_subcategory_id AND A.name = B.name
```

### Задание 3
Вывести на экран название категорий и количество товаров в данной категории.

```sql
SELECT PC.name, COUNT(*)
FROM production.product AS P

JOIN production.product_subcategory as PSC
ON P.product_subcategory_id = PSC.product_subcategory_id

JOIN production.product_category AS PC
ON PSC.product_category_id = PC.product_category_id

GROUP BY PC.name
```

### Задание 4
Вывести на экран название подкатегории, а также количество товаров в данной подкатегории с учетом ситуации, что могут существовать подкатегории с одинаковыми именами.

```sql
SELECT PSC.name, COUNT(P.product_id)
FROM production.product AS P

JOIN production.product_subcategory as PSC
ON P.product_subcategory_id = PSC.product_subcategory_id

GROUP BY PSC.name
```

### Задание 5
Вывести на экран название первых трех подкатегорий с наибольшим количеством товаров.

```sql
SELECT PSC.name
FROM production.product as P

JOIN production.product_subcategory as PSC
ON P.product_subcategory_id = PSC.product_subcategory_id

GROUP BY PSC.name, P.product_id
ORDER BY COUNT(*) DESC
LIMIT 3
```

### Задание 6
Вывести на экран название подкатегории и максимальную цену продукта с цветом Red в этой подкатегории.

```sql
SELECT PSC.name, MAX(P.list_price)
FROM production.product as P

JOIN production.product_subcategory as PSC
ON P.product_subcategory_id = PSC.product_subcategory_id

WHERE (P.color = 'Red')
GROUP BY PSC.name
```

### Задание 7
Вывести на экран название поставщика и количество товаров, которые он поставляет.

```sql
SELECT V.name, COUNT(*)
FROM purchasing.product_vendor as PV

JOIN purchasing.vendor AS V
ON PV.business_entity_id = V.business_entity_id

GROUP BY V.name
```

### Задание 8
Вывести на экран название товаров, которые поставляются более чем одним поставщиком.

```sql
SELECT P.name
FROM production.product as P

JOIN purchasing.product_vendor as PV
ON P.product_id = PV.product_id

GROUP BY P.product_id
HAVING COUNT(PV.business_entity_id) > 1
```

### Задание 9
Вывести на экран название самого продаваемого товара.

```sql
SELECT P.name
FROM production.product AS P

JOIN sales.sales_order_detail as SOD
ON P.product_id = SOD.product_id

GROUP BY P.product_id
ORDER BY SUM(SOD.order_qty) DESC
LIMIT 1
```

### Задание 10
Вывести на экран название категории, товары из которой продаются наиболее активно.

```sql
SELECT PC.name
FROM production.product_category as PC

JOIN production.product_subcategory as PSC
ON PC.product_category_id = PSC.product_category_id

JOIN production.product as P
ON PSC.product_subcategory_id = P.product_subcategory_id 

JOIN sales.sales_order_detail as SOD
ON P.product_id = SOD.product_id

GROUP BY PC.name
ORDER BY SUM(SOD.order_qty) DESC
LIMIT 1
```

### Задание 11
Вывести на экран названия категорий, количество подкатегорий и количество товаров в них.

```sql
SELECT PC.name, COUNT(DISTINCT PSC.product_subcategory_id), COUNT(DISTINCT P.product_id)
FROM production.product_category as PC

JOIN production.product_subcategory as PSC
ON PC.product_category_id = PSC.product_category_id

JOIN production.product AS P
ON PSC.product_subcategory_id = P.product_subcategory_id

GROUP BY PC.name
```

### Задание 12
Вывести на экран номер кредитного рейтинга и количество товаров, поставляемых компаниями, имеющими этот кредитный рейтинг.

```sql
SELECT V.credit_rating, COUNT(PV.product_id)
FROM purchasing.vendor as V

JOIN purchasing.product_vendor as PV
ON V.business_entity_id = PV.business_entity_id 

GROUP BY V.credit_rating
```

### Задание 13
Найти первые 10 процентов самых дорогих товаров, с учетом ситуации, когда цены у некоторых товаров могут совпадать.

```sql
SELECT DISTINCT P1.name, P1.list_price
FROM production.product AS P1
JOIN (
 SELECT list_price
 FROM production.product
 ORDER BY list_price DESC
 LIMIT 0.1 * (SELECT COUNT(*) FROM production.product)
) AS P2
 ON P1.list_price = P2.list_price
ORDER BY P1.list_price DESC;
```

### Задание 14
Найти первых трех поставщиков, отсортированных по количеству поставляемых товаров, с учетом ситуации, что количество поставляемых товаров может совпадать для разных поставщиков.

```sql
SELECT V.name
FROM purchasing.vendor as V

JOIN purchasing.product_vendor as PV
ON V.business_entity_id = PV.business_entity_id

JOIN production.product as P
ON PV.product_id = P.product_id

GROUP BY V.name
ORDER BY COUNT(P.product_id) DESC
LIMIT 3
```

### Задание 15
Найти для каждого поставщика количество подкатегорий продуктов, к которым относится продукты, поставляемые им, без учета ситуации, когда продукт не относится ни к какой подкатегории.

```sql
SELECT V.name, COUNT(DISTINCT PS.product_category_id)
FROM purchasing.vendor as V

JOIN purchasing.product_vendor as PV
ON V.business_entity_id = PV.business_entity_id

JOIN production.product as P
ON PV.product_id = P.product_id

JOIN production.product_subcategory as PS
ON P.product_subcategory_id = PS.product_subcategory_id

GROUP BY V.business_entity_id
```

### Задание 16
Проверить, есть ли продукты с одинаковым названием, если есть, то вывести эти названия. (Решение через JOIN)

```sql
SELECT A.name
FROM production.product AS A

JOIN production.product as B
ON A.name = B.name AND A.product_id <> B.product_id
```
