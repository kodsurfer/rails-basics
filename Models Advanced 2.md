## Продвинутые возможности Ruby on Rails: Модели, Полиморфные связи, Вложенные формы и другие фишки

В этой лекции мы рассмотрим несколько продвинутых тем, связанных с разработкой на Ruby on Rails. Мы поговорим о полиморфных связях, редиректе назад, вложенных формах, кастомизации интерфейса Devise и создании категорий и тегов.

### 1. Полиморфные связи

**Полиморфные связи** (polymorphic associations) позволяют одной модели быть связанной с несколькими другими моделями через одну ассоциацию. 

**Пример:**

Предположим, у нас есть модели `Comment`, `Post` и `Image`. Мы хотим, чтобы комментарии могли быть оставлены к постам и изображениям. 

**Решение:**

1. Добавьте в модель `Comment` два поля: `commentable_id` и `commentable_type`.
2. Определите полиморфную ассоциацию в модели `Comment`:

```ruby
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
```

3. Определите обратную ассоциацию в моделях `Post` и `Image`:

```ruby
class Post < ApplicationRecord
  has_many :comments, as: :commentable
end

class Image < ApplicationRecord
  has_many :comments, as: :commentable
end
```

**Преимущества:**

* Уменьшает дублирование кода.
* Упрощает добавление новых типов комментируемых объектов.

### 2. Редирект назад

**Редирект назад** (redirect back) позволяет перенаправить пользователя на предыдущую страницу после выполнения действия.

**Пример:**

После создания комментария мы хотим перенаправить пользователя на страницу, с которой он пришел.

**Решение:**

Используйте метод `redirect_back`:

```ruby
def create
  @comment = Comment.new(comment_params)
  if @comment.save
    redirect_back(fallback_location: root_path)
  else
    render :new
  end
end
```

**Параметры:**

* `fallback_location`: URL, на который будет перенаправлен пользователь, если предыдущая страница недоступна.

### 3. Вложенные формы (nested forms)

**Вложенные формы** (nested forms) позволяют создавать и редактировать связанные объекты в одной форме.

**Пример:**

Предположим, у нас есть модели `Article` и `Comment`. Мы хотим создавать и редактировать комментарии к статье в одной форме.

**Решение:**

1. Добавьте в модель `Article` ассоциацию `has_many :comments, inverse_of: :article`.
2. В форме для модели `Article` используйте хелпер `fields_for`:

```erb
<%= form_for @article do |f| %>
  <%= f.text_field :title %>
  <%= f.fields_for :comments do |cf| %>
    <%= cf.text_area :body %>
  <% end %>
  <%= f.submit %>
<% end %>
```

3. В контроллере разрешите массив параметров для комментариев:

```ruby
def article_params
  params.require(:article).permit(:title, comments_attributes: [:id, :body, :_destroy])
end
```

**Преимущества:**

* Упрощает создание и редактирование связанных объектов.
* Улучшает пользовательский опыт.

### 4. Включаем кастомные вьюс у Devise

**Devise** - популярный гем для аутентификации в Rails. По умолчанию Devise предоставляет ограниченный набор представлений (views). 

**Кастомизация представлений:**

1. Сгенерируйте представления Devise:

```bash
rails generate devise:views
```

2. Отредактируйте сгенерированные файлы представлений в папке `app/views/devise`.

**Пример:**

Добавьте кастомный заголовок на страницу входа:

```erb
<h1>Добро пожаловать!</h1>
<%= render "devise/shared/links" %>
```

### 5. Создание категорий, тегов

**Категории и теги** - распространенные способы классификации контента.

**Пример:**

Предположим, у нас есть модель `Post`. Мы хотим добавить к постам категории и теги.

**Решение:**

1. Создайте модели `Category` и `Tag`:

```bash
rails generate model Category name:string
rails generate model Tag name:string
```

2. Добавьте ассоциации в модели:

```ruby
class Post < ApplicationRecord
  belongs_to :category
  has_and_belongs_to_many :tags
end

class Category < ApplicationRecord
  has_many :posts
end

class Tag < ApplicationRecord
  has_and_belongs_to_many :posts
end
```

3. Создайте связующую таблицу для связи "многие-ко-многим" между `Post` и `Tag`:

```bash
rails generate migration CreateJoinTablePostsTags posts tags
```

4. Добавьте поля для категории и тегов в форму для модели `Post`:

```erb
<%= form_for @post do |f| %>
  <%= f.collection_select :category_id, Category.all, :id, :name %>
  <%= f.collection_check_boxes :tag_ids, Tag.all, :id, :name %>
  <%= f.submit %>
<% end %>
```

**Преимущества:**

* Позволяет легко классифицировать и искать контент.
* Улучшает структуру данных.

### Заключение

В этой лекции мы рассмотрели несколько продвинутых возможностей Ruby on Rails, которые помогут вам создавать более гибкие и функциональные приложения. Используйте эти инструменты для улучшения вашего кода и повышения эффективности разработки.
