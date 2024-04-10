### Table of Contents
- **[README](../README.md)**
- **Ansible** (English/[Russian](ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** (English/[Russian](elastic-ru.md)): Search and data analysis.
- **Kubernetes** (English/[Russian](kubectl-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** (English/[Russian](regex-ru.md)): A powerful tool for working with text.
- **Ubuntu** (English/[Russian](ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](../en/vault-en.md)/Russian): Management of secrets and access.

# Терминология и концепции

**Кластер (Cluster)** - это группа одного или нескольких узлов (nodes), объединенных вместе 
для хранения и обработки данных. Кластер обеспечивает распределение данных и высокую 
доступность.

**Узел (Node)** - это один сервер или экземпляр Elasticsearch, входящий в состав кластера. 
Узлы выполняют индексацию, хранение и выполнение операций поиска.

**Индекс (Index)** - представляет собой логическую категорию или контейнер, который содержит 
набор документов с одинаковой схемой. Индексы используются для организации и ускорения 
поиска по данным.

**Шард (Shard)** - это физическая часть индекса, которая содержит фрагмент данных. Индекс 
может быть разделен на несколько шардов, чтобы обеспечить горизонтальное масштабирование 
и распределение нагрузки.

**Репликация (Replication)** - позволяет создать дубликаты шардов для обеспечения 
отказоустойчивости и увеличения доступности данных. Реплики автоматически восстанавливаются 
в случае сбоя.

**DSL (Domain-Specific Language)** - Elasticsearch использует собственный язык запросов (DSL) для формулирования поисковых запросов. Этот язык позволяет создавать сложные запросы 
и агрегации для анализа данных.

**Запрос (Query)** - Запрашивает документы совпадающие по заданным критериям.

**Агрегации (Aggregations)** - суммирует данные как метрики, статистики и прочее, на основе запросов. Это полезно для аналитики и создания отчетов.

**Кластеризация (Clustering)** - это процесс автоматического распределения данных и запросов между узлами кластера для балансировки нагрузки и обеспечения масштабируемости.

**Точность (Precision)** - Precision измеряет долю правильно классифицированных положительных экземпляров (True Positives) среди всех экземпляров, которые модель отнесла к положительному классу (True Positives + False Positives). Точность оценивает, насколько модель верно идентифицирует положительные случаи и избегает ложных срабатываний. Формула: Precision = TP / (TP + FP)

**Полнота (Recall)** - Recall измеряет долю правильно классифицированных положительных экземпляров (True Positives) среди всех экземпляров, которые действительно принадлежат положительному классу (True Positives + False Negatives). Полнота оценивает способность модели обнаруживать все положительные случаи в данных. Формула: Recall = TP / (TP + FN)

**Маппинг (Mapping)** - определяет схему и типы данных для индекса и его полей. Это позволяет Elasticsearch понимать и обрабатывать данные правильно:
- Если вы не определили mapping заранее, Elasticsearch динамически создаст его для вас.
- Если вы решите определить собственный mapping, вы можете сделать это при создании индекса.
- Для каждого индекса определяется ОДИН mapping. После создания индекса мы можем только добавлять в mapping новые поля. Мы НЕ МОЖЕМ изменить отображение существующего поля.
- Если вам необходимо изменить тип существующего поля, вы должны создать новый индекс с нужным mapping, а затем переиндексировать все документы в новый индекс.
# Strings:
- **Text** - Полнотекстный поиск, где слова в тексте разбиваются на отдельные токены 
            (например, слова) и индексируются для более гибкого поиска и анализа текстовых 
            данных. Занимает больше места и тратит больше ресурсов для индексации.
- **Keyword** - Хранение данных, которые не должны быть разбиты на отдельные токены. Это полезно, например, для хранения и поиска идентификаторов, ключей или значений, которые нуждаются в точном сопоставлении. Занимает меньше места и ресурсов.
- **Возможные поля**:
    - **name/description** - Всегда используют text тип данных, поскольку агрегировать или сортировать их не требуется.
    - **country** - По усмотрению. Всегда text, но если планируется агрегировать то и keyword.
    - **product_type** - Всегда keyword, поле используется только для агрегации и сортировки.

## Пример изменения типов данных в индексе:
### Создаем новый индекс с ПРАВИЛЬНЫМИ типами данных:
```
PUT produce_index
{
    "mappings": {
        "properties": {
            "country": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 256
                    }
                }
            },
            "date_purchased": {
                "type": "date"
            },
            "description": {
                "type": "text"
            }
        }
    }
}
```

### Переносим данные старого индекса в новый:
```
POST _reindex
{
    "source": {
        "index": "old_index"
    },
    "dest": {
        "index": "produce_index"
    }
}
```
# Ситуативные примеры
## Пример создания поля с формулой:
```
PUT produce_v2/_mapping     #требуется создать функцию которая находит суммарные месячные затраты
{
    "runtime": {            #runtime поле создается ТОЛЬКО когда юзер к нему обращается
        "total": {
        "type": "double",
            "script": {     #умножаем стоимость 1 юнита на их колличество
                "source": "emit(doc['unit_price'].value* doc['quantity'].value)"
            }
        }
    }
}
```
## Вызов runtime поля:
```
GET produce_v2/_search
{
"size": 0,
    "aggs": {
        "total_expense": {
            "sum": {
                "field": "total"
            }
        }
    }
}
```             


## Получить значение индекса:
    GET index_name/_doc/document_id

Создать/перезаписать существующий индекс: (POST если нужно автосгенерировать document_id)
    PUT index_name/_doc/document_id
    {
        "field1": "value",
        "field2": "value"
    }

## Создать без случайной перезаписи:
```
PUT index_name/_create/document_id
{
    "field1": "value",
    "field2": "value"
}
```
## Апдейт индекса новыми строчками или изменение старых:
```
POST index_name/_update/document_id
{
    "doc": {
        "field1": "value",
        "field2": "value"
    }
} 
```
## Удаление индекса:
```
DELETE index_name/_doc/document_id
```
## Вывод всего индекса:
```
GET index_name/_search
```
## Вывод точного колличества документов индекса:
```
GET index_name/_search
{
    "track_total_hits": true
}
```
## Вывод по запросу:
```
GET index_name/_search
{
    "query": {
        "range": {
            "date": {
                "gte": "2015-06-20",
                "lte": "2015-09-22"
            }
        }
    }
}
```
### ИЛИ
```
GET index_name/_search
{
    "query": {
        "match": { #поиск точной фразы, более Recall метод
            "headline": {
                "query": "query"
            }
        }
    }
}
```
### ИЛИ
```
GET index_name/_search
{
    "query": {
        "match_phrase": { #поиск точной фразы, более Precision метод
            "headline": {
                "query": "query"
            }
        }
    }
}
```

### ИЛИ
```
GET index_name/_search
{
"query": {
        "multi_match": { #поиск в нескольких разных полях
        "query": "Enter search terms here",
            "fields": [
                "List the field you want to search over", #для приоритезации можно добавить "^2"
                "List the field you want to search over",
                "List the field you want to search over"
            ],
            "type": "phrase" #точная фраза, увеличивает Precision
        }
    }
}
```
## Агрегированный вывод:
```
GET index_name/_search
{
    "aggs": {
        "name your aggregation here (by_category)": {
            "specify aggregation type here (terms)": {
                "field": "name the field you want to aggregate here (category)",
                "size": "state how many buckets you want returned here (100)"
            }
        }
    }
}
```
## Вывод по запросу и агрегации:
```
GET index_name/_search
{
"query": {
    "match": {
    "Enter the name of the field": "Enter the value you are looking for"
    }
},
"aggs": {
        "Name your aggregation here": {
            "significant_text": {
                "field": "Enter the name of the field you are searching for"
            }
        }
    }
}
```
## Recall метод (больше выдача, меньше точность): 

    GET index_name/_search
    {
        "query": {
            "match": {
                "headline": {
                    "query": "Khloe Kardashian Kendall Jenner" #совпадение чего угодно из строки
                }
            }
        }
    }

## Precision метод (меньше выдача, больше точность):
```
GET index_name/_search
{
    "query": {
        "match": {
            "headline": {
                "query": "Khloe Kardashian Kendall Jenner",
                "operator": "and" #совпадение всех слов
                #ИЛИ
                "minimum_should_match": 3 #совпадение нескольких слов
            }
        }
    }
}
```
## Bool запрос:
```
POST index/_search
{
    "query": {
        "bool" : {
            "must" : {                              #всегда должно включать
                "term" : { "user.id" : "kimchy" }
            },
            "filter": {                             #фильтр по условию, гибкий параметр
                "term" : { "tags" : "production" }
            },
            "must_not" : {                          #всегда должно исключать
                "range" : {
                    "age" : { "gte" : 10, "lte" : 20 }
                }
            },
            "should" : [                            #выдача с приоритетом (выше score для значения)
                { "term" : { "tags" : "env1" } },
                { "term" : { "tags" : "deployed" } }
            ],
            "minimum_should_match" : 1,
            "boost" : 1.0
        }
    }
}
```

# Аггригирование:
## Metric aggregation:
Используются для вычисления числовых значений на основе вашего набора данных. Его можно использовать для расчета значений суммы, минимума, максимума, среднего, уникального количества (мощности) и т. д.
```
GET Enter_name_of_the_index_here/_search
{
    "aggs": {
            "size": 0,                          #не выводить все документы
            "Name your aggregations here": {
            "sum/min/max/avg/stats...": {
                "field": "Name the field you want to aggregate on here"
            }
        }
    }
}
```
## Cardinality Aggregation:
Вычисляет количество уникальных значений для поля.
```
GET Enter_name_of_the_index_here
{
"size": 0,
    "aggs": {
        "Name your aggregations here": {
            "cardinality": {
                "field": "Name the field you want to aggregate on here"
            }
        }
    }
}
```
## Bucket aggregation:
Если требуется агрегировать несколько подмножеств документов, вам пригодятся агрегированные бакеты. Агрегация бакетов группируют документы в несколько наборов документов, называемых бакетами. Все документы в корзине имеют общие критерии.
### Агрегация бакетов имеент следующие виды:
- Date Histogram Aggregation
- Histogram Aggregation
- Range Aggregation
- Terms aggregation
        
    - ### Date Histogram Aggregation:
    Создаются бакеты с интервалом в 8 часов которые содержат порционную информацию о колличестве новых документов за этот промежуток времени (представляется в виде гистограммы)
    ```
    GET ecommerce_data/_search
    {
    "size": 0,
        "aggs": {
            "transactions_by_8_hrs": {
                "date_histogram": {
                    "field": "InvoiceDate",
                    ###
                    "fixed_interval": "8h"      #по 8 часов
                                    #OR
                    "calendar_interval": "1M",  #помесячно
                    "order": {
                    "_key": "asc/desc"          #сортировка
                    }
                    ###
                }
            }
        }
    }
    ```
    - ### Histogram Aggregation:
    Обычная гистограмма не для дат
    ```
    GET ecommerce_data/_search
    {
    "size": 0,
        "aggs": {
            "transactions_per_price_interval": {
                "histogram": {
                    "field": "UnitPrice",
                    "interval": 10
                }
            }
        }
    }
    ```
    - ### Range Aggregation:
    Гистограмма с интервальными шагами вида 0-50, 50-100, 100+:
    ```
    GET ecommerce_data/_search
    {
    "size": 0,
        "aggs": {
            "transactions_per_custom_price_ranges": {
                "range": {
                    "field": "UnitPrice",
                    "ranges": [
                    {
                        "to": 50
                    },
                    {
                        "from": 50,
                        "to": 100
                    },
                    {
                        "from": 100
                    }
                    ]
                }
            }
        }
    }
    ```
    - ### Terms aggregation:
    Создает отдельный бакет для каждого уникального выражения. Чаще всего используется для самых часто используемых выражений среди данных
    ```
    GET ecommerce_data/_search
    {
    "size": 0,
        "aggs": {
            "top_5_customers": {            #находим самого частого покупателя
                "terms": {
                    "field": "CustomerID",  #ищем по частоте айди покупателя
                    "size": 5               #топ 5
                }
            }
        }
    } 
    ```
## Combinet aggregation:
Комбинация двух предыдущих аггрегаций. Используется, например, при нахождении суммы дохода за день
```
GET ecommerce_data/_search
{
"size": 0,
    "aggs": {
        "transactions_per_day": {       #сплитим документы на дневные интервалы
        "date_histogram": {
            "field": "InvoiceDate",
            "calendar_interval": "day"
        },
            "aggs": {                       #субагрегация внутри каждого интервала
                "daily_revenue": {          #сумма дохода
                    "sum": {
                        "script": {         #скрипт умножающий кол-во продаж на цену одного проданного юнита
                            "source": "doc['UnitPrice'].value * doc['Quantity'].value" 
                        }
                    }   
                },
                "unique_customers": {       #находим уникальных покупателей за день в той же субагрегации
                    "cardinality": {
                        "field": "CustomerID"   
                    }
                }
            }
        }
    }
}
```
