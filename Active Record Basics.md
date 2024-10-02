## Основы Active Record Basics

---

### Что такое Active Record?

* **Active Record:** 
    * ORM (Object-Relational Mapping) фреймворк в Ruby on Rails.
    * Предоставляет объектно-ориентированный интерфейс для работы с базой данных.
* **Основные функции:**
    * Моделирование данных
    * Валидация данных
    * Взаимодействие с базой данных

---

### Создание модели

* **Генерация модели:** 
    * Создание новой модели с помощью генератора.

```bash
rails generate model Product name:string price:decimal
```

* **Миграция:** 
    * Создание таблицы в базе данных.

```bash
rails db:migrate
```

---

### Основные атрибуты модели

* **Атрибуты:** 
    * Свойства модели, соответствующие столбцам таблицы.
* **Пример модели:**

```ruby
class Product < ApplicationRecord
  # Атрибуты: id, name, price, created_at, updated_at
end
```

---

### CRUD операции

* **Создание (Create):**

```ruby
Product.create(name: 'Laptop', price: 1000)
```

* **Чтение (Read):**

```ruby
Product.find(1)
Product.where(name: 'Laptop')
```

* **Обновление (Update):**

```ruby
product = Product.find(1)
product.update(price: 1200)
```

* **Удаление (Delete):**

```ruby
product = Product.find(1)
product.destroy
```

---

### Валидация данных

* **Валидация:** 
    * Проверка данных перед сохранением в базу данных.
* **Пример валидации:**

```ruby
class Product < ApplicationRecord
  validates :name, presence: true
  validates :price, numericality: { greater_than: 0 }
end
```

---

### Ассоциации

* **Ассоциации:** 
    * Определяют связи между моделями.
* **Типы ассоциаций:**
    * `has_many`
    * `belongs_to`
    * `has_one`
    * `has_many :through`

```ruby
class Order < ApplicationRecord
  belongs_to :customer
  has_many :order_items
end
```

---

### Скоупы (Scopes)

* **Скоупы:** 
    * Предопределенные запросы, которые можно использовать в моделях.
* **Пример скоупа:**

```ruby
class Product < ApplicationRecord
  scope :expensive, -> { where('price > ?', 1000) }
end

Product.expensive
```

---

### Колбэки (Callbacks)

* **Колбэки:** 
    * Методы, вызываемые до или после определенных событий (создание, обновление, удаление).
* **Пример колбэка:**

```ruby
class Product < ApplicationRecord
  before_save :set_default_price

  private

  def set_default_price
    self.price ||= 0
  end
end
```

---

### Транзакции

* **Транзакции:** 
    * Группировка нескольких операций в одну, которая либо выполняется полностью, либо не выполняется вовсе.
* **Пример транзакции:**

```ruby
Product.transaction do
  product1 = Product.create(name: 'Laptop', price: 1000)
  product2 = Product.create(name: 'Phone', price: 500)
end
```

---

### Заключение

* **Active Record – мощный инструмент:** 
    * Упрощает работу с базой данных в Ruby on Rails.
    * Предоставляет множество функций для моделирования данных и управления ими.
* **Основные компоненты:** 
    * Модели, валидация, ассоциации, скоупы, колбэки, транзакции.

---

### Вопросы?

Спасибо за внимание!

---
