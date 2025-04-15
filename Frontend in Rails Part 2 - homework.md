## 1. Интерактивная форма с валидацией через Turbo Streams

### 1.1 Модель Post с валидациями
```ruby
# app/models/post.rb
class Post < ApplicationRecord
  validates :title, presence: true, length: { minimum: 5 }
  validates :content, presence: true, length: { minimum: 10 }
end
```

### 1.2 Обновленный PostsController
```ruby
# app/controllers/posts_controller.rb
def create
  @post = Post.new(post_params)

  respond_to do |format|
    if @post.save
      format.html { redirect_to @post, notice: 'Post was successfully created.' }
      format.turbo_stream
    else
      format.html { render :new }
      format.turbo_stream do
        render turbo_stream: turbo_stream.replace(
          'post_form',
          partial: 'posts/form',
          locals: { post: @post }
        )
      end
    end
  end
end
```

### 1.3 Шаблон формы с валидацией
```erb
<%# app/views/posts/_form.html.erb %>
<%= form_with(model: post, id: 'post_form') do |f| %>
  <% if post.errors.any? %>
    <div id="error_explanation" class="bg-red-50 text-red-500 px-3 py-2 font-medium rounded-lg mt-3">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>
      <ul>
        <% post.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="my-5">
    <%= f.label :title %>
    <%= f.text_field :title, class: "block shadow rounded-md border border-gray-200 outline-none px-3 py-2 mt-2 w-full" %>
  </div>

  <div class="my-5">
    <%= f.label :content %>
    <%= f.text_area :content, rows: 4, class: "block shadow rounded-md border border-gray-200 outline-none px-3 py-2 mt-2 w-full" %>
  </div>

  <%= f.submit class: "rounded-lg py-3 px-5 bg-blue-600 text-white inline-block font-medium cursor-pointer" %>
<% end %>
```

## 2. Компонент дашборда с графиками на Stimulus

### 2.1 Установка Chart.js
```bash
yarn add chart.js
```

### 2.2 Stimulus контроллер для графиков
```javascript
// app/javascript/controllers/dashboard_controller.js
import { Controller } from "@hotwired/stimulus"
import Chart from 'chart.js/auto'

export default class extends Controller {
  static values = { data: Object }

  connect() {
    this.renderCharts()
  }

  renderCharts() {
    // График 1: Посещения
    new Chart(
      this.element.querySelector('#visitsChart'),
      {
        type: 'line',
        data: {
          labels: this.dataValue.dates,
          datasets: [{
            label: 'Посещения',
            data: this.dataValue.visits,
            borderColor: 'rgb(59, 130, 246)',
            tension: 0.1
          }]
        }
      }
    )

    // График 2: Комментарии
    new Chart(
      this.element.querySelector('#commentsChart'),
      {
        type: 'bar',
        data: {
          labels: this.dataValue.post_titles,
          datasets: [{
            label: 'Комментарии',
            data: this.dataValue.comments_count,
            backgroundColor: 'rgb(16, 185, 129)'
          }]
        }
      }
    )
  }
}
```

### 2.3 Шаблон дашборда
```erb
<%# app/views/dashboards/show.html.erb %>
<div data-controller="dashboard" 
     data-dashboard-data-value="<%= @chart_data.to_json %>">
  <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
    <div class="bg-white p-6 rounded-lg shadow">
      <h3 class="text-lg font-medium mb-4">Посещения за последние 7 дней</h3>
      <canvas id="visitsChart" height="300"></canvas>
    </div>
    
    <div class="bg-white p-6 rounded-lg shadow">
      <h3 class="text-lg font-medium mb-4">Комментарии по постам</h3>
      <canvas id="commentsChart" height="300"></canvas>
    </div>
  </div>
</div>
```

### 2.4 Контроллер дашборда
```ruby
# app/controllers/dashboards_controller.rb
class DashboardsController < ApplicationController
  def show
    @chart_data = {
      dates: 7.days.ago.to_date..Date.today,
      visits: rand(100..200).downto(1).to_a,
      post_titles: Post.limit(5).pluck(:title),
      comments_count: Post.joins(:comments)
                         .group(:id)
                         .limit(5)
                         .order('COUNT(comments.id) DESC')
                         .count('comments.id')
                         .values
    }
  end
end
```

## 3. Ленивая загрузка для тяжелых компонентов

