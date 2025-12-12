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
  
  SORT formCount DESC
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

Найти лексемы с леммой "дом", вывести их леммы и lexeme_id.
```
FOR lex IN Lexeme
  FILTER lex.lemma == "дом"
  RETURN {
    lemma: lex.lemma,
    lexeme_id: lex.lexeme_id
  }
```

Найти все лексемы из категории Q1084, у которых есть хотя бы 1 значение. Вывести их леммы и количество значений.
```
FOR lex IN 1..1 INBOUND "Category/Q1084" HAS_CATEGORY
  LET senseCount = LENGTH(
    FOR sense IN 1..1 OUTBOUND lex HAS_SENSE
      RETURN 1
  )
  FILTER senseCount > 0
  RETURN {
    lemma: lex.lemma,
    senseCount: senseCount
  }
```

Найти все лексемы, у которых более 5 форм и более 3 значений одновременно. Для каждой лексемы вывести лемму, количество форм, количество значений и все формы (массив репрезентаций form.representation).
```
FOR lex IN Lexeme
  LET forms = (
    FOR form IN 1..1 OUTBOUND lex HAS_FORM
      RETURN form.representation
  )
  LET sensesCount = LENGTH(
    FOR sense IN 1..1 OUTBOUND lex HAS_SENSE
      RETURN 1
  )
  FILTER LENGTH(forms) > 5 AND sensesCount > 3
  RETURN {
    lemma: lex.lemma,
    formsCount: LENGTH(forms),
    sensesCount: sensesCount,
    forms: forms
  }
```

Найти все лексемы из языка Q7737, отсортировать по возрастанию леммы и вывести леммы
```
for lex in 1..1 inbound "Language/Q7737" IN_LANGUAGE
  sort lex.lemma asc
  return lex.lemma
```

Найти все лексемы из языка Q7737, вывести их леммы и категории, убрать дубликаты
```
for lex in 1..1 inbound "Language/Q7737" IN_LANGUAGE
  for cat in 1..1 outbound lex HAS_CATEGORY
    return distinct {lemma: lex.lemma, category: cat.name}
```

Найдите лексемы, которые имеют наибольшее количество связей в своей категории. Выведите их леммы, количество связей, имя категории и количество лексем в этой категории
```
for cat in Category
  let ml = max(for lex in 1..1 inbound cat HAS_CATEGORY
      return length(for neigh in 1..1 any lex HAS_FORM, HAS_SENSE, IN_LANGUAGE, HAS_CATEGORY, HAS_CLAIM return 1))
  let lex_count = length(for lex in 1..1 inbound cat HAS_CATEGORY
      return 1)
  let good_names = (for lex in 1..1 outbound cat HAS_CATEGORY
        filter length(for neigh in 1..1 any lex HAS_FORM, HAS_SENSE, IN_LANGUAGE, HAS_CATEGORY, HAS_CLAIM 
        return 1) == ml
      return lex.name)
  return {lemma: good_names[0], 
          cat_name: cat.name,
          count_edges: ml,
          lexemes_count: lex_count
    }
```

Найти все лексемы с леммой "дом".
```
FOR l IN Lexeme
  FILTER l.lemma == 'дом'
  RETURN {
    id: l.lexeme_id,
    lemma: l.lemma
  }
```

Найти все лексемы из категории "Q1084", у которых есть хотя бы одно значение.
```
FOR c IN Category
  FILTER c.category_id == 'Q1084'
  FOR l IN Lexeme
    LET s_cnt = LENGTH(FOR s IN OUTBOUND l HAS_SENSE RETURN 1)
    FILTER s_cnt > 0
    RETURN l.lemma
```

Найти все лексемы, у которых одновременно больше 5 форм и больше 3 значений, вывести лемму, количество форм, количество значений, категорию, все формы в текстовом представлении.
```
FOR l IN Lexeme
  LET s_cnt = LENGTH(FOR s IN OUTBOUND l HAS_SENSE RETURN 1)
  let forms = (FOR f IN OUTBOUND l HAS_FORM RETURN f.representation)
  LET f_cnt = LENGTH(forms)
  
  LET cat = (FOR c IN OUTBOUND l HAS_CATEGORY RETURN c)
  
  FILTER s_cnt > 3 AND f_cnt > 5
  RETURN {
    lemma: l.lemma,
    form_count: f_cnt,
    sense_count: s_cnt,
    category: cat[0].name,
    forms: forms
  }
```

