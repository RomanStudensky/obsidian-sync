
**DDL** (Data Definition Language) – для определения структуры данных.

Команды: CREATE, ALTER, DROP, TRUNCATE, RENAME.

Пример: 
``` sql
CREATE TABLE employees (id INT, name VARCHAR(100));
```

**DML** (Data Manipulation Language) — для манипуляции данными.

Команды: SELECT, INSERT, UPDATE, DELETE, MERGE.

Пример: 
``` sql
INSERT INTO employees (id, name) VALUES (1, 'Иван Иванов');
```

**DCL** (Data Control Language) — для управления доступом.

Команды: GRANT, REVOKE.

Пример: 
``` sql
GRANT SELECT ON employees TO user_name;
```


**TCL** (Transaction Control Language) — для управления транзакциями.

Команды: COMMIT, ROLLBACK, SAVEPOINT, SET TRANSACTION.

Пример: `COMMIT`; — подтверждение изменений в транзакции.


**DQL** (Data Query Language) — для выполнения запросов к данным.

Команды: `SELECT`.

Примечание: иногда SELECT включают в DML, но некоторые специалисты выделяют его в отдельную категорию DQL.

Пример: 
```sql
SELECT * FROM employees;
```