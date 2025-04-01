# Тестирование в Ruby on Rails

## Введение в тестирование

Тестирование — критически важная часть разработки современных приложений. В Rails тестирование встроено в фреймворк "из коробки" и поддерживает несколько подходов:

1. **Test-Driven Development (TDD)**  
   - Сначала пишется тест, затем код
   - Красный → Зеленый → Рефакторинг
   - Фокус на технической реализации

2. **Behavior-Driven Development (BDD)**  
   - Описание поведения системы
   - Читаемый синтаксис (DSL)
   - Фокус на бизнес-ценность

## 1. Встроенные инструменты тестирования Rails

### 1.1 Минифреймворки

Rails включает:
- **Minitest** (по умолчанию)
- Поддержка **RSpec** (через гем)

Структура папок:
```
test/
  models/
  controllers/
  mailers/
  integration/
  system/
  helpers/
```

### 1.2 Типы тестов

| Тип теста         | Назначение                          | Примеры                  |
|-------------------|-------------------------------------|--------------------------|
| Модели            | Бизнес-логика, валидации            | UserTest                 |
| Контроллеры       | HTTP-ответы, экшены                 | PostsControllerTest      |
| Системные         | End-to-end в браузере               | UserFlowTest             |
| Интеграционные    | Взаимодействие компонентов          | CheckoutProcessTest      |
| Мэйлеры           | Тестирование email                  | UserMailerTest           |

## 2. Test-Driven Development (TDD) на практике

### 2.1 Цикл TDD

1. **Красная фаза**: Пишем падающий тест
2. **Зеленая фаза**: Пишем минимальный код для прохождения теста
3. **Рефакторинг**: Улучшаем код, сохраняя зеленый статус

### 2.2 Пример для модели

```ruby
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase
  test "should not save without email" do
    user = User.new
    assert_not user.save, "Saved user without email"
  end
end
```

Запуск:
```bash
rails test test/models/user_test.rb
```

## 3. Behavior-Driven Development (BDD) с RSpec

### 3.1 Установка RSpec

```ruby
# Gemfile
group :development, :test do
  gem 'rspec-rails'
  gem 'factory_bot_rails'
end
```

Установка:
```bash
bundle install
rails generate rspec:install
```

### 3.2 Пример теста

```ruby
# spec/models/user_spec.rb
require 'rails_helper'

RSpec.describe User, type: :model do
  describe "validations" do
    it { should validate_presence_of(:email) }
    it { should validate_uniqueness_of(:email).case_insensitive }
  end

  context "with admin role" do
    let(:admin) { create(:user, :admin) }
    
    it "has admin privileges" do
      expect(admin).to be_admin
    end
  end
end
```

## 4. Системное тестирование (System Tests)

### 4.1 Capybara + Selenium

```ruby
# test/system/users_test.rb
require "application_system_test_case"

class UsersTest < ApplicationSystemTestCase
  setup do
    @user = users(:one)
  end

  test "visiting the index" do
    visit users_url
    assert_selector "h1", text: "Users"
  end
end
```

### 4.2 Пример с RSpec

```ruby
# spec/system/user_flows_spec.rb
require 'rails_helper'

RSpec.describe "UserFlows", type: :system do
  before do
    driven_by(:selenium_chrome_headless)
  end

  scenario "User signs up" do
    visit new_user_registration_path
    
    fill_in "Email", with: "test@example.com"
    fill_in "Password", with: "password"
    click_button "Sign up"
    
    expect(page).to have_text("Welcome!")
  end
end
```

## 5. Фикстуры и фабрики

### 5.1 Фикстуры (YAML)

```yaml
# test/fixtures/users.yml
admin:
  email: admin@example.com
  password_digest: <%= BCrypt::Password.create('secret') %>
  role: admin
```

### 5.2 Фабрики (FactoryBot)

```ruby
# spec/factories/users.rb
FactoryBot.define do
  factory :user do
    email { "user#{rand(1000)}@example.com" }
    password { "password" }
    
    trait :admin do
      role { 'admin' }
    end
  end
end
```

## 6. Mock и Stub

### 6.1 Пример с mocha

```ruby
test "should call external service" do
  user = users(:one)
  ExternalService.any_instance.stubs(:call).returns(true)
  
  assert user.make_external_call
end
```

### 6.2 Пример с RSpec

```ruby
it "mocks API call" do
  allow(WeatherService).to receive(:forecast).and_return({ temp: 25 })
  
  result = WeatherService.forecast
  expect(result[:temp]).to eq(25)
end
```

## 7. Покрытие кода (Code Coverage)

Используем simplecov:

```ruby
# Gemfile
group :test do
  gem 'simplecov', require: false
end
```

```ruby
# spec/spec_helper.rb
require 'simplecov'
SimpleCov.start 'rails'
```

## 8. Лучшие практики

1. **3A Pattern**: Arrange-Act-Assert
2. **F.I.R.S.T принципы**:
   - Fast (быстрые)
   - Independent (независимые)
   - Repeatable (повторяемые)
   - Self-validating (самопроверяемые)
   - Timely (своевременные)
3. **Тестируйте поведение, а не реализацию**
4. **Избегайте избыточных тестов**
5. **Используйте describe/context правильно**

## Заключение

Современное тестирование в Rails включает:

1. **Многоуровневую стратегию** (юнит, интеграционные, системные тесты)
2. **Поддержку TDD/BDD подходов**
3. **Богатый набор инструментов** (Minitest, RSpec, Capybara)
4. **Гибкие методы** (фабрики, моки)

### Домашнее задание

1. Реализуйте TDD-цикл для новой модели
2. Напишите системный тест для ключевого сценария
3. Настройте измерение покрытия кода
4. Сравните Minitest и RSpec подходы

### Полезные ссылки

- [Официальный гайд по тестированию](https://guides.rubyonrails.org/testing.html)
- [TDD на Википедии](https://en.wikipedia.org/wiki/Test-driven_development)
- [BDD на Википедии](https://en.wikipedia.org/wiki/Behavior-driven_development)

Пишите стабильные приложения! 🚀
