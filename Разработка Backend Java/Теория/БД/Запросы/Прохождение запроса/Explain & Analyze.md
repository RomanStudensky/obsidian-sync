## Что делает `EXPLAIN` и чем отличается `EXPLAIN ANALYZE`

|Команда|Что показывает|Когда применять|
|---|---|---|
|`EXPLAIN`|**План, рассчитанный оптимизатором**: порядок узлов, типы сканов, предполагаемые **стоимости**, ‑‑‑без выполнения запроса‑‑‑|Быстро оценить, **какой** план выбрался и сколько (по мнению Postgres) это стоит|
|`EXPLAIN ANALYZE`|Тот же план **+ фактическое исполнение**: реальное время, количество строк, число повторов узла, использование буферов|Проверить, **прав ли** оптимизатор и где «узкое место»|

> ⚠️ `EXPLAIN ANALYZE` действительно выполняет запрос. Будьте осторожны с `UPDATE/DELETE` — добавляйте `ROLLBACK` или тестовую транзакцию.

---

### Мини‑схема для примеров

``` sql
CREATE TABLE employees (
  emp_id    serial PRIMARY KEY,
  name      text,
  salary    int,
  dept_id   int
);

CREATE INDEX ON employees (dept_id);
ANALYZE employees;     -- собираем статистику
```

Допустим, в таблице 1 000 000 строк, а `dept_id=5` встречается 20 000 раз.

---

## 1. `EXPLAIN` — прогноз оптимизатора

``` sql
EXPLAIN
SELECT * FROM employees WHERE dept_id = 5;

```

``` sql
Index Scan using employees_dept_id_idx on employees  (cost=0.43..4541.58 rows=20000 width=24)
  Index Cond: (dept_id = 5)

```

- **Index Scan** — будет использован индекс
    
- `cost=0.43..4541.58` — оценочная _стоимость_ (чем меньше, тем лучше)
    
    - 0.43 — «startup cost» (до первой строки)
        
    - 4541.58 — «total cost» (до последней)
        
- `rows=20000`, `width=24` — сколько строк и байт оптимизатор _ожидает_ вернуть
    

---

## 2. `EXPLAIN ANALYZE` — фактические цифры

``` sql
EXPLAIN ANALYZE
SELECT * FROM employees WHERE dept_id = 5;
```

``` sql
Index Scan using employees_dept_id_idx on employees  (cost=0.43..4541.58 rows=20000 width=24)
  Index Cond: (dept_id = 5)
  Actual Rows: 20018  Loops: 1
  Buffers: shared hit=1234 read=18
  Planning Time: 0.179 ms
  Execution Time: 5.732 ms
```
Что добавилось:

|Поле|Что означает|Как читать|
|---|---|---|
|**Actual Rows**|Реально прочитано строк|Сильно отличается от `rows=`? → статистика неточна|
|**Loops**|Сколько раз выполнялся узел (внутренние циклы вложенных join’ов)|>1 — узел вызвали повторно|
|**Buffers**|Сколько страниц памяти/диска задействовано|`hit` > 0 & `read` = 0 → всё было в shared buffers|
|**Planning Time**|Сколько заняло построение плана||
|**Execution Time**|Полное время запроса (до последней строки)||

---

## 3. Пример на `JOIN` с разными планами

``` sql
EXPLAIN ANALYZE
SELECT *
FROM   employees  e
JOIN   departments d  ON d.dept_id = e.dept_id
WHERE  d.name = 'HR';

```

``` sql
Nested Loop  (cost=0.42..58.70 rows=10 width=48)
             (actual time=0.052..0.944 rows=12 loops=1)
  ->  Index Scan using departments_name_idx on departments d  (cost=0.29..8.30 rows=1 width=28)
                               (actual time=0.020..0.022 rows=1 loops=1)
        Index Cond: (name = 'HR')
  ->  Index Scan using employees_dept_id_idx on employees e   (cost=0.13..50.35 rows=10 width=24)
                               (actual time=0.024..0.680 rows=12 loops=1)
        Index Cond: (dept_id = d.dept_id)
Planning Time: 0.230 ms
Execution Time: 1.012 ms

```

