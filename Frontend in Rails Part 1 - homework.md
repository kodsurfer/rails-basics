# Домашнее задание: Интеграция JavaScript в Rails

## 1. Turbo Frame форма

**Задача:** Создать форму для комментариев, которая обновляется без перезагрузки страницы

### Реализация:

1. Добавьте в модель Comment:
```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :post
  validates :content, presence: true
end
```

2. Создайте контроллер:
```ruby
# app/controllers/comments_controller.rb
class CommentsController < ApplicationController
  def create
    @post = Post.find(params[:post_id])
    @comment = @post.comments.build(comment_params)
    
    if @comment.save
      respond_to do |format|
        format.turbo_stream
        format.html { redirect_to @post }
      end
    else
      render :new
    end
  end

  private
  
  def comment_params
    params.require(:comment).permit(:content)
  end
end
```

3. Создайте шаблон формы:
```erb
<%# app/views/comments/_form.html.erb %>
<%= turbo_frame_tag "comment_form" do %>
  <%= form_with(model: [post, Comment.new], data: { turbo_frame: "comments" }) do |f| %>
    <%= f.text_area :content %>
    <%= f.submit "Post Comment" %>
  <% end %>
<% end %>
```

4. Добавьте шаблон для Turbo Stream:
```erb
<%# app/views/comments/create.turbo_stream.erb %>
<%= turbo_stream.append "comments" do %>
  <%= render @comment %>
<% end %>

<%= turbo_stream.replace "comment_form" do %>
  <%= render "form", post: @post %>
<% end %>
```

## 2. Live-поиск через Stimulus

**Задача:** Реализовать мгновенный поиск по статьям

### Реализация:

1. Создайте Stimulus контроллер:
```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "results"]
  static values = { url: String }

  async search() {
    const params = new URLSearchParams({
      q: this.inputTarget.value
    })
    
    const response = await fetch(`${this.urlValue}?${params}`)
    this.resultsTarget.innerHTML = await response.text()
  }

  debounce(fn, delay = 300) {
    let timeout
    return (...args) => {
      clearTimeout(timeout)
      timeout = setTimeout(() => fn.apply(this, args), delay)
    }
  }

  connect() {
    this.search = this.debounce(this.search)
  }
}
```

2. Добавьте вьюху для поиска:
```erb
<%# app/views/posts/_search.html.erb %>
<div data-controller="search" data-search-url-value="<%= search_posts_path %>">
  <input type="text" data-search-target="input" data-action="input->search#search">
  <div data-search-target="results">
    <%= render @posts || [] %>
  </div>
</div>
```

3. Настройте маршрут:
```ruby
# config/routes.rb
resources :posts do
  collection do
    get :search
  end
end
```

4. Создайте экшен поиска:
```ruby
# app/controllers/posts_controller.rb
def search
  @posts = Post.where("title LIKE ?", "%#{params[:q]}%")
  respond_to do |format|
    format.turbo_stream do
      render turbo_stream: turbo_stream.update("search_results", partial: "posts/results")
    end
    format.html
  end
end
```

## 3. Настройка Tailwind CSS

**Задача:** Интегрировать Tailwind CSS в Rails приложение

### Реализация:

1. Установите Tailwind:
```bash
rails css:install:tailwind
```

2. Настройте конфигурацию:
```javascript
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  }
}
```

3. Добавьте базовые стили:
```css
/* app/assets/stylesheets/application.tailwind.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

4. Пример использования в форме:
```erb
<%= form_with model: @post, class: "max-w-md mx-auto mt-8" do |f| %>
  <div class="mb-4">
    <%= f.label :title, class: "block text-gray-700 text-sm font-bold mb-2" %>
    <%= f.text_field :title, class: "shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" %>
  </div>
<% end %>
```

## 4. React-компонент

**Задача:** Создать и интегрировать React-компонент счетчика

### Реализация:

1. Установите необходимые гемы:
```ruby
# Gemfile
gem 'jsbundling-rails'
gem 'react-rails'
```

2. Установите зависимости:
```bash
bundle install
rails javascript:install:esbuild
rails generate react:install
```

3. Создайте компонент:
```javascript
// app/javascript/components/Counter.jsx
import React, { useState } from 'react'

const Counter = ({ initialValue = 0 }) => {
  const [count, setCount] = useState(initialValue)

  return (
    <div className="p-4 border rounded-lg">
      <p className="text-lg">Count: {count}</p>
      <button 
        onClick={() => setCount(count + 1)}
        className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
      >
        Increment
      </button>
    </div>
  )
}

export default Counter
```

4. Интегрируйте в Rails-вью:
```erb
<%# app/views/posts/show.html.erb %>
<%= react_component("Counter", { initialValue: 10 }, { class: "my-4" }) %>
```

5. Обновите конфигурацию:
```javascript
// app/javascript/application.js
import React from 'react'
import ReactDOM from 'react-dom'
import Counter from '../components/Counter'

// Глобальная регистрация для react-rails
global.Counter = Counter
```

## Проверка выполнения

Для проверки убедитесь, что:
1. Форма комментариев работает без перезагрузки страницы
2. Поиск мгновенно отображает результаты при вводе
3. Стили Tailwind применяются корректно
4. React-счетчик работает и сохраняет состояние

## Дополнительные материалы

1. [Официальная документация Turbo](https://turbo.hotwired.dev/)
2. [Stimulus Handbook](https://stimulus.hotwired.dev/handbook/introduction)
3. [Tailwind CSS Docs](https://tailwindcss.com/docs/installation)
4. [React-Rails Guide](https://github.com/reactjs/react-rails)
