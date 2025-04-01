# Работа с Rake-тасками в Ruby on Rails

## Введение в Rake

Rake (Ruby Make) — это утилита сборки на Ruby, аналогичная make в Unix-системах, но с более мощными возможностями благодаря использованию Ruby.

### Основные возможности:
- Автоматизация повторяющихся задач
- Создание сложных цепочек зависимостей
- Использование всего богатства Ruby в задачах
- Интеграция с Rails-экосистемой

## 1. Основы Rake

### 1.1 Структура задачи

Базовый синтаксис Rake-задачи:

```ruby
task :name do
  # Код задачи
end
```

Пример простой задачи:

```ruby
task :greet do
  puts "Hello, Rake!"
end
```

Выполнение:
```bash
rake greet
```

### 1.2 Пространства имен

Для организации задач:

```ruby
namespace :db do
  task :backup do
    puts "Creating database backup..."
  end
end
```

Вызов:
```bash
rake db:backup
```

## 2. Rake в Rails

Rails активно использует Rake для различных задач:

### 2.1 Стандартные Rails-задачи

Основные команды:
- `rake db:migrate`
- `rake assets:precompile`
- `rake test`
- `rake routes`

Полный список:
```bash
rake -T
```

### 2.2 Создание собственных задач

Файлы задач размещаются в `lib/tasks` с расширением `.rake`

Пример `lib/tasks/data_cleanup.rake`:

```ruby
namespace :data do
  desc "Cleanup old records"
  task cleanup: :environment do
    OldRecordsCleaner.call
  end
end
```

## 3. Продвинутые техники

### 3.1 Зависимости между задачами

```ruby
task backup: ['db:backup', 'files:backup'] do
  puts "Full backup completed"
end
```

### 3.2 Передача аргументов

```ruby
task :greet, [:name] do |t, args|
  puts "Hello, #{args.name}!"
end
```

Вызов:
```bash
rake greet[John]
```

### 3.3 Доступ к окружению Rails

```ruby
task import_data: :environment do
  # Теперь доступны все модели Rails
  User.import_from_csv
end
```

## 4. Практические примеры

### 4.1 Импорт данных

```ruby
namespace :data do
  desc "Import users from CSV"
  task :import_users, [:file] => :environment do |t, args|
    require 'csv'
    
    CSV.foreach(args.file, headers: true) do |row|
      User.create!(row.to_h)
    end
    
    puts "Imported #{User.count} users"
  end
end
```

### 4.2 Периодические задачи

Пример задачи для cron:

```ruby
task cleanup_sessions: :environment do
  ActiveRecord::SessionStore::Session.where('updated_at < ?', 1.week.ago).delete_all
end
```

### 4.3 Сложные цепочки задач

```ruby
namespace :deploy do
  task preflight: :environment do
    # Проверки перед деплоем
  end
  
  task migrate: :environment do
    # Миграции БД
  end
  
  task restart: :environment do
    # Перезапуск сервера
  end
  
  task full: ['deploy:preflight', 'deploy:migrate', 'deploy:restart'] do
    puts "Deploy completed!"
  end
end
```

## 5. Тестирование Rake-задач

Пример теста с RSpec:

```ruby
require 'rake'
require 'rails_helper'

RSpec.describe 'data:import_users', type: :task do
  before do
    Rails.application.load_tasks
  end

  it 'imports users from CSV' do
    expect {
      Rake::Task['data:import_users'].invoke('spec/fixtures/users.csv')
    }.to change(User, :count).by(2)
  end
end
```

## 6. Лучшие практики

1. **Используйте пространства имен** для организации задач
2. **Добавляйте описания** (desc) для всех задач
3. **Разделяйте большие задачи** на мелкие подзадачи
4. **Обрабатывайте ошибки** в задачах
5. **Используйте окружение Rails** (:environment) для доступа к моделям
6. **Документируйте** параметры задач
7. **Тестируйте** сложные задачи

Пример хорошо оформленной задачи:

```ruby
namespace :reports do
  desc "Generate monthly sales report"
  task :monthly_sales, [:year, :month] => :environment do |t, args|
    begin
      year = args.year || Date.current.year
      month = args.month || Date.current.month
      
      report = SalesReport.generate(year: year, month: month)
      report.export_to_s3
      
      puts "Report generated: #{report.url}"
    rescue => e
      puts "Error generating report: #{e.message}"
      raise
    end
  end
end
```

## 7. Полезные гемы

1. **whenever** - для планирования задач через cron
2. **airbrake** - для обработки ошибок в задачах
3. **rake-notes** - для поиска TODO в коде
4. **parallel_tests** - для параллельного выполнения задач

## Заключение

Rake — мощный инструмент автоматизации в Rails-приложениях. Правильное использование Rake-задач позволяет:

- Упростить рутинные операции
- Автоматизировать процессы развертывания
- Создавать сложные цепочки задач
- Интегрировать с другими системами

### Домашнее задание

1. Создайте задачу для импорта данных из JSON
2. Реализуйте задачу с зависимостями
3. Напишите тест для Rake-задачи
4. Настройте выполнение задачи по расписанию (через whenever)

### Полезные ссылки

- [Официальный репозиторий Rake](https://github.com/ruby/rake)
- [Документация Rake](https://ruby.github.io/rake/)
- [Rails Rake Tasks](https://guides.rubyonrails.org/command_line.html#rake)

Удачной автоматизации! 🚀
