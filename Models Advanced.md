## Продвинутые модели в Ruby on Rails: Ассоциации, Скоупы, STI и другие возможности

В этой лекции мы рассмотрим несколько продвинутых тем, связанных с моделями в Ruby on Rails. Мы поговорим о ассоциациях, включая самосоединения (self-joins), скоупах (scopes), одиночном наследовании таблиц (STI), валидациях и колбэках.

### 1. Ассоциации

Ассоциации в Rails позволяют определить отношения между моделями. Рассмотрим несколько типов ассоциаций:

#### 1.1. Самосоединения (Self-Joins)

**Самосоединения** используются, когда модель связана сама с собой. Например, комментарии могут иметь ответы на другие комментарии.

**Пример:**

```ruby
class Comment < ApplicationRecord
  belongs_to :parent, class_name: 'Comment', optional: true
  has_many :replies, class_name: 'Comment', foreign_key: 'parent_id'
end
```

**Использование:**

```ruby
comment = Comment.find(1)
comment.replies # Возвращает все ответы на этот комментарий
```

#### 1.2. has_one: Профиль пользователя

**has_one** ассоциация используется, когда у модели есть один связанный объект. Например, у пользователя может быть один профиль.

**Пример:**

```ruby
class User < ApplicationRecord
  has_one :profile
end

class Profile < ApplicationRecord
  belongs_to :user
end
```

**Использование:**

```ruby
user = User.find(1)
user.profile # Возвращает профиль пользователя
```

### 2. Скоупы (Scopes)

**Скоупы** позволяют определить часто используемые запросы в модели. Это упрощает код и делает его более читаемым.

**Пример:**

```ruby
class Post < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :recent, -> { order(created_at: :desc) }
end
```

**Использование:**

```ruby
Post.published # Возвращает все опубликованные посты
Post.recent # Возвращает посты, отсортированные по дате создания
```

### 3. Одиночное наследование таблиц (STI)

**Одиночное наследование таблиц** (STI) позволяет использовать одну таблицу для нескольких моделей, которые наследуют общие атрибуты.

**Пример:**

```ruby
class Vehicle < ApplicationRecord
end

class Car < Vehicle
end

class Motorcycle < Vehicle
end
```

**Использование:**

```ruby
car = Car.create(name: 'Toyota')
motorcycle = Motorcycle.create(name: 'Honda')

Vehicle.all # Возвращает все автомобили и мотоциклы
```

### 4. Валидации

**Валидации** используются для проверки данных перед их сохранением в базу данных.

**Пример:**

```ruby
class User < ApplicationRecord
  validates :email, presence: true, uniqueness: true
  validates :age, numericality: { greater_than: 18 }
end
```

**Использование:**

```ruby
user = User.new(email: 'test@example.com', age: 17)
user.valid? # Возвращает false, так как возраст меньше 18
```

### 5. Колбэки (Callbacks)

**Колбэки** позволяют выполнять код до или после определенных событий, таких как создание, обновление или удаление записи.

**Пример:**

```ruby
class User < ApplicationRecord
  before_save :normalize_email

  private

  def normalize_email
    self.email = email.downcase
  end
end
```

**Использование:**

```ruby
user = User.new(email: 'TEST@example.com')
user.save # Email будет сохранен в нижнем регистре
```

### Заключение

В этой лекции мы рассмотрели несколько продвинутых возможностей моделей в Ruby on Rails, которые помогут вам создавать более гибкие и функциональные приложения. Используйте ассоциации, скоупы, STI, валидации и колбэки для улучшения вашего кода и повышения эффективности разработки.
