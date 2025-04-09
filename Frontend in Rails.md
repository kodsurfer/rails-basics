# Лекция: Фронтенд в Ruby on Rails

## Введение
Rails — это full-stack фреймворк, и его подход к фронтенду постоянно эволюционирует. Рассмотрим современные практики 2024 года:

---

## 1. Основные подходы

### 1.1 Asset Pipeline (устаревший)
- Управление CSS/JS через `app/assets`
- Компиляция SASS/CoffeeScript
- Проблемы: сложность с npm-пакетами

```ruby
# application.js
//= require jquery
//= require_tree .
```

### 1.2 Webpacker (deprecated)
- Использование Webpack 4
- Интеграция npm-пакетов
- С 2023 заменен на новые подходы

### 1.3 Modern Bundlers (Rails 7+)
```bash
rails new myapp -j esbuild   # или -j bun, -j rollup
rails new myapp --css=tailwind
```

Варианты:
- **esbuild** (быстрый, Go-based)
- **vite** (горячая перезагрузка)
- **Import Maps** (без сборки)

---

## 2. Import Maps
Стандартный подход для Rails 7+

### 2.1 Настройка
```ruby
# Gemfile
gem "importmap-rails"
```

```bash
rails importmap:install
```

### 2.2 Использование
```ruby
# config/importmap.rb
pin "application"
pin "@hotwired/turbo-rails", to: "turbo.min.js"
pin "react", to: "https://ga.jspm.io/npm:react@18.2.0/index.js"
```

```erb
<%= javascript_importmap_tags %>
```

---

## 3. Hotwire (HTML-over-the-wire)
Современный подход для SPA-like приложений

### 3.1 Turbo Drive
- Ускорение навигации
- Частичная загрузка страниц

```erb
<%= link_to "Profile", profile_path, data: { turbo_action: "advance" } %>
```

### 3.2 Turbo Frames
```erb
<%= turbo_frame_tag "messages" do %>
  <% @messages.each do |message| %>
    <%= render message %>
  <% end %>
<% end %>
```

### 3.3 Turbo Streams
Real-time обновления через WebSocket:

```ruby
# Контроллер
def create
  @message = Message.new(message_params)
  if @message.save
    render turbo_stream: turbo_stream.append(:messages, @message)
  end
end
```

---

## 4. Stimulus
Минималистичный JS-фреймворк для интерактивности

```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "results"]
  
  search() {
    fetch(`/search?q=${this.inputTarget.value}`)
      .then(response => response.text())
      .then(html => this.resultsTarget.innerHTML = html)
  }
}
```

```erb
<div data-controller="search">
  <input type="text" data-search-target="input">
  <div data-search-target="results"></div>
</div>
```

---

## 5. Работа со стилями

### 5.1 CSS Bundling
```bash
rails css:install:tailwind
```

Варианты:
- Tailwind
- Bootstrap
- PostCSS
- Sass

### 5.2 View Components
Компонентный подход:

```ruby
# app/components/button_component.rb
class ButtonComponent < ViewComponent::Base
  def initialize(color: :blue)
    @color = color
  end
end
```

```erb
<%# app/components/button_component.html.erb %>
<button class="btn btn-<%= @color %>">
  <%= content %>
</button>
```

---

## 6. Интеграция с React/Vue
Через jsbundling-rails:

```bash
rails javascript:install:react
```

```javascript
// app/javascript/components/Hello.jsx
export default function Hello({ name }) {
  return <div>Hello, {name}!</div>
}
```

```erb
<%= react_component("Hello", { name: "World" }) %>
```

---

## 7. Best Practices

1. **Избегайте jQuery** в новых проектах
2. **Используйте Turbo** для навигации
3. **Добавляйте JS только когда необходимо**
4. **Тестируйте** системными тестами
5. **Оптимизируйте загрузку** (ленивая загрузка)

---

## Пример: Редактор статей

```erb
<%# app/views/articles/show.html.erb %>
<%= turbo_frame_tag "article_content" do %>
  <article data-controller="content-editor">
    <div data-content-editor-target="content"><%= @article.content %></div>
    <%= button_to "Edit", edit_article_path(@article), form: { data: { turbo_frame: "modal" } } %>
  </article>
<% end %>

<%= turbo_frame_tag "modal" %>
```

---

## Заключение

Современный фронтенд в Rails:
- **Интеграция с любыми инструментами**
- **Hotwire для интерактивности**
- **Компонентный подход**
- **Оптимизация производительности**

Ресурсы:
- [Hotwire Handbook](https://hotwired.dev/)
- [Import Maps Guide](https://github.com/rails/importmap-rails)
- [ViewComponent Docs](https://viewcomponent.org/)

**Домашнее задание:**
1. Создайте Turbo Frame форму
2. Реализуйте live-поиск через Stimulus
3. Настройте Tailwind CSS
4. Создайте React-компонент

[Документация Rails по фронтенду](https://guides.rubyonrails.org/working_with_javascript_in_rails.html)