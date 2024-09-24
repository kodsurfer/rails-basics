## Основы Ruby on Rails

---

### Что такое Ruby on Rails?

* **Ruby on Rails (RoR):** 
    * Веб-фреймворк, написанный на языке Ruby.
    * Использует паттерн MVC (Model-View-Controller).
    * Обеспечивает быструю разработку веб-приложений.
* **Основные принципы:**
    * DRY (Don't Repeat Yourself)
    * Convention Over Configuration

---

### Установка Ruby on Rails

* **Требования:** 
    * Ruby (рекомендуется последняя версия)
    * Bundler (менеджер зависимостей)
    * SQLite (по умолчанию)
* **Установка:**

```bash
gem install rails
```

---

### Создание нового проекта

* **Генерация проекта:**

```bash
rails new my_app
cd my_app
```

* **Запуск сервера:**

```bash
rails server
```

* **Открыть в браузере:**

```
http://localhost:3000
```

---

### Структура проекта

* **app/:** 
    * Основной код приложения (модели, контроллеры, представления).
* **config/:** 
    * Конфигурационные файлы (маршруты, базы данных).
* **db/:** 
    * Скрипты для управления базой данных (миграции, схемы).
* **public/:** 
    * Статические файлы (изображения, CSS, JavaScript).

---

### Модели (Models)

* **Модели:** 
    * Представляют данные и бизнес-логику.
    * Наследуются от `ApplicationRecord`.
* **Генерация модели:**

```bash
rails generate model Product name:string price:decimal
```

* **Миграции:**

```bash
rails db:migrate
```

---

### Контроллеры (Controllers)

* **Контроллеры:** 
    * Обрабатывают запросы и управляют логикой приложения.
    * Наследуются от `ApplicationController`.
* **Генерация контроллера:**

```bash
rails generate controller Products
```

* **Пример метода контроллера:**

```ruby
class ProductsController < ApplicationController
  def index
    @products = Product.all
  end
end
```

---

### Представления (Views)

* **Представления:** 
    * Отображают данные пользователю.
    * Используют шаблонизатор ERB (Embedded Ruby).
* **Пример представления:**

```erb
<h1>Products</h1>
<ul>
  <% @products.each do |product| %>
    <li><%= product.name %> - <%= product.price %></li>
  <% end %>
</ul>
```

---

### Маршрутизация (Routing)

* **Маршрутизация:** 
    * Определяет, какой контроллер и метод будут обрабатывать запросы.
* **Пример маршрута:**

```ruby
Rails.application.routes.draw do
  get 'products', to: 'products#index'
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

### Валидация

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

### Заключение

* **Ruby on Rails – мощный фреймворк:** 
    * Ускоряет разработку веб-приложений.
    * Использует принципы DRY и Convention Over Configuration.
* **Основные компоненты:** 
    * Модели, контроллеры, представления, маршрутизация.

---

### Вопросы?

Спасибо за внимание!

---
