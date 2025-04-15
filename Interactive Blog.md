## Проект называется InteractiveBlog.

## 1. Установка и настройка

```bash
rails new interactive_blog -j esbuild --css=tailwind
cd interactive_blog
bundle add hotwire-rails react-rails
rails hotwire:install
rails generate react:install
```

## 2. Структура проекта

### Модели
```ruby
# app/models/post.rb
class Post < ApplicationRecord
  has_many :comments, dependent: :destroy
  validates :title, :content, presence: true
end

# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :post
  validates :content, presence: true
end
```

### Контроллеры
```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
    @posts = Post.all
    @posts = @posts.where("title LIKE ?", "%#{params[:q]}%") if params[:q].present?
  end

  def show
    @post = Post.find(params[:id])
    @comment = Comment.new
  end
end

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

## 3. Реализация заданий

### 3.1 Turbo Frame форма (комментарии)
```erb
<!-- app/views/posts/show.html.erb -->
<div class="max-w-2xl mx-auto">
  <h1 class="text-3xl font-bold mb-4"><%= @post.title %></h1>
  
  <div class="mb-8">
    <%= @post.content %>
  </div>

  <h2 class="text-xl font-semibold mb-4">Comments</h2>
  
  <div id="comments" class="mb-6">
    <%= render @post.comments %>
  </div>

  <%= turbo_frame_tag "comment_form" do %>
    <%= render "comments/form", post: @post, comment: @comment %>
  <% end %>

  <%= react_component("Counter", { initialValue: 0 }, { class: "mt-8" }) %>
</div>
```

### 3.2 Live-поиск через Stimulus
```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "results"]
  static values = { url: String }

  search() {
    clearTimeout(this.timeout)
    this.timeout = setTimeout(() => {
      this.performSearch()
    }, 300)
  }

  performSearch() {
    const params = new URLSearchParams({ q: this.inputTarget.value })
    fetch(`${this.urlValue}?${params}`)
      .then(response => response.text())
      .then(html => this.resultsTarget.innerHTML = html)
  }
}
```

### 3.3 Tailwind CSS
```erb
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
<head>
  <title>InteractiveBlog</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <%= csrf_meta_tags %>
  <%= csp_meta_tag %>
  <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
  <%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>
</head>
<body class="bg-gray-50">
  <div class="container mx-auto px-4 py-8">
    <%= yield %>
  </div>
</body>
</html>
```

### 3.4 React-компонент
```javascript
// app/javascript/components/Counter.jsx
import React, { useState } from 'react'

const Counter = ({ initialValue }) => {
  const [count, setCount] = useState(Number(initialValue) || 0)

  return (
    <div className="p-4 border rounded-lg bg-white shadow">
      <h3 className="text-lg font-medium mb-2">React Counter</h3>
      <div className="flex items-center gap-4">
        <button 
          onClick={() => setCount(c => c - 1)}
          className="bg-red-500 hover:bg-red-600 text-white font-bold py-1 px-3 rounded"
        >
          -
        </button>
        <span className="text-xl">{count}</span>
        <button 
          onClick={() => setCount(c => c + 1)}
          className="bg-green-500 hover:bg-green-600 text-white font-bold py-1 px-3 rounded"
        >
          +
        </button>
      </div>
    </div>
  )
}

export default Counter
```

## 4. Запуск проекта

1. Создайте и выполните миграции:
```bash
rails db:create db:migrate
```

2. Добавьте тестовые данные:
```ruby
# db/seeds.rb
10.times do |i|
  post = Post.create!(
    title: "Post #{i + 1}",
    content: "This is content for post #{i + 1}"
  )
  
  rand(1..5).times do |j|
    post.comments.create!(
      content: "Comment #{j + 1} for post #{i + 1}"
    )
  end
end
```
```bash
rails db:seed
```

3. Запустите сервер:
```bash
bin/dev
```

## 5. Проверка функционала

1. **Turbo Frame форма**:
   - Откройте http://localhost:3000/posts/1
   - Добавьте комментарий - страница не перезагружается

2. **Live-поиск**:
   - Откройте http://localhost:3000/posts
   - Введите текст в поле поиска - результаты обновляются по мере ввода

3. **Tailwind CSS**:
   - Проверьте, что все элементы имеют современные стили

4. **React-компонент**:
   - На странице поста должен быть интерактивный счетчик
   - Кнопки "+" и "-" должны изменять значение

Полный код проекта доступен в репозитории: [InteractiveBlog на GitHub](https://github.com/example/interactive_blog) (вымышленная ссылка)
