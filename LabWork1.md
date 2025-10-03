# **Лабораторная работа №1**

### Задание 1

Найти и вывести на экран названия продуктов, их цвет и размер.

```sql
SELECT name, color, size FROM production.product
```

### Задание 2

Найти и вывести на экран названия, цвет и размер таких продуктов, у которых цена более 100.

```sql
SELECT name, color, size FROM production.product
WHERE list_price > 100
```

### Задание 3

Найти и вывести на экран название, цвет и размер таких продуктов, у которых цена менее 100 и цвет Black.

```sql
SELECT name, color, size FROM production.product
WHERE list_price < 100 AND color = 'Black'
```

### Задание 4

Найти и вывести на экран название, цвет и размер таких продуктов, у которых цена менее 100 и цвет Black, упорядочив вывод по возрастанию стоимости продуктов.

```sql
SELECT name, color, size FROM production.product
WHERE list_price < 100 AND color = 'Black'
ORDER BY list_price ASC
```

### Задание 5

Найти и вывести на экран название и размер первых трех самых дорогих товаров с цветом Black.

```sql
SELECT name, size FROM production.product
WHERE color = 'Black'
ORDER BY list_price DESC
LIMIT 3
```

### Задание 6

Найти и вывести на экран название и цвет таких продуктов, для которых определен и цвет, и размер.

```sql
SELECT name, color FROM production.product
WHERE name IS NOT NULL and color IS NOT NULL
```

### Задание 7

Найти и вывести на экран не повторяющиеся цвета продуктов, у которых цена находится в диапазоне от 10 до 50 включительно.

```sql
SELECT DISTINCT color from production.product
WHERE list_price >= 10 AND list_price <= 50
```

### Задание 8

Найти и вывести на экран се цвета таких продуктов, у которых в имени первая буква ‘L’ и третья ‘N’.

```sql
SELECT color FROM production.product
WHERE name ILIKE 'L_N%'
```

### Задание 9

Найти и вывести на экран названия таких продуктов, которых начинаются либо на букву ‘D’, либо на букву ‘M’, и при этом длина имени – более трех символов.

```sql
SELECT name FROM production.product 
WHERE name SIMILAR TO '(D|M)%' AND LENGTH(name) > 3
```

```sql
SELECT name FROM production.product
WHERE (name ILIKE 'D%' OR name ILIKE 'M%') AND LENGTH(name) > 3
```

### Задание 10

Вывести на экран названия продуктов, у которых дата начала продаж – не позднее 2012 года.

```sql
SELECT name FROM production.product
WHERE sell_start_date < '2013-01-01'
```

### Задание 11

Найти и вывести на экран названия всех подкатегорий товаров.

```sql
SELECT name FROM production.product_subcategory
```

### Задание 12

Найти и вывести на экран названия всех категорий товаров.

```sql
SELECT name FROM production.product_category
```

### Задание 13

Найти и вывести на экран имена всех клиентов из таблицы Person, у которых обращение (Title) указано как «Mr.».

```sql
SELECT first_name FROM person.person
WHERE title ILIKE 'Mr.'
```

### Задание 14

Найти и вывести на экран имена всех клиентов из таблицы Person, для которых не определено обращение (Title).

```sql
SELECT first_name FROM person.person
WHERE title IS NULL
```

### Задание 15

Получить все названия товаров в системе, в названии которых третий символ – либо буква “s”, либо буква “r”. Решить задачу как минимум двумя способами.

```sql
SELECT name FROM production.product 
WHERE name LIKE '__s%' OR name LIKE '__r%'
```
```sql
SELECT name FROM production.product 
WHERE name SIMILAR TO '__(s|r)%'
```

### Задание 16

Получить все названия товаров в системе, в названии которых ровно 5 символов.

```sql
SELECT name FROM production.product
WHERE LENGTH(name) = 5
```

### Задание 17

Написать запрос, который возвращает названия товаров, которые были в продаже между мартом 2011 года и мартом 2012 года включительно (необходимо учитывать формат даты).

```sql
SELECT name FROM production.product
WHERE sell_start_date <= '2012-03-31' AND sell_end_date >= '2011-03-01'
```

### Задание 18

Найти максимальную стоимость товара (отпускная цена ListPrice) из тех, которые были произведены, начиная с марта 2011 года.

```sql
SELECT max(list_price) FROM production.product
WHERE sell_start_date >= '2011-03-01'
```

### Задание 19

Найти и вывести на экран количество товаров каждого цвета, исключив из поиска товары, цена которых меньше 30.

```sql
SELECT color, COUNT(*) FROM production.product
WHERE list_price >= 30
GROUP BY color
```

### Задание 20

Найти и вывести на экран список, состоящий из цветов товаров, таких, что минимальная цена товара данного цвета более 100.

