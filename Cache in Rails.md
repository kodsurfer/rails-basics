# Кеширование в Ruby on Rails

## Введение в кеширование

Кеширование — это мощный механизм повышения производительности веб-приложений, который позволяет:

- Уменьшить нагрузку на сервер
- Сократить время отклика приложения
- Улучшить пользовательский опыт
- Снизить затраты на инфраструктуру

В Rails существует несколько уровней кеширования, которые мы рассмотрим:

## 1. Виды кеширования в Rails

### 1.1 Page Caching (Устаревший)

Кеширование целых страниц. Не рекомендуется для Rails 6+.

### 1.2 Action Caching (Устаревший)

Кеширование действий контроллера. Также устарел в современных версиях Rails.

### 1.3 Fragment Caching

Наиболее полезный и часто используемый тип кеширования.

```erb
<% @products.each do |product| %>
  <% cache product do %>
    <%= render product %>
  <% end %>
<% end %>
```

### 1.4 Russian Doll Caching

Вложенное кеширование (как матрешка):

```erb
<% cache @product do %>
  <%= render @product %>
  <% cache @product.comments do %>
    <%= render @product.comments %>
  <% end %>
<% end %>
```

### 1.5 Low-Level Caching

Ручное кеширование произвольных данных:

```ruby
class Product < ApplicationRecord
  def competing_price
    Rails.cache.fetch("#{cache_key}/competing_price", expires_in: 12.hours) do
      Competitor::API.find_price(id)
    end
  end
end
```

## 2. Настройка кеширования

### 2.1 Конфигурация

В файле `config/environments/production.rb`:

```ruby
config.action_controller.perform_caching = true
config.cache_store = :memory_store
```

### 2.2 Доступные хранилища

- `:memory_store` (по умолчанию для development)
- `:file_store`
- `:mem_cache_store`
- `:redis_cache_store`
- `:null_store` (для тестирования)

Пример настройки Redis:

```ruby
config.cache_store = :redis_cache_store, {
  url: ENV['REDIS_URL'],
  namespace: 'myapp:cache',
  expires_in: 1.day
}
```

## 3. Ключи кеширования

### 3.1 Автоматическое определение

Для Active Record объектов:

```ruby
product.cache_key # => "products/1-20230610123045"
```

### 3.2 Кастомные ключи

```erb
<% cache [current_user, @product] do %>
  ...
<% end %>
```

## 4. Условное кеширование

```erb
<% cache_if @product.published?, @product do %>
  <%= render @product %>
<% end %>
```

## 5. Экспирация кеша

### 5.1 По времени

```ruby
Rails.cache.fetch('recent_products', expires_in: 1.hour) do
  Product.recent.to_a
end
```

### 5.2 По событию

Использование touch:

```ruby
class Product < ApplicationRecord
  belongs_to :category, touch: true
end
```

## 6. Кеширование в контроллерах

```ruby
class ProductsController < ApplicationController
  caches_action :show, expires_in: 1.hour
  
  def show
    @product = Product.find(params[:id])
  end
end
```

## 7. Кеширование SQL-запросов

```ruby
def products_for_category(category)
  Rails.cache.fetch("products_for_category_#{category.id}_v2") do
    category.products.where('price > 100').to_a
  end
end
```

## 8. Практические примеры

### 8.1 Кеширование сложных вычислений

```ruby
class Report
  def generate
    Rails.cache.fetch("#{cache_key}/report", expires_in: 1.day) do
      # Длительные вычисления
      calculate_report
    end
  end
end
```

### 8.2 Кеширование API-ответов

```ruby
def weather_data
  Rails.cache.fetch('weather_data', expires_in: 30.minutes) do
    WeatherAPI.fetch(latitude, longitude)
  end
end
```

## 9. Очистка кеша

### 9.1 Ручная очистка

```ruby
Rails.cache.delete('recent_products')
```

### 9.2 Очистка по шаблону

```ruby
Rails.cache.delete_matched('products/*')
```

## 10. Лучшие практики

1. **Кешируйте правильно**: Не кешируйте данные, которые часто меняются
2. **Используйте разумные сроки**: Разные данные требуют разного времени жизни
3. **Мониторьте эффективность**: Следите за hit/miss ratio
4. **Разделяйте кеш**: Используйте namespaces для разных типов данных
5. **Тестируйте**: Убедитесь, что кеш корректно очищается

## Заключение

Кеширование — это мощный инструмент оптимизации Rails-приложений. Правильное использование кеширования позволяет значительно повысить производительность приложения и улучшить пользовательский опыт.

### Домашнее задание

1. Реализуйте fragment caching для списка товаров
2. Настройте Redis как хранилище кеша
3. Создайте low-level кеш для сложного SQL-запроса
4. Реализуйте механизм сброса кеша при изменении данных

### Полезные ссылки

- [Официальный гайд по кешированию](https://guides.rubyonrails.org/caching_with_rails.html)
- [Документация по Redis](https://github.com/redis/redis-rb)
- [Гем redis-rails](https://github.com/redis-store/redis-rails)

Удачного кеширования! 🚀
