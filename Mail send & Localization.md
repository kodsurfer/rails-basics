## Лекция: Отправка почты и перевод интерфейса в Ruby on Rails

---

### Часть 1: Отправка почты в Rails

#### 1.1. Основы Action Mailer

Action Mailer — это встроенный в Rails механизм для отправки электронной почты. Он работает аналогично контроллерам и представлениям, но предназначен для создания и отправки писем.

##### Создание почтового класса

Для создания почтового класса используется генератор:
```bash
rails generate mailer UserMailer
```
Это создаст класс `UserMailer` в папке `app/mailers` и соответствующие шаблоны в `app/views/user_mailer`.

##### Пример почтового класса

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email(user)
    @user = user
    @url  = 'http://example.com/login'
    mail(to: @user.email, subject: 'Добро пожаловать на наш сайт!')
  end
end
```

##### Шаблон письма

Шаблоны писем находятся в `app/views/user_mailer`. Например, `welcome_email.html.erb`:
```erb
<h1>Добро пожаловать, <%= @user.name %>!</h1>
<p>Вы успешно зарегистрировались на нашем сайте.</p>
<p>Для входа перейдите по ссылке: <%= @url %></p>
```

#### 1.2. Отправка писем

Для отправки письма достаточно вызвать метод почтового класса:
```ruby
UserMailer.welcome_email(@user).deliver_now
```
Или отложить отправку с помощью Active Job:
```ruby
UserMailer.welcome_email(@user).deliver_later
```

#### 1.3. Тестирование писем

Для тестирования писем в development-окружении можно использовать гем `letter_opener`, который открывает письма в браузере вместо реальной отправки.

##### Установка `letter_opener`

Добавьте в `Gemfile`:
```ruby
gem 'letter_opener', group: :development
```
И настройте `config/environments/development.rb`:
```ruby
config.action_mailer.delivery_method = :letter_opener
```

#### 1.4. Стилизация писем

Для стилизации писем рекомендуется использовать inline CSS, так как многие почтовые клиенты не поддерживают внешние стили. Ресурсы для работы с inline CSS:
- [Mailchimp Inline CSS Tool](https://templates.mailchimp.com/resources/inline-css/)
- [HTML to Plain Text Converter](https://templates.mailchimp.com/resources/html-to-text/)

---

### Часть 2: Локализация интерфейса (i18n)

#### 2.1. Основы I18n в Rails

Rails предоставляет встроенную поддержку интернационализации (i18n) через модуль `I18n`. Это позволяет переводить интерфейс на разные языки.

##### Структура файлов локализации

Файлы локализации хранятся в `config/locales`. Например, `config/locales/en.yml` и `config/locales/ru.yml`.

##### Пример файла локализации

```yaml
ru:
  hello: "Привет, мир!"
  user:
    welcome: "Добро пожаловать, %{name}!"
```

##### Использование переводов

В коде приложения можно использовать метод `t`:
```ruby
t('hello') # => "Привет, мир!"
t('user.welcome', name: @user.name) # => "Добро пожаловать, Иван!"
```

#### 2.2. Локализация моделей

Для перевода атрибутов моделей можно использовать файлы локализации:
```yaml
ru:
  activerecord:
    attributes:
      user:
        email: "Электронная почта"
        password: "Пароль"
```

#### 2.3. Локализация форм

Для перевода кнопок и других элементов форм:
```erb
<%= form_with model: @user do |f| %>
  <%= f.label :email %>
  <%= f.text_field :email %>
  <%= f.submit t('helpers.submit.user.create') %>
<% end %>
```

##### Пример перевода для кнопки:

```yaml
ru:
  helpers:
    submit:
      user:
        create: "Создать пользователя"
```

#### 2.4. Смена языка

Для смены языка можно использовать параметр `locale` в URL или настройку в контроллере:
```ruby
before_action :set_locale

def set_locale
  I18n.locale = params[:locale] || I18n.default_locale
end
```

---

### Часть 3: Смена/выбор лейаута для рендеринга

#### 3.1. Настройка лейаутов

Rails позволяет выбирать разные лейауты для разных действий или контроллеров. Лейауты хранятся в `app/views/layouts`.

##### Пример выбора лейаута

```ruby
class UsersController < ApplicationController
  layout "special", only: [:new, :create]
end
```

#### 3.2. Динамический выбор лейаута

Можно динамически выбирать лейаут в зависимости от условий:
```ruby
class UsersController < ApplicationController
  layout :choose_layout

  private

  def choose_layout
    current_user.admin? ? "admin" : "application"
  end
end
```

---

#### Дополнительные ресурсы:
- [Action Mailer Basics](https://guides.rubyonrails.org/action_mailer_basics.html)
- [I18n in Rails](https://guides.rubyonrails.org/i18n.html)
- [Layouts and Rendering](https://guides.rubyonrails.org/layouts_and_rendering.html)