```sql
SELECT color FROM production.product
GROUP BY color
HAVING MIN(list_price) > 100;
```

### Задание 21

Найти и вывести на экран номера подкатегорий товаров и количество товаров в каждой подкатегории.

```sql
SELECT product_subcategory_id, COUNT(*) FROM production.product
GROUP BY product_subcategory_id
```

### Задание 22

Найти и вывести на экран номера товаров и количество фактов продаж данного товара (используется таблица SalesORDERDetail).

```sql
SELECT product_id, COUNT(*) FROM sales.sales_order_detail
GROUP BY product_id
```

### Задание 23

Найти и вывести на экран номера товаров, которые были куплены более пяти раз.

```sql
SELECT product_id FROM sales.sales_order_detail
GROUP BY product_id
HAVING COUNT(*) > 5
```

### Задание 24

Найти и вывести на экран номера покупателей, CustomerID, у которых существует более одного чека, SalesORDERID, с одинаковой датой.

```sql
SELECT DISTINCT customer_id FROM sales.sales_order_header
GROUP BY customer_id, order_date
HAVING COUNT(*) > 1
```

### Задание 25

Найти и вывести на экран все номера чеков, на которые приходится более трех продуктов.

```sql
SELECT sales_order_id FROM sales.sales_order_detail
GROUP BY sales_order_id
HAVING COUNT(*) > 3
```

### Задание 26

Найти и вывести на экран все номера продуктов, которые были куплены более трех раз.

```sql
SELECT product_id FROM sales.sales_order_detail
GROUP BY product_id 
HAVING COUNT(*) > 3
```

### Задание 27

Найти и вывести на экран все номера продуктов, которые были куплены или три или пять раз.

```sql
SELECT product_id FROM sales.sales_order_detail
GROUP BY product_id 
HAVING COUNT(*) = 3 OR COUNT(*) = 5
```

### Задание 28

Найти и вывести на экран все номера подкатегорий, в которым относится более десяти товаров.

```sql
SELECT product_subcategory_id FROM production.product
WHERE product_subcategory_id IS NOT NULL
GROUP BY product_subcategory_id
HAVING COUNT(*) > 10
```

### Задание 29

Найти и вывести на экран номера товаров, которые всегда покупались в одном экземпляре за одну покупку.

```sql
SELECT product_id FROM sales.sales_order_detail
GROUP BY product_id
HAVING MIN(order_qty) = 1 AND MAX(order_qty) = 1
```

### Задание 30

Найти и вывести на экран номер чека, SalesORDERID, на который приходится с наибольшим разнообразием товаров купленных на этот чек.

```sql
SELECT sales_order_id FROM sales.sales_order_detail
GROUP BY sales_order_id
ORDER BY COUNT(DISTINCT product_id) DESC
LIMIT 1
```

### Задание 31

Найти и вывести на экран номер чека, SalesORDERID с наибольшей суммой покупки, исходя из того, что цена товара – это UnitPrice, а количество конкретного товара в чеке – это ORDERQty.

```sql
SELECT sales_order_id FROM sales.sales_order_detail
GROUP BY sales_order_id
ORDER BY SUM(unit_price * order_qty) DESC
LIMIT 1
```

### Задание 32

Определить количество товаров в каждой подкатегории, исключая товары, для которых подкатегория не определена, и товары, у которых не определен цвет.

```sql
SELECT product_subcategory_id, COUNT(*) as count FROM production.product
WHERE product_subcategory_id IS NOT NULL AND color IS NOT NULL
GROUP BY product_subcategory_id
```

### Задание 33

Получить список цветов товаров в порядке убывания количества товаров данного цвета.

```sql
SELECT color FROM production.product
WHERE color IS NOT NULL
GROUP BY color
ORDER BY COUNT(*) DESC
```

### Задание 34

Вывести на экран ProductID тех товаров, что всегда покупались в количестве более 1 единицы на один чек, при этом таких покупок было более двух.

```sql
SELECT product_id FROM sales.sales_order_detail
GROUP BY product_id
HAVING MIN(order_qty) > 1 AND COUNT(*) > 2
```

## Таски, которые были на защите

```sql
SELECT product_id FROM sales.sales_order_detail
GROUP BY product_id
HAVING COUNT(*) > 3
```
```sql
SELECT color FROM production.product
GROUP BY color
HAVING COUNT(*) > 3 AND COUNT(*) < 5
```

```sql
SELECT product_id FROM sales.sales_order_detail
WHERE unit_price > 100
GROUP BY product_id
HAVING COUNT(*) > 3
```

```sql
SELECT product_category_id FROM production.product_subcategory
GROUP BY product_category_id
ORDER BY COUNT(product_subcategory_id) DESC
LIMIT 1
```
