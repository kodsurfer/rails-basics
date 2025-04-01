# Отложенные задачи в Ruby on Rails с Active Job и Resque

## Введение в отложенные задачи

Отложенные задачи (background jobs) - важная часть современных веб-приложений, позволяющая:
- Выполнять длительные операции асинхронно
- Улучшать отзывчивость приложения
- Обрабатывать задачи вне HTTP-запроса
- Планировать выполнение на будущее

В Rails для этого существует несколько решений, сегодня мы рассмотрим:
1. Active Job - стандартный интерфейс для фоновых задач
2. Resque - популярный бэкенд для обработки очередей

---

## 1. Active Job Basics

### Что такое Active Job?
Active Job - это стандартный интерфейс Rails для объявления фоновых задач и их запуска на различных бэкендах (Resque, Sidekiq, Delayed Job и др.)

### Основные возможности:
- Единый API для разных бэкендов
- Поддержка очередей
- Планирование выполнения
- Интеграция с Action Mailer

### Создание задачи

1. Генерация джоба:
```bash
rails generate job process_data
```

2. Реализация джоба:
```ruby
# app/jobs/process_data_job.rb
class ProcessDataJob < ApplicationJob
  queue_as :default

  def perform(user_id, data)
    user = User.find(user_id)
    # Длительная обработка данных
    user.process(data)
  end
end
```

3. Постановка в очередь:
```ruby
ProcessDataJob.perform_later(current_user.id, large_dataset)
```

### Варианты запуска:
- `perform_later` - постановка в очередь
- `perform_now` - немедленное выполнение
- `set(wait: 1.hour)` - отложенный запуск

---

## 2. Настройка Resque

### Что такое Resque?
Resque - система обработки фоновых задач, использующая Redis для хранения очередей.

### Установка

1. Добавляем гемы:
```ruby
# Gemfile
gem 'resque'
gem 'redis-rails'
```

2. Устанавливаем:
```bash
bundle install
```

3. Конфигурация:
```ruby
# config/initializers/resque.rb
Resque.redis = Redis.new(host: 'localhost', port: 6379)
```

4. Указываем Active Job использовать Resque:
```ruby
# config/application.rb
config.active_job.queue_adapter = :resque
```

### Запуск воркеров

1. Запуск одного воркера:
```bash
QUEUE=default rake resque:work
```

2. Запуск нескольких воркеров (с помощью Foreman):
```bash
# Procfile
worker: QUEUE=default rake resque:work
```

3. Для продакшена можно использовать systemd или upstart

---

## 3. Продвинутые техники

### Очереди

Разделение задач по очередям:
```ruby
class ProcessDataJob < ApplicationJob
  queue_as :high_priority
  # ...
end

class CleanupJob < ApplicationJob
  queue_as :low_priority
  # ...
end
```

Запуск воркеров для конкретных очередей:
```bash
QUEUE=high_priority,low_priority rake resque:work
```

### Планирование задач

Отложенный запуск:
```ruby
# Запустить через 1 час
ProcessDataJob.set(wait: 1.hour).perform_later(user.id, data)
```

Повторяющиеся задачи (с помощью resque-scheduler):
```ruby
# config/resque_schedule.yml
cleanup_job:
  every: "1d"
  class: "CleanupJob"
  queue: low_priority
  description: "Daily cleanup task"
```

### Обработка ошибок

1. Повторные попытки:
```ruby
class ProcessDataJob < ApplicationJob
  retry_on StandardError, wait: 5.minutes, attempts: 3
  
  def perform(user_id, data)
    # ...
  end
end
```

2. Кастомная обработка ошибок:
```ruby
rescue_from(ActiveRecord::RecordNotFound) do |exception|
  # Логирование ошибки
end
```

---

## 4. Мониторинг и администрирование

### Resque Web Interface

1. Добавляем гем:
```ruby
# Gemfile
gem 'resque-web', require: 'resque_web'
```

2. Маршруты:
```ruby
# config/routes.rb
mount ResqueWeb::Engine => "/resque_web"
```

Доступно по адресу `/resque_web`:
- Просмотр очередей
- Статистика выполнения
- Управление воркерами

### Логирование

Добавление логов в джобы:
```ruby
class ProcessDataJob < ApplicationJob
  def perform(user_id, data)
    Rails.logger.info "Starting data processing for user #{user_id}"
    # ...
  end
end
```

---

## 5. Лучшие практики

1. **Идемпотентность** - джобы должны безопасно выполняться несколько раз
2. **Небольшие задачи** - разбивайте большие задачи на несколько маленьких
3. **Обработка ошибок** - всегда предусматривайте сценарии сбоев
4. **Мониторинг** - следите за выполнением задач
5. **Ограничение ресурсов** - контролируйте использование памяти и CPU

Пример идемпотентного джоба:
```ruby
class ProcessPaymentJob < ApplicationJob
  def perform(payment_id)
    payment = Payment.find(payment_id)
    return if payment.processed? # Предотвращаем повторную обработку
    
    payment.process!
  end
end
```

---

## Заключение

1. **Active Job** предоставляет стандартный интерфейс для фоновых задач
2. **Resque** - надежный бэкенд на основе Redis
3. Правильная настройка очередей улучшает производительность
4. Мониторинг и обработка ошибок обязательны для production

### Домашнее задание

1. Создайте джоб для обработки изображений
2. Настройте Resque с двумя очередями
3. Реализуйте отложенный запуск задачи
4. Добавьте интерфейс мониторинга

### Полезные ссылки

- [Active Job Basics](https://guides.rubyonrails.org/active_job_basics.html)
- [Resque GitHub](https://github.com/resque/resque)
- [Resque and Active Job](https://github.com/resque/resque/wiki/ActiveJob)

Удачной разработки! 🚀