- Оптимизатор решил: сначала найти отдел `HR` (1 строка), затем циклом подтащить сотрудников (Nested Loop).
    
- Если отделов много, но у `employees.dept_id` **нет индекса**, план сменится на **Hash Join** + **Seq Scan**. Узнать это можно, убрав индекс и повторив `EXPLAIN`.
    

---

## 4. Выявление «тяжёлых» мест

### a) Seq Scan вместо ожидаемого Index Scan

``` sql
Seq Scan on employees  (cost=0.00..18334.00 rows=20000 width=24)
  Filter: (dept_id = 5)
  Actual Rows: 20018  Loops: 1

```

**Почему случилось?**  
_Доля_ строк с `dept_id=5` слишком велика (20 k / 1 M = 2 %), и оптимизатор решил, что последовательное чтение дешевле, чем прыжки по индексу.  
**Что делать?**

1. Проверить статистику: `ANALYZE employees;`
    
2. Попробовать `SET enable_seqscan = off;` — посмотреть, быстрее ли с индексом.
    
3. Создать «покрывающий» индекс, включающий запрашиваемые колонки:

```sql
CREATE INDEX ON employees (dept_id) INCLUDE (name, salary);
```
### b) Переоценка количества строк

sql

КопироватьРедактировать

```sql
Hash Join  (cost=350..6000 rows=500 width=...)            
			(actual time=800..4500 rows=50000 loops=1)
```

`rows=500` vs `Actual Rows=50 000` → оценка в 100× меньше реальности. Такое вызывает слишком «скромный» план (Nested Loop, Hash Join small side), в итоге запрос медленнее.  
**Лекарства:** поднять `default_statistics_target`, собрать расширенную статистику `CREATE STATISTICS`, либо пересчитать гистограммы анализом.

---

## 5. Полезные опции `EXPLAIN`

|Опция|Что добавляет|Когда полезна|
|---|---|---|
|`BUFFERS`|Строку `Buffers:` (shared hit, read, dirtied, temp)|искать лишние чтения с диска|
|`TIMING` _(on по умолч.)_|Время на каждый узел|при огромных планах можно отключить, чтоб планировка не тормозила|
|`PARALLEL`|Сколько рабочих процессов занято|анализировать параллельные планы|
|`VERBOSE`|Имена внутренних столбцов|отладка сложных CTE/под‑запросов|
|`FORMAT JSON / YAML`|Машино‑читаемый вывод|визуализация в pgAdmin, AyaSLR и др.|

Пример:

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, FORMAT JSON) SELECT ...
```

---

## 6. Быстрый чек‑лист чтения плана

1. **Самый дорогой узел** — где «Execution Time»/«Actual Total Time» велик.
    
2. Сравнить `rows` vs `Actual Rows`. Если отличаются на порядки → плохая статистика.
    
3. Посмотреть `Buffers read=` — индикатор i/o.
    
4. При `Loops ≫ 1` убедиться, что внутренний узел действительно дешёвый (иначе N²‑эффект).
    
5. Проверить, нет ли `Hash Cond:`/`Merge Cond:` без индексов на соединяемые поля.
    
6. Убедиться, что `Parallel Workers Planned/Launched` > 0 для больших таблиц (если CPU простаивают).
    

---

### Заключение

- **`EXPLAIN`** — быстрый взгляд, _не_ выполняет запрос.
    
- **`EXPLAIN ANALYZE`** — выполняет и показывает реальные метрики; незаменим для оптимизации.
    
- Сравнивайте ожидаемые и фактические строки/время, смотрите буферы и количество циклов, включайте опции `BUFFERS`, `PARALLEL`, `FORMAT JSON` для глубины.
    
- Любые расхождения лечатся статистикой, правильными индексами и выбором подходящего алгоритма (Hash Join vs Nested Loop vs Merge Join).