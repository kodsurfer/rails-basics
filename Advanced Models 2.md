# Лекция: Продвинутые модели в Ruby on Rails (часть 2)

## 1. Полиморфные связи (Polymorphic Associations)
**Что это?** Ассоциация, где модель может принадлежать разным типам моделей через единый интерфейс.

**Пример:** Комментарии для статей и видео:
```ruby
# Модель Comment
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

# Модели, которые могут иметь комментарии
class Article < ApplicationRecord
  has_many :comments, as: :commentable
end

class Video < ApplicationRecord
  has_many :comments, as: :commentable
end
```

**Миграция для полиморфной связи:**
```ruby
create_table :comments do |t|
  t.text :content
  t.references :commentable, polymorphic: true
  t.timestamps
end
```
*Результат:* В таблице автоматически создадутся колонки `commentable_type` (класс) и `commentable_id`.

**Использование:**
```ruby
article = Article.first
comment = article.comments.create(content: "Отличная статья!")
comment.commentable # => возвращает Article объект
```

---

## 2. Редирект "Назад"
**Проблема:** `redirect_to :back` устарел в Rails 5+ из-за риска ошибки при отсутствии HTTP_REFERER.

**Решение:** Используйте `redirect_back` с фолбэком:
```ruby
redirect_back(fallback_location: root_path)
```

**Пример обработки ошибки:**
```ruby
def update
  # ...
rescue ActiveRecord::RecordInvalid
  redirect_back(fallback_location: edit_post_path(@post), alert: "Ошибка сохранения!")
end
```

---

## 3. Вложенные формы (Nested Forms)
**Для чего?** Создание/редактирование связанных моделей через одну форму.

**Шаги реализации:**
1. Добавьте `accepts_nested_attributes_for` в родительскую модель:
```ruby
class Post < ApplicationRecord
  has_many :comments
  accepts_nested_attributes_for :comments, allow_destroy: true
end
```

2. Разрешите параметры в контроллере:
```ruby
def post_params
  params.require(:post).permit(:title, comments_attributes: [:id, :content, :_destroy])
end
```

3. Используйте `fields_for` в форме:
```erb
<%= form_with model: @post do |f| %>
  <%= f.text_field :title %>
  
  <%= f.fields_for :comments do |comment_form| %>
    <%= comment_form.text_area :content %>
    <% if comment_form.object.persisted? %>
      <%= comment_form.check_box :_destroy %>
      Удалить комментарий
    <% end %>
  <% end %>
  
  <%= f.submit %>
<% end %>
```

---

## 4. Кастомизация Devise Views
**Шаги:**
1. Сгенерируйте вьюхи Devise:
```bash
rails generate devise:views
```

2. Редактируйте шаблоны в папке `app/views/devise`:
```erb
# app/views/devise/registrations/new.html.erb
<h2>Регистрация</h2>
<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= f.email_field :email, placeholder: "Ваш email" %>
  ...
<% end %>
```

**Важно:** Для глубокой кастомизации можно наследоваться от Devise контроллеров.

---

## 5. Создание категорий и тегов
**Способ 1:** Ручная реализация через связи "многие-ко-многим":
```ruby
class Post < ApplicationRecord
  has_many :post_tags
  has_many :tags, through: :post_tags
end

class Tag < ApplicationRecord
  has_many :post_tags
  has_many :posts, through: :post_tags
end
```

**Способ 2:** Гем `acts-as-taggable-on`
1. Установка:
```ruby
# Gemfile
gem 'acts-as-taggable-on'
```
```bash
bundle install
rails acts_as_taggable_on_engine:install:migrations
rails db:migrate
```

2. Использование:
```ruby
class Post < ApplicationRecord
  acts_as_taggable_on :categories, :keywords
end

# Добавление тегов
post = Post.new(title: "Rails Guide")
post.tag_list = "ruby, webdev" 
post.save

# Поиск по тегам
Post.tagged_with("ruby")
```

---

## 6. Best Practices
1. **Полиморфные связи:** Используйте, когда логика одинакова для разных типов объектов.
2. **Nested Forms:** Добавляйте `reject_if: :all_blank` для игнорирования пустых полей:
```ruby
accepts_nested_attributes_for :comments, reject_if: ->(attr){ attr[:content].blank? }
```
3. **Теги:** Для сложных систем (например, с статистикой) используйте специализированные гемы.

---

## Практические задания
1. Реализуйте систему оценок (Rating) для статей и видео через полиморфную связь.
2. Создайте форму создания поста с 3 комментариями "по умолчанию".
3. Добавьте тегирование к модели проекта через `acts-as-taggable-on`.

**Ресурсы:**
- [Polymorphic Associations Guide](https://guides.rubyonrails.org/association_basics.html#polymorphic-associations)
- [Nested Forms Documentation](https://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html)
- [ActsAsTaggableOn GitHub](https://github.com/mbleigh/acts-as-taggable-on)
