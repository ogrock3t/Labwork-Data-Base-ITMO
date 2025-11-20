# **Лабораторная работа №5**

### Задание 1
Показать все названия товаров из подкатегории Mountain Bikes (коллекции prod_products и prod_product_subcategories).

```
db.prod_products.aggregate([
  {
    $lookup: {
      from: "prod_product_subcategories",
      localField: "product_subcategory_id",
      foreignField: "_id",           
      as: "subcategory"
    }
  },
  {
    $unwind: "$subcategory"
  },
  {
    $match: {
      "subcategory.name": "Mountain Bikes"
    }
  },
  {
    $project: {
      _id: 0,
      name: "$name"
    }
  }
])
```

### Задание 2
Вывести список заказов, сделанных в 2012 году (коллекция sales_sales_order_headers).

```
db.sales_sales_order_headers.find({
    $expr: {
        $eq: [ { $year: "$order_date" }, 2012 ]
    }
})
```

### Задание 3
Вывести список названий товаров и их количество (Quantity) в заказах (коллекции sales_sales_order_details и prod_products).

```
db.sales_sales_order_details.aggregate([
    {
        $lookup: {
            from: "prod_products",
            localField: "product_id",
            foreignField: "product_id",
            as: "product"
        }
    },
    {
        $unwind: "$product"
    },
    {
        $project: {
            _id: 0,
            name: "$product.name",
            quantity: "$order_qty"
        }
    }
])
```

### Задание 4
Для каждого сотрудника вывести должность (job_title), дату приема на работу (hire_date) и сумму всех выплат (employee_pay_history) (коллекция hr_employees).

```
db.hr_employees.aggregate([
    {
        $project: {
            _id: 0,
            job_title: "$job_title",
            hire_date: "$hire_date",
            employee_pay_history: { $sum: "$employee_pay_history.rate" }
        }
    }
])
```

### Задание 5
Найти общее количество проданных товаров (сумма order_qty) по каждому заказу (sales_order_header_ref). Используйте агрегацию (коллекция sales_sales_order_details).

```
db.sales_sales_order_details.aggregate([
  {
    $group: {
      _id: "$sales_order_header_ref",
      total: { $sum: "$order_qty" }
    }
  },
  {
    $project: {
      sales_order_header_ref: "$_id",
      total_quantity: "$total",
    }
  }
])
```

### Задание 6
Вычислить среднюю стоимость товара (list_price) по каждой подкатегории (product_subcategory_id) (коллекция prod_products).

```
db.prod_products.aggregate([
  {
    $group: {
      _id: "$product_subcategory_id",
      avg: { $avg: "$list_price" }
    }
  },
  {
    $project: {
      name_subcategory: "$_id",
      avg: 1
    }
  }
])
```

### Задание 7
Определить, сколько заказов оформил каждый сотрудник (sales_person_ref) в 2013 году (коллекция sales_sales_order_headers).

```
db.sales_sales_order_headers.aggregate([
  {
    $match: {
      $expr: {$eq: [{ $year: "$order_date" }, 2013]}
    }
  },
  {
    $group: {
      _id: "$sales_person_ref",
      order_count: { $sum: 1 }
    }
  },
  {
    $project: {
      person: "$_id",
      order_count: 1
    }
  }
])
```

### Задание 8
Найти подкатегории товаров, у которых более 10 товаров (коллекция prod_products).

```
db.prod_products.aggregate([
  {
    $group: {
      _id: "$product_subcategory_id",
      count: { $sum: 1 }
    }
  },
  {
    $match: {
      count: { $gt: 10 }
    }
  },
  {
    $project: {
      subcat: "$_id",
      count: 1
    }
  }
])
```

### Задание 9
Вывести список заказов, в которых купили более 5 единиц одного товара (коллекция sales_sales_order_details).

```
db.sales_sales_order_details.distinct(
    "sales_order_id",
    {
        order_qty: { $gt: 5 }
    }
)
```

### Задание 10
Показать клиентов, которые совершили более 3 заказов в 2013 году (коллекция sales_sales_order_headers).

```
db.sales_sales_order_headers.aggregate([
  {
    $match: {
      $expr: { $eq: [{ $year: "$order_date" }, 2013] }
    }
  },
  {
    $group: {
      _id: "$customer_ref",
      count: { $sum: 1 }
    }
  },
  {
    $match: {
      count: { $gt: 3 }
    }
  },
  {
    $project: {
      customer_ref: "$_id",
      count: 1
    }
  }
])
```

### Задание 11
Найти сотрудников, у которых день рождения в текущем месяце (коллекция hr_employees).

```
db.hr_employees.find({
  $expr: { $eq: [ { $month: "$birth_date" }, 11 ] }
})
```
