# Лекция: Продвинутые модели в Ruby on Rails

## 1. Self-Join Associations
**Что это?** Ассоциация, где модель ссылается сама на себя.  
**Пример:** Древовидные структуры (комментарии с ответами, сотрудники и менеджеры).

```ruby
class Comment < ApplicationRecord
  has_many :replies, class_name: "Comment", foreign_key: "parent_id"
  belongs_to :parent, class_name: "Comment", optional: true
end
```

```ruby
# Миграция:
add_column :comments, :parent_id, :integer
add_index :comments, :parent_id
```

**Использование:**
```ruby
parent_comment = Comment.create(content: "Основной комментарий")
reply = parent_comment.replies.create(content: "Ответ")
```

---

## 2. Single Table Inheritance (STI)
**Что это?** Наследование моделей с хранением в одной таблице.  
**Пример:** Разные типы пользователей (User, Admin, Moderator).

```ruby
# Таблица users:
#   id | type     | name | role
# ---------------------------------
class User < ApplicationRecord; end
class Admin < User; end
class Moderator < User; end
```

**Особенности:**
- Автоматическое заполнение столбца `type`.
- Общие поля в базе, уникальные методы в классах-наследниках.

---

## 3. Scopes
**Что это?** Переиспользуемые запросы, объявленные в модели.

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :recent, -> { order(created_at: :desc).limit(5) }
end
```

**Цепочка вызовов:**
```ruby
Article.published.recent
```

**Метод класса как scope:**
```ruby
def self.by_author(author)
  where(author: author)
end
```

---

## 4. Валидации (Validations)
**Примеры:**
```ruby
class User < ApplicationRecord
  validates :email, 
    presence: true,
    uniqueness: { case_sensitive: false },
    format: { with: URI::MailTo::EMAIL_REGEXP }

  validate :password_complexity

  private

  def password_complexity
    return if password.match?(/^(?=.*[a-z])(?=.*[A-Z])/)
    errors.add :password, "должен содержать заглавные и строчные буквы"
  end
end
```

---

## 5. Колбэки (Callbacks)
**Жизненный цикл объекта:**
- `before_validation`
- `before_save`
- `after_create`
- И др.

**Пример:** Извлечение YouTube ID из URL перед сохранением.
```ruby
class Video < ApplicationRecord
  before_save :extract_youtube_id

  private

  def extract_youtube_id
    self.youtube_id = if url.include?("youtube.com/watch?v=")
                        url.split("v=").last[0..10]
                      else
                        url.split("/").last
                      end
  end
end
```

---

## 6. Ассоциация `has_one`
**Пример:** Профиль пользователя.
```ruby
class User < ApplicationRecord
  has_one :profile
end

class Profile < ApplicationRecord
  belongs_to :user
end
```

**Миграция для профиля:**
```ruby
create_table :profiles do |t|
  t.text :bio
  t.references :user, foreign_key: true
end
```

---

## 7. Советы и Best Practices
1. **STI:** Используйте, когда наследники имеют общие атрибуты. Для сильно отличающихся данных рассмотрите отдельные таблицы.
2. **Скоупы:** Держите запросы в моделях, избегайте RAW SQL.
3. **Колбэки:** Не злоупотребляйте – сложная логика в колбэках усложняет тестирование.
4. **Self-Join:** Добавляйте индексы на `parent_id` для оптимизации запросов.

---

## Задания для практики
1. Реализуйте модель `Category` с возможностью вложенности (self-join).
2. Создайте STI для модели `Vehicle` с наследниками `Car` и `Bicycle`.
3. Добавьте валидацию для YouTube URL, проверяющую корректность формата.

**Ресурсы:**
- [Active Record Associations](https://guides.rubyonrails.org/association_basics.html)
- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html)
- [Active Record Validations](https://guides.rubyonrails.org/active_record_validations.html)
