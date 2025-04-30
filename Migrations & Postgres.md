# Лекция: Чистка миграций, переезд на Postgres, production окружение в Rails

## Введение

Сегодня мы рассмотрим три важных аспекта профессиональной разработки на Ruby on Rails:
1. Управление миграциями базы данных
2. Переход с SQLite на PostgreSQL
3. Настройка production окружения

## Часть 1: Чистка миграций

### Проблемы с накоплением миграций

Со временем в проекте может накопиться большое количество файлов миграций, что приводит к:
- Увеличению времени тестирования
- Сложностям в понимании текущей структуры БД
- Потенциальным конфликтам при слиянии веток

### Решения

1. **Сброс миграций** (для разработки):
   ```bash
   rails db:drop db:create db:migrate
   ```

2. **Создание новой схемы**:
   ```bash
   rails db:schema:dump
   ```

3. **Удаление старых миграций** (после создания новой схемы):
   - Удалите все файлы из `db/migrate/`
   - Создайте одну новую миграцию с текущей структурой:
     ```bash
     rails generate migration InitialSchema
     ```

4. **Для production** используйте `db:schema:load` вместо `db:migrate`:
   ```bash
   rails db:schema:load RAILS_ENV=production
   ```

## Часть 2: Переход на PostgreSQL

### Почему PostgreSQL?

- Более производительный для production
- Поддержка сложных типов данных
- Расширенные возможности (JSON, полнотекстовый поиск и т.д.)
- Лучшая масштабируемость

### Установка PostgreSQL

**Ubuntu:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql.service
```

**Mac:**
```bash
brew install postgresql
```
Или используйте [Postgres.app](https://postgresapp.com/downloads.html)

### Настройка Rails для работы с PostgreSQL

1. Удалите `gem "sqlite3"` из Gemfile
2. Добавьте `gem "pg"`
3. Выполните `bundle install`

### Конфигурация database.yml

```yaml
default: &default
  adapter: postgresql
  encoding: utf8
  host: localhost
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: appname_dev

test:
  <<: *default
  database: appname_test

production:
  <<: *default
  database: appname
```

### Перенос данных

1. Создайте дамп SQLite:
   ```bash
   sqlite3 db/development.sqlite3 .dump > dump.sql
   ```

2. Очистите dump.sql от специфичных для SQLite команд

3. Импортируйте в PostgreSQL:
   ```bash
   psql appname_dev < dump.sql
   ```

## Часть 3: Production окружение

### Запуск в production

```bash
rails server -e production
# или
rails s RAILS_ENV=production
```

### Настройка базы данных

```bash
rails db:create RAILS_ENV=production
rails db:migrate RAILS_ENV=production
rails db:seed RAILS_ENV=production DISABLE_DATABASE_ENVIRONMENT_CHECK=1
```

### Подготовка ассетов

```ruby
# config/environments/production.rb
config.assets.compile = true
config.public_file_server.enabled = true
```

```bash
rails assets:precompile RAILS_ENV=production
```

### Настройка JavaScript

1. Обновите `.gitignore`:
   ```
   /app/assets/builds
   ```

2. Упростите `app/assets/config/manifest.js`:
   ```javascript
   //= link_tree ../images
   //= link_directory ../stylesheets .css
   //= link_tree ../builds
   ```

3. Установите необходимые пакеты:
   ```bash
   bun add @hotwired/turbo-rails trix @rails/actiontext @hotwired/stimulus
   ```

4. Обновите JavaScript контроллеры:
   ```javascript
   import './controllers';
   import { application } from "./app"
   ```

5. Запустите:
   ```bash
   bin/rails stimulus:manifest:update RAILS_ENV=production
   rails assets:precompile RAILS_ENV=production
   bin/dev
   ```

### Мониторинг логов

```bash
tail -f -n 300 log/production.log
```

## Заключение

Мы рассмотрели:
- Методы управления миграциями
- Процесс перехода на PostgreSQL
- Настройку production окружения