### 3.1 LazyLoad Stimulus контроллер
```javascript
// app/javascript/controllers/lazy_load_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { url: String }

  connect() {
    if ('IntersectionObserver' in window) {
      this.observer = new IntersectionObserver(
        (entries) => this.handleIntersection(entries),
        { threshold: 0.1 }
      )
      this.observer.observe(this.element)
    } else {
      this.loadContent()
    }
  }

  disconnect() {
    this.observer?.unobserve(this.element)
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadContent()
        this.observer.unobserve(this.element)
      }
    })
  }

  async loadContent() {
    const response = await fetch(this.urlValue)
    this.element.innerHTML = await response.text()
  }
}
```

### 3.2 Пример использования
```erb
<%# app/views/posts/show.html.erb %>
<div data-controller="lazy-load" 
     data-lazy-load-url-value="<%= lazy_comments_post_path(@post) %>">
  <div class="flex justify-center py-8">
    <div class="animate-pulse">Загрузка комментариев...</div>
  </div>
</div>
```

### 3.3 Контроллер для ленивой загрузки
```ruby
# app/controllers/posts_controller.rb
def lazy_comments
  @post = Post.find(params[:id])
  render partial: 'posts/comments', locals: { comments: @post.comments }
end
```

## 4. Системные тесты

### 4.1 Тест формы с валидацией
```ruby
# test/system/posts_test.rb
require "application_system_test_case"

class PostsTest < ApplicationSystemTestCase
  test "form validation with turbo streams" do
    visit new_post_path
    
    # Попытка сохранить с пустыми полями
    click_button "Create Post"
    
    # Проверяем отображение ошибок
    assert_selector "#error_explanation"
    assert_text "Title can't be blank"
    assert_text "Content can't be blank"
    
    # Заполняем форму правильно
    fill_in "Title", with: "Valid title"
    fill_in "Content", with: "Valid content that is long enough"
    click_button "Create Post"
    
    # Проверяем успешное создание
    assert_text "Post was successfully created"
  end
end
```

### 4.2 Тест дашборда
```ruby
# test/system/dashboards_test.rb
require "application_system_test_case"

class DashboardsTest < ApplicationSystemTestCase
  test "viewing dashboard with charts" do
    # Создаем тестовые данные
    3.times do |i|
      post = Post.create!(title: "Post #{i}", content: "Content #{i}")
      (i+1).times { post.comments.create!(content: "Comment") }
    end

    visit dashboard_path
    
    # Проверяем наличие графиков
    assert_selector "#visitsChart"
    assert_selector "#commentsChart"
    
    # Проверяем данные
    assert_text "Посещения за последние 7 дней"
    assert_text "Комментарии по постам"
  end
end
```

### 4.3 Тест ленивой загрузки
```ruby
# test/system/comments_test.rb
require "application_system_test_case"

class CommentsTest < ApplicationSystemTestCase
  test "lazy loading comments" do
    post = Post.create!(title: "Test", content: "Content")
    5.times { post.comments.create!(content: "Comment") }

    visit post_path(post)
    
    # Проверяем состояние загрузки
    assert_text "Загрузка комментариев..."
    
    # Ждем загрузки (в реальном тесте можно использовать Capybara.wait)
    sleep 1
    
    # Проверяем загруженные комментарии
    assert_selector ".comment", count: 5
  end
end
```

## 5. Дополнительные настройки

### 5.1 Конфигурация Capybara для System Tests
```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
  
  # Добавляем ожидание для асинхронных операций
  def wait_for_turbo(timeout = 2)
    sleep timeout
  end
end
```

### 5.2 Добавление тестовых данных
```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  include FactoryBot::Syntax::Methods
end
```

## Как запустить проект

1. Установите зависимости:
```bash
bundle install
yarn install
```

2. Создайте и наполните БД:
```bash
rails db:create db:migrate db:seed
```

3. Запустите сервер:
```bash
bin/dev
```

4. Запустите тесты:
```bash
rails test:system
```

Проект демонстрирует:
- Интерактивные формы с валидацией через Turbo Streams
- Динамические графики на Stimulus + Chart.js
- Оптимизацию производительности через ленивую загрузку
- Полное покрытие системными тестами

Для дальнейшего развития можно добавить:
1. Аутентификацию пользователей
2. Реальные данные для графиков
3. Более сложные сценарии ленивой загрузки
4. Дополнительные интерактивные элементы