Найти все лексемы с type 'lexeme'. Вывести все lexeme_id. Вывести первые 20 штук.

```
FOR lex IN Lexeme
  FILTER lex.type == "lexeme"
  LIMIT 20
  RETURN lex.lexeme_id
```
  
Найти все лексемы, которые имеют значение с толкованием содержащим подстроку "жилище". Вывести все их леммы и количество таких значений.

```
FOR sense IN Sense
  FILTER CONTAINS(sense.gloss, "жилище")
  FOR lex IN 1..1 INBOUND sense HAS_SENSE
    COLLECT lemma = lex.lemma INTO sensesGroup
    RETURN {
      lemma: lemma,
      senses_count: LENGTH(sensesGroup)
    }
```    
    
Найти все лексемы которые связаны с более чем одной категорией (если такое возможно в структуре данных) или имеют свойства с разными 'property id'. Вывести лемму, количество различных категорий и свойств и список этих категорий/свойств.

```
FOR lex IN Lexeme
  LET categories = UNIQUE(
    FOR cat IN 1..1 OUTBOUND lex HAS_CATEGORY
      RETURN { category_id: cat.category_id, name: cat.name }
  )
  LET properties = UNIQUE(
    FOR prop IN 1..1 OUTBOUND lex HAS_CLAIM
      RETURN { property_id: prop.property_id, name: prop.name }
  )
  LET categories_count = LENGTH(categories)
  LET properties_count = LENGTH(properties)
  FILTER categories_count > 1 OR properties_count > 1
  RETURN {
    lemma: lex.lemma,
    categories_count: categories_count,
    properties_count: properties_count,
    categories: categories,
    properties: properties
  }
  ```

Найти все лексеммы русского языка (Q7737 язык), которе существительные (Q1084 категория) у которых есть хотя бы две формы и смысл 'предмет', вывести их леммы, найденные количества и все смыслы

```
FOR lex IN Lexeme
    LET languageCheck = (
        FOR lang IN 1..1 OUTBOUND lex IN_LANGUAGE
            FILTER lang.language_id == 'Q7737'
            RETURN 1
    )
    FILTER LENGTH(languageCheck) > 0

    LET categoryCheck = (
        FOR cat IN 1..1 OUTBOUND lex HAS_CATEGORY
            FILTER cat.category_id == 'Q1084'
            RETURN 1
    )
    FILTER LENGTH(categoryCheck) > 0

    LET forms = (
        FOR form IN 1..1 OUTBOUND lex HAS_FORM
            RETURN form
    )
    LET formCount = LENGTH(forms)
    FILTER formCount >= 2
    
    LET senses = (
        FOR sense IN 1..1 OUTBOUND lex HAS_SENSE
            RETURN sense
    )
    
    LET hasSubjectSense = (
        FOR sense IN senses
            FILTER CONTAINS(sense.gloss, 'предмет')
            LIMIT 1
            RETURN 1
    )
    FILTER LENGTH(hasSubjectSense) > 0
    
    RETURN {
        lemma: lex.lemma,
        lexeme_id: lex.lexeme_id,
        forms_count: formCount,
        senses_count: LENGTH(senses),
        all_forms: forms[*].representation,
        all_senses: senses[*].gloss,
    }
```

Найти все лексемы с максимальным количеством связей (степень вершины). Вывести топ-10 лексем с их леммами и количеством связей.

```
LET outboundCount = LENGTH(
        FOR v IN 1..1 OUTBOUND lex HAS_FORM, HAS_SENSE, HAS_CATEGORY, IN_LANGUAGE, HAS_CLAIM
            RETURN 1
    )
    
    LET inboundCount = LENGTH(
        FOR v IN 1..1 INBOUND lex HAS_FORM, HAS_SENSE, HAS_CATEGORY, IN_LANGUAGE, HAS_CLAIM
            RETURN 1
    )
    
    LET degree = outboundCount + inboundCount
    
    SORT degree DESC
    LIMIT 10
    
    RETURN {
        lexeme_id: lex.lexeme_id,
        lemma: lex.lemma,
        degree: degree,
        outbound: outboundCount,
        inbound: inboundCount
    }
```
