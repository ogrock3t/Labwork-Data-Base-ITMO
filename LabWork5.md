# **Лабораторная работа №5**

### Задание 1
Показать все названия товаров из подкатегории Mountain Bikes (коллекции prod_products и prod_product_subcategories).

```
db.prod_product_subcategories.aggregate([
    {
        $match: {
            name: "Mountain Bikes"
        }
    },
    {
        $lookup: {
            from: "prod_products",
            localField: "product_subcategory_id",
            foreignField: "product_subcategory_id",
            as: "product"
        }
    },
    {
        $unwind: "$product"
    },
    {
        $project: {
            _id: 0,
            product_name: "$product.name"
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

## Таски, которые были на защите

Вывести продукты, у которых есть подкатегория и отсортировать.
```
db.prod_products.find(
  {
    product_subcategory_id: { $ne: null }
  },
  {
    _id: 0,
    product_subcategory_id: 1,
    name: 1
  }
).sort({ name: 1 })
```

Вывести товары, таких что товаров этого размера больше 10.
```
db.prod_products.aggregate([
  {
    $match: {
      size: { $ne: null }
    }
  },
  {
    $group: {
      _id: "$size",
      counter: { $sum: 1 }
    }
  },
  {
    $match: {
      counter: { $gt: 10 }
    }
  },
  {
    $project: {
     _id: 0,
      size: "$_id",
      counter: 1
    }
  }
])
```

3 первых товара с наибольшим количеством продаж.
```
db.sales_sales_order_details.aggregate([
    {
        $group: {
            _id: "$product_id",
            count: {$sum: "$order_qty"}
        }
    },
    {
        $sort: {
            count: -1
        }
    },
    {
        $limit: 3
    }
])
```

Вывести товары которые ни разу не продавались.
```
db.prod_products.aggregate([
  {
    $lookup: {
        from: "sales_sales_order_details",
        localField: "_id",
        foreignField: "product_id",
        as: "sales"
    }
  },
  {
    $match: { sales: { $eq: [] } }
  },
  {
    $project: {
        _id: 0,
        name: 1
    }
  }
])
```

Товары у которых цена менялась больше 1 раза.
```
db.prod_products.find({
  $expr: { $gt: [ { $size: "$product_cost_history" }, 1 ] }
})
```

Вывести товары, которые начали продаваться после мая 2013.
```
db.prod_products.find({
  sell_start_date: { $gt: ISODate("2013-05-31T23:59:59Z") }
}).sort( { size: 1 } )
```

Вывести названия красных товаров без размера, упорядочив по modified_date.
```
db.prod_products.aggregate([
    {
        $match: {
            color: "Red",
            size: { $eq: null }
        }
    },
    {
        $sort: { modified_date: 1 }
    },
    {
        $project: {
            _id: 0,
            name: 1,
            modified_date: 1
        }
    }
])
```

Вывести количество товаров в каждой подкатегории.
```
db.prod_products.aggregate([
    {
        $group: {
            _id: "$product_subcategory_id",
            total_products: { $sum : 1 }
        }
    },
    {
        $project: {
            _id: 0,
            product_subcategory_id: "$_id",
            total_products: 1
        }
    }
])
```
