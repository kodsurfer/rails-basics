### Лекция: Продвинутые Views в Ruby on Rails (расширенная версия)

В этой лекции мы углубимся в продвинутые техники работы с Views в Ruby on Rails. Мы рассмотрим не только базовые концепции, но и более сложные подходы, которые помогут вам создавать мощные, гибкие и производительные веб-приложения.

---

## 1. **Layouts and Rendering (Макеты и рендеринг)**

### 1.1. **Макеты (Layouts)**
Макеты — это шаблоны, которые определяют общую структуру страницы. Они позволяют избежать дублирования кода, такого как заголовки, футеры и навигационные панели.

#### Пример макета:
```erb
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>MyApp</title>
  <%= stylesheet_link_tag 'application', media: 'all' %>
  <%= javascript_include_tag 'application' %>
  <%= csrf_meta_tags %>
</head>
<body>
  <header>
    <h1>Welcome to MyApp</h1>
  </header>
  <main>
    <%= yield %> <!-- Сюда подставляется содержимое конкретного view -->
  </main>
  <footer>
    <p>&copy; 2023 MyApp</p>
  </footer>
</body>
</html>
```

#### Использование разных макетов:
Вы можете использовать разные макеты для разных контроллеров или действий:
```ruby
class AdminController < ApplicationController
  layout 'admin' # Использует app/views/layouts/admin.html.erb
end
```

#### Вложенные макеты:
Иногда нужно создать макет, который расширяет другой макет. Это можно сделать с помощью `content_for` и `yield`:
```erb
# app/views/layouts/admin.html.erb
<%= render template: 'layouts/application' %>
<%= content_for :admin_sidebar do %>
  <aside>
    <ul>
      <li>Dashboard</li>
      <li>Settings</li>
    </ul>
  </aside>
<% end %>
```

---

### 1.2. **Частичные шаблоны (Partials)**
Частичные шаблоны позволяют выносить повторяющийся код в отдельные файлы. Это упрощает поддержку и делает код более читаемым.

#### Пример partial:
```erb
# app/views/shared/_header.html.erb
<header>
  <h1>MyApp</h1>
  <nav>
    <%= link_to 'Home', root_path %>
    <%= link_to 'About', about_path %>
  </nav>
</header>
```

#### Использование partial:
```erb
# app/views/layouts/application.html.erb
<%= render 'shared/header' %>
<main>
  <%= yield %>
</main>
```

#### Локальные переменные в partial:
Вы можете передавать локальные переменные в partial:
```erb
<%= render partial: 'user', locals: { user: @user } %>
```

---

### 1.3. **Рендеринг**
Rails поддерживает различные способы рендеринга:
- **Рендеринг шаблонов**: По умолчанию Rails рендерит шаблон, соответствующий действию контроллера.
- **Рендеринг JSON/XML**: Полезно для API.
- **Рендеринг текста**: Простой вывод текста.

#### Пример рендеринга JSON:
```ruby
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    render json: @user
  end
end
```

---

## 2. **Asset Pipeline (Управление активами)**

Asset Pipeline — это механизм для управления CSS, JavaScript и изображениями. Он включает:
- Минификацию и объединение файлов.
- Использование препроцессоров (Sass, CoffeeScript).
- Автоматическую генерацию хэшей для кэширования.

### 2.1. **Sass и SCSS**
Sass — это препроцессор CSS, который добавляет возможности вложенности, переменных, миксинов и других функций.

#### Пример использования Sass:
```scss
# app/assets/stylesheets/application.scss
$primary-color: #333;

body {
  background-color: $primary-color;
  font-family: Arial, sans-serif;
}

.header {
  h1 {
    color: white;
  }
}
```

### 2.2. **JavaScript с CoffeeScript**
CoffeeScript — это язык, который компилируется в JavaScript. Он делает код более лаконичным.

#### Пример CoffeeScript:
```coffee
# app/assets/javascripts/application.coffee
alert "Hello, World!"
```

---

## 3. **Мета-теги и SEO**

Для управления мета-тегами можно использовать гем `meta-tags`. Он позволяет легко задавать title, description, keywords и другие мета-теги.

### Пример:
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :prepare_meta_tags

  def prepare_meta_tags
    set_meta_tags title: 'MyApp',
                  description: 'Welcome to MyApp',
                  keywords: 'ruby, rails, web development',
                  og: {
                    title: 'MyApp',
                    type: 'website',
                    url: request.original_url,
                    image: image_url('logo.png')
                  }
  end
end
```

---

## 4. **Загрузка файлов с CarrierWave**

CarrierWave — это гем для загрузки файлов. Он поддерживает:
- Интеграцию с облачными хранилищами (AWS S3, Google Cloud Storage).
- Оптимизацию изображений (с использованием гемов `carrierwave-imageoptimizer`, `jpegoptim`, `pngquant`).

### Пример:
```ruby
# app/uploaders/image_uploader.rb
class ImageUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick

  storage :file

  version :thumb do
    process resize_to_fit: [100, 100]
  end
end
```

---

## 5. **Интернационализация и локализация (I18n)**

Rails поддерживает интернационализацию, что позволяет адаптировать приложение для разных языков.

### Пример:
```yaml
# config/locales/ru.yml
ru:
  hello: "Привет"
  welcome: "Добро пожаловать в MyApp"
```

### Использование в views:
```erb
<%= t('hello') %>
<%= t('welcome') %>
```

---

## 6. **Красивые ссылки с FriendlyId**

FriendlyId позволяет создавать читаемые URL-адреса, используя человеко-понятные строки вместо числовых идентификаторов.

### Пример:
```ruby
# app/models/post.rb
class Post < ApplicationRecord
  extend FriendlyId
  friendly_id :title, use: :slugged
end
```

### Транслитерация для кириллицы:
Для автоматической транслитерации можно использовать гем `babosa`:
```ruby
class Post < ApplicationRecord
  extend FriendlyId
  friendly_id :title, use: :slugged

  def normalize_friendly_id(text)
    text.to_slug.normalize(transliterations: :russian).to_s
  end
end
```

---

## Заключение

Использование продвинутых техник работы с Views в Ruby on Rails позволяет создавать более гибкие, производительные и удобные для пользователя интерфейсы. Освоение этих инструментов поможет вам значительно улучшить качество ваших веб-приложений.

### Дополнительные ресурсы:
- [Rails Guides: Layouts and Rendering](https://guides.rubyonrails.org/layouts_and_rendering.html)
- [Sass-Rails](https://github.com/rails/sass-rails)
- [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html)
- [Meta-Tags](https://github.com/kpumuk/meta-tags)
- [CarrierWave](https://github.com/carrierwaveuploader/carrierwave)
- [FriendlyId](https://github.com/norman/friendly_id)
- [I18n in Rails](https://guides.rubyonrails.org/i18n.html)

Используйте эти ресурсы для более глубокого изучения темы и улучшения ваших навыков в веб-разработке на Ruby on Rails.
