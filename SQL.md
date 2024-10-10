## Основы SQL

---

### Что такое SQL?

* **SQL (Structured Query Language):** 
    * Язык программирования для управления реляционными базами данных.
    * Используется для создания, модификации и управления данными в БД.
* **Основные операции:**
    * Выборка данных (SELECT)
    * Вставка данных (INSERT)
    * Обновление данных (UPDATE)
    * Удаление данных (DELETE)

---

### Создание таблиц

* **CREATE TABLE:** 
    * Создание новой таблицы в базе данных.
    * Определение столбцов и их типов данных.

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
);
```

---

### Вставка данных

* **INSERT INTO:** 
    * Добавление новых записей в таблицу.
    * Указываются столбцы и значения для вставки.

```sql
INSERT INTO employees (id, name, department, salary)
VALUES (1, 'John Doe', 'Sales', 50000.00);
```

---

### Выборка данных

* **SELECT:** 
    * Извлечение данных из таблицы.
    * Можно указать столбцы для выборки и условия фильтрации.

```sql
SELECT name, department, salary
FROM employees
WHERE department = 'Sales';
```

---

### Обновление данных

* **UPDATE:** 
    * Изменение существующих записей в таблице.
    * Указываются столбцы и новые значения.

```sql
UPDATE employees
SET salary = 55000.00
WHERE id = 1;
```

---

### Удаление данных

* **DELETE:** 
    * Удаление записей из таблицы.
    * Указываются условия для удаления.

```sql
DELETE FROM employees
WHERE id = 1;
```

---

### Соединения таблиц

* **JOIN:** 
    * Объединение данных из нескольких таблиц на основе связанных столбцов.
    * Типы соединений: INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL JOIN.

```sql
SELECT employees.name, departments.name
FROM employees
INNER JOIN departments ON employees.department_id = departments.id;
```

---

### Группировка и агрегация

* **GROUP BY:** 
    * Группировка строк по значениям в указанных столбцах.
* **Агрегатные функции:** 
    * SUM, AVG, COUNT, MIN, MAX.

```sql
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department;
```

---

### Подзапросы

* **Подзапросы:** 
    * Запросы, вложенные в другие запросы.
    * Могут использоваться для фильтрации, выборки или обновления данных.

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

---

### Индексы

* **Индексы:** 
    * Структуры данных, ускоряющие поиск и сортировку данных.
    * Создаются на основе одного или нескольких столбцов.

```sql
CREATE INDEX idx_department ON employees (department);
```

---

### Заключение

* **SQL – мощный инструмент:** 
    * Управление данными в реляционных базах данных.
    * Гибкость и выразительность.
* **Основные операции:** 
    * CRUD (Create, Read, Update, Delete).

---

### Вопросы?

Спасибо за внимание!

---
