## Учебная лекция: Создание API на Rails с использованием Grape

### Введение

В современном мире веб-разработки API (Application Programming Interface) играют ключевую роль в обеспечении взаимодействия между различными сервисами и приложениями. Ruby on Rails, мощный фреймворк для разработки веб-приложений, предоставляет удобные инструменты для создания API. Однако, для более гибкой и сложной маршрутизации, многие разработчики используют дополнительные гемы, такие как Grape. В этой лекции мы рассмотрим, как создать API на Rails с использованием Grape, используя видеоурок по ссылке: [Создание API на Rails с использованием Grape](https://youtu.be/3a2e14xOhjw).

### 1. Настройка проекта

#### 1.1. Создание нового Rails приложения

Для начала создадим новое Rails приложение, которое будет служить основой для нашего API:

```bash
rails new my_api --api
```

Опция `--api` указывает Rails создать приложение в режиме API, что означает, что будут исключены компоненты, не нужные для API, такие как представления (views) и middleware для обработки HTML.

#### 1.2. Добавление Grape в проект

Добавим гем Grape в наш проект. Откройте файл `Gemfile` и добавьте следующую строку:

```ruby
gem 'grape'
```

Затем выполните команду для установки гемов:

```bash
bundle install
```

### 2. Создание моделей и миграций

#### 2.1. Генерация модели

Создадим модель `Product` с атрибутами `name` и `price`:

```bash
rails generate model Product name:string price:decimal
```

Эта команда создаст файл модели `app/models/product.rb` и файл миграции для создания соответствующей таблицы в базе данных.

#### 2.2. Выполнение миграции

Примените миграцию для создания таблицы в базе данных:

```bash
rails db:migrate
```

### 3. Создание API с использованием Grape

#### 3.1. Создание базового API класса

Создадим директорию `app/api` и в ней файл `base.rb`, который будет служить базовым классом для нашего API:

```ruby
# app/api/base.rb
class API < Grape::API
  format :json

  mount V1::Base
end
```

#### 3.2. Создание версии API

Создадим директорию `app/api/v1` и в ней файл `base.rb`, который будет служить базовым классом для версии 1 нашего API:

```ruby
# app/api/v1/base.rb
module V1
  class Base < Grape::API
    mount V1::Products
  end
end
```

#### 3.3. Создание ресурса Products

Создадим файл `products.rb` в директории `app/api/v1`, который будет обрабатывать запросы к ресурсу `Products`:

```ruby
# app/api/v1/products.rb
module V1
  class Products < Grape::API
    resource :products do
      desc 'Return all products'
      get do
        Product.all
      end

      desc 'Return a product'
      params do
        requires :id, type: Integer, desc: 'Product ID'
      end
      get ':id' do
        Product.find(params[:id])
      end

      desc 'Create a product'
      params do
        requires :name, type: String, desc: 'Product name'
        requires :price, type: BigDecimal, desc: 'Product price'
      end
      post do
        Product.create!(name: params[:name], price: params[:price])
      end

      desc 'Update a product'
      params do
        requires :id, type: Integer, desc: 'Product ID'
        requires :name, type: String, desc: 'Product name'
        requires :price, type: BigDecimal, desc: 'Product price'
      end
      put ':id' do
        product = Product.find(params[:id])
        product.update(name: params[:name], price: params[:price])
        product
      end

      desc 'Delete a product'
      params do
        requires :id, type: Integer, desc: 'Product ID'
      end
      delete ':id' do
        product = Product.find(params[:id])
        product.destroy
      end
    end
  end
end
```

### 4. Подключение Grape к Rails

#### 4.1. Настройка маршрутов

В файле `config/routes.rb` подключим наш API:

```ruby
Rails.application.routes.draw do
  mount API => '/'
end
```

### 5. Тестирование API

#### 5.1. Использование Postman

Для тестирования API можно использовать инструмент Postman. Создайте запросы к различным эндпоинтам и убедитесь, что они работают корректно.

#### 5.2. Автоматизированное тестирование

Rails также предоставляет инструменты для автоматизированного тестирования. Создайте файл `test/controllers/api/v1/products_test.rb` и напишите тесты для проверки работы API:

```ruby
require 'test_helper'

class ProductsTest < ActionDispatch::IntegrationTest
  test "should get all products" do
    get '/api/v1/products'
    assert_response :success
  end

  test "should create product" do
    post '/api/v1/products', params: { name: 'New Product', price: 10.99 }
    assert_response :success
  end

  test "should get product" do
    product = Product.create(name: 'Test Product', price: 9.99)
    get "/api/v1/products/#{product.id}"
    assert_response :success
  end

  test "should update product" do
    product = Product.create(name: 'Test Product', price: 9.99)
    put "/api/v1/products/#{product.id}", params: { name: 'Updated Product', price: 11.99 }
    assert_response :success
  end

  test "should delete product" do
    product = Product.create(name: 'Test Product', price: 9.99)
    delete "/api/v1/products/#{product.id}"
    assert_response :success
  end
end
```

### 6. Заключение

В этой лекции мы рассмотрели, как создать API на Rails с использованием Grape. Мы создали новое Rails приложение в режиме API, добавили гем Grape, создали модели и миграции, определили маршруты и протестировали API. Grape предоставляет мощные инструменты для гибкой и сложной маршрутизации, что делает его отличным выбором для разработчиков, которым нужна большая гибкость в создании API.

### Дополнительные ресурсы

- [Официальная документация Rails](https://guides.rubyonrails.org/)
- [Документация Grape](https://github.com/ruby-grape/grape)
- [Postman](https://www.postman.com/)
- [RSpec для тестирования в Rails](https://rspec.info/)

Надеюсь, эта лекция помогла вам понять основы создания API на Rails с использованием Grape. Удачи в ваших проектах!
