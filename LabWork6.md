# Лабораторная работа №6

### Задание 1
Подсчитать общее количество всех лексем в базе данных.

```
FOR lex IN Lexeme
  COLLECT WITH COUNT INTO count
  RETURN count
```
  
### Задание 2
Найти все существительные в базе данных. Категория существительных имеет category_id = 'Q1084'.

```
FOR v, e, p IN 1..1 INBOUND 'Category/Q1084' HAS_CATEGORY
  RETURN v.lemma
```

```
FOR cat IN Category
  FILTER cat.category_id == 'Q1084'
  FOR lex IN 1..1 INBOUND cat HAS_CATEGORY
    RETURN lex.lemma
```    

### Задание 3
Получить все формы лексемы lexeme_id=L82809. Вывести текстовые представления форм (representation).

```
FOR v, e, p IN 1..1 OUTBOUND 'Lexeme/L82809' HAS_FORM
  RETURN v.representation
```
```
FOR lex IN Lexeme
  FILTER lex.lexeme_id == 'L82809'
  FOR form IN 1..1 OUTBOUND lex HAS_FORM
    RETURN form.representation
```  

### Задание 4
Найти все лексемы с их количеством форм и значений. Вывести лемму, количество форм и количество значений. Отсортировать по количеству форм по убыванию. Вывести первые 10.

```
FOR lex IN Lexeme
  LET formCount = LENGTH(
    FOR form IN 1..1 OUTBOUND lex HAS_FORM
      RETURN 1
  )
  
  LET senseCount = LENGTH(
    FOR sense in 1..1 OUTBOUND lex HAS_SENSE
      RETURN 1
  )
  
  SORT senseCount DESC
  LIMIT 10
  
  RETURN {
    lexeme: lex.lemma,
    formCount: formCount,
    senseCount: senseCount
  }
```  

### Задание 5
Найти все лексемы с максимальным количеством связей (степень вершины). Вывести топ-10 лексем с их леммами и количеством связей.

```
FOR lex IN Lexeme
  LET outBoundCount = LENGTH(
    FOR v IN 1..1 OUTBOUND lex HAS_FORM, HAS_SENSE, IN_LANGUAGE, HAS_CATEGORY, HAS_CLAIM
      RETURN 1
  )
  
  LET inBoundCount = LENGTH(
    FOR v IN 1..1 INBOUND lex HAS_FORM, HAS_SENSE, IN_LANGUAGE, HAS_CATEGORY, HAS_CLAIM
      RETURN 1
  )
  
  LET degree = outBoundCount + inBoundCount
  
  SORT degree DESC
  LIMIT 10
  
  RETURN {
    lexeme: lex.lemma,
    degree: degree
  }
```

### Задание 6
Найти все лексемы, которые имеют определённое значение (sense) с толкованием, содержащим подстроку “специальность”, и вывести их категории. Результат должен содержать пары (лемма, название категории).

```
FOR sense IN Sense
  FILTER contains(sense.gloss, 'специальность')
  FOR lex IN 1..1 INBOUND sense HAS_SENSE
    FOR cat IN 1..1 OUTBOUND lex HAS_CATEGORY
      RETURN {
        lemma: lex.lemma,
        category: cat.name
      }
```

### Задание 7
Найти все лексемы без форм или без значений. Вывести их леммы и lexeme_id.

```
FOR lex IN Lexeme
  LET formCount = LENGTH(
    FOR v IN 1..1 OUTBOUND lex HAS_FORM
      RETURN 1
  )
  
  LET senseCount = LENGTH(
    FOR v IN 1..1 OUTBOUND lex HAS_SENSE
      RETURN 1
  )
  
  FILTER formCount == 0 OR senseCount == 0
  
  RETURN {
    lemma: lex.lemma,
    id: lex.lexeme_id
  }
```

### Задание 8
Найти все лексемы с максимальным количеством связей (степень вершины). Вывести топ-10 лексем с их леммами и количеством связей.

```
FOR lex IN Lexeme
  LET outBoundCount = LENGTH(
    FOR v IN 1..1 OUTBOUND lex HAS_FORM, HAS_SENSE, IN_LANGUAGE, HAS_CATEGORY, HAS_CLAIM
      RETURN 1
  )
  
  LET inBoundCount = LENGTH(
    FOR v IN 1..1 INBOUND lex HAS_FORM, HAS_SENSE, IN_LANGUAGE, HAS_CATEGORY, HAS_CLAIM
      RETURN 1
  )
  
  LET degree = outBoundCount + inBoundCount
  
  SORT degree DESC
  LIMIT 10
  
  RETURN {
    lexeme: lex.lemma,
    degree: degree
  }
```

## Таски, которые были на защите

Найти все лексемы, у которых 'pageid' не равен 1000. Вывести их леммы и 'padeid'

```
FOR lex IN Lexeme
  FILTER lex.pageid != 1000
  RETURN {
    lemma: lex.lemma,
    pageid: lex.pageid
  }
```

Найти все категории и для каждой категории. Вывести ее название и среднее количество форм на лексему в этой категории.
```
FOR cat IN Category
  LET lexemes = (
    FOR lex IN 1..1 INBOUND cat HAS_CATEGORY
      RETURN lex
  )
  LET formsCounts = (
    FOR lex IN lexemes
      RETURN LENGTH(
        FOR form IN 1..1 OUTBOUND lex HAS_FORM
          RETURN 1
      )
  )
  RETURN {
    category: cat.name,
    avg_forms: AVG(formsCounts)
  }
```

Найти все лексемы и для каждой вычислить "плотность связей" (отношение общего количества связей к сумме количества форм и значений). Вывести топ 10 лексем с наибольшей плотностью связей, их леммы, количество форм, количество значений, общее количество связей и значение плотности
```
FOR lex IN Lexeme
  LET formsCount = LENGTH(
    FOR form IN 1..1 OUTBOUND lex HAS_FORM
      RETURN 1
  )

  LET sensesCount = LENGTH(
    FOR sense IN 1..1 OUTBOUND lex HAS_SENSE
      RETURN 1
  )

  LET totalEdges = LENGTH(
    FOR v IN 1..1 OUTBOUND lex HAS_FORM, HAS_SENSE, HAS_CATEGORY, IN_LANGUAGE, HAS_CLAIM
      RETURN 1
  ) + LENGTH(
    FOR v IN 1..1 INBOUND lex HAS_FORM, HAS_SENSE, HAS_CATEGORY, IN_LANGUAGE, HAS_CLAIM
      RETURN 1
  )

  LET denom = formsCount + sensesCount
  LET density = totalEdges / (denom == 0 ? 1 : denom)

  SORT density DESC
  LIMIT 10

  RETURN {
    lemma: lex.lemma,
    formsCount: formsCount,
    sensesCount: sensesCount,
    totalEdges: totalEdges,
    density: density
  }
```
