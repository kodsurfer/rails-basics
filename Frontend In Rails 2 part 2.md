# Продвинутая интеграция JavaScript в Ruby on Rails: Hotwire, Turbo Frames и реальные примеры

## Введение в современный JavaScript в Rails

Современный Rails предлагает мощные инструменты для интеграции JavaScript, позволяя создавать интерактивные приложения без сложных SPA-фреймворков. В этой лекции мы углубимся в практические аспекты работы с:

1. Turbo Frames и Streams для динамических интерфейсов
2. Реализацию реального времени с Hotwire Broadcasts
3. Кастомизацию рендеринга моделей
4. Оптимизацию seed-данных с skip callbacks

## 1. Turbo Frames: углублённое использование

### 1.1 Динамическая загрузка контента

Пример из [StackOverflow](https://stackoverflow.com/questions/67074124/how-to-define-the-way-instance-model-should-be-rendered-in-rails-6):

```erb
<!-- app/views/posts/_post.html.erb -->
<%= turbo_frame_tag dom_id(post) do %>
  <div class="post">
    <%= render partial: "posts/#{post.type.underscore}", locals: { post: post } %>
  </div>
<% end %>
```

Контроллер:
```ruby
def show
  @post = Post.find(params[:id])
  respond_to do |format|
    format.html
    format.turbo_stream do
      render turbo_stream: turbo_stream.replace(@post, partial: "posts/post", locals: { post: @post })
    end
  end
end
```

### 1.2 Вложенные Turbo Frames

```erb
<%= turbo_frame_tag "main_content" do %>
  <%= turbo_frame_tag "comments", src: post_comments_path(@post) %>
<% end %>
```

## 2. Hotwire Broadcasts: реальное время

На основе примера из [Corsego Blog](https://blog.corsego.com/turbo-hotwire-broadcasts):

### 2.1 Настройка модели

```ruby
# app/models/reaction.rb
class Reaction < ApplicationRecord
  belongs_to :message
  belongs_to :user

  after_create_commit -> { broadcast_append_to message }
  after_destroy_commit -> { broadcast_remove_to message }
end
```

### 2.2 Контроллер реакций

Адаптация из [примера GitHub](https://github.com/HSEADC/B21DZ08-L3-Project-03/blob/ADC_Rise/app/controllers/reactions_controller.rb#L26C1-L31C6):

```ruby
# app/controllers/reactions_controller.rb
class ReactionsController < ApplicationController
  def create
    @message = Message.find(params[:message_id])
    @reaction = @message.reactions.new(reaction_params.merge(user: current_user))

    if @reaction.save
      respond_to do |format|
        format.turbo_stream
        format.html { redirect_to @message }
      end
    else
      render :new, status: :unprocessable_entity
    end
  end

  private

  def reaction_params
    params.require(:reaction).permit(:emoji)
  end
end
```

### 2.3 Шаблон Turbo Stream

```erb
<!-- app/views/reactions/create.turbo_stream.erb -->
<%= turbo_stream.replace "reaction_form" do %>
  <%= render "form", message: @message, reaction: Reaction.new %>
<% end %>

<%= turbo_stream.append "reactions" do %>
  <%= render @reaction %>
<% end %>
```

## 3. Кастомизация рендеринга моделей

### 3.1 Полиморфный рендеринг

Решение из [StackOverflow](https://stackoverflow.com/questions/67074124/how-to-define-the-way-instance-model-should-be-rendered-in-rails-6):

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  def to_partial_path
    "posts/#{self.class.name.underscore}"
  end
end

# app/views/posts/_text_post.html.erb
<div class="text-post">
  <h3><%= post.title %></h3>
  <p><%= post.content %></p>
</div>

# app/views/posts/_video_post.html.erb
<div class="video-post">
  <h3><%= post.title %></h3>
  <iframe src="<%= post.video_url %>"></iframe>
</div>
```

### 3.2 Декораторы для сложного рендеринга

```ruby
# app/decorators/post_decorator.rb
class PostDecorator
  def initialize(post)
    @post = post
  end

  def render_card
    if @post.video?
      render_video_card
    else
      render_text_card
    end
  end

  private

  def render_video_card
    content_tag(:div, class: 'video-card') do
      # ...
    end
  end
end
```

## 4. Оптимизация seed-данных

На основе [StackOverflow](https://stackoverflow.com/questions/28503327/rails-seeds-rb-how-can-i-skip-multiple-callbacks-using-class-skip-callback):

### 4.1 Временное отключение колбэков

```ruby
# db/seeds.rb
ActiveRecord::Base.transaction do
  # Отключаем колбэки
  User.skip_callback(:create, :after, :send_welcome_email)
  Post.skip_callback(:save, :before, :update_slug)

  # Создаём тестовые данные
  100.times do |i|
    user = User.create!(name: "User #{i}", email: "user#{i}@example.com")
    
    5.times do |j|
      Post.create!(
        title: "Post #{j} by User #{i}",
        content: "Lorem ipsum...",
        user: user
      )
    end
  end

  # Включаем колбэки обратно
  User.set_callback(:create, :after, :send_welcome_email)
  Post.set_callback(:save, :before, :update_slug)
end
```

### 4.2 Альтернативный подход с disable_all_callbacks

```ruby
# db/seeds.rb
def without_callbacks(*models)
  models.each do |model|
    model.reset_callbacks(:save)
    model.reset_callbacks(:create)
    model.reset_callbacks(:update)
  end

  yield

  models.each { |model| model.set_callbacks }
end

without_callbacks(User, Post) do
  # Массовое создание данных
end
```

## 5. Продвинутые паттерны с Hotwire

### 5.1 Интерактивные формы с валидацией

```erb
<!-- app/views/posts/_form.html.erb -->
<%= form_with(model: post, data: { controller: "form", action: "turbo:submit-start->form#submitStart" }) do |f| %>
  <%= f.text_field :title, data: { form_target: "input" } %>
  <%= f.submit "Save", data: { form_target: "submit" } %>
<% end %>
```

```javascript
// app/javascript/controllers/form_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "submit"]

  submitStart() {
    this.submitTarget.disabled = true
    this.submitTarget.value = "Saving..."
  }
}
```

### 5.2 Lazy loading с Turbo Frames

```erb
<%= turbo_frame_tag "lazy_comments", src: post_comments_path(@post), loading: :lazy do %>
  <div class="loading-spinner">Loading comments...</div>
<% end %>
```

## 6. Оптимизация производительности

### 6.1 Частичная загрузка данных

```ruby
# app/controllers/posts_controller.rb
def index
  @posts = Post.all
  @posts = @posts.limit(10) if turbo_frame_request?
end
```

### 6.2 Кэширование Turbo Streams

```ruby
# app/controllers/comments_controller.rb
def index
  @comments = @post.comments
  fresh_when(@comments)
end
```

## 7. Тестирование JavaScript в Rails

### 7.1 Системные тесты с Turbo

```ruby
# test/system/posts_test.rb
test "creating post updates list" do
  visit posts_path
  
  click_link "New Post"
  fill_in "Title", with: "New Post"
  click_button "Create Post"

  assert_selector "turbo-frame#posts", text: "New Post"
end
```

### 7.2 Тестирование Stimulus контроллеров

```javascript
// test/javascript/controllers/form_controller.test.js
import { Application } from "@hotwired/stimulus"
import FormController from "controllers/form_controller"

describe("FormController", () => {
  beforeEach(() => {
    document.body.innerHTML = `
      <form data-controller="form">
        <input data-form-target="input">
        <button data-form-target="submit">Submit</button>
      </form>
    `
    
    Application.start().register("form", FormController)
  })

  test("disables submit on start", () => {
    const submit = document.querySelector("[data-form-target='submit']")
    const event = new Event("turbo:submit-start")
    document.querySelector("form").dispatchEvent(event)
    
    assert.equal(submit.disabled, true)
    assert.equal(submit.value, "Saving...")
  })
})
```

## Заключение и лучшие практики

1. **Используйте Turbo Frames** для создания SPA-like интерфейсов
2. **Hotwire Broadcasts** идеальны для real-time обновлений
3. **Кастомизируйте рендеринг** через partials и декораторы
4. **Оптимизируйте seed-данные** временным отключением колбэков
5. **Тестируйте** все сценарии взаимодействия

### Дополнительные ресурсы:
- [Официальная документация Hotwire](https://hotwired.dev/)
- [Turbo Handbook](https://turbo.hotwired.dev/handbook/)
- [Stimulus Components](https://stimulus-components.netlify.app/)

### Практическое задание:
1. Реализуйте систему комментариев с real-time обновлениями
2. Создайте полиморфную ленту с разными типами постов
3. Напишите seed-файл с оптимизацией колбэков
4. Протестируйте все сценарии с системными тестами
