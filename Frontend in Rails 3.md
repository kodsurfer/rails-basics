# Фронтенд в Rails, часть 3: Работа с изображениями и сортировка элементов

## Введение

В этой лекции мы рассмотрим два важных аспекта фронтенд-разработки в Rails:

1. **Оптимизация работы с изображениями** с помощью FastImage
2. **Сортировка элементов интерфейса** с использованием acts_as_list

## 1. Оптимизация работы с изображениями (FastImage)

### 1.1 Установка и настройка FastImage

Добавьте в Gemfile:
```ruby
gem 'fastimage'
```

Установите гем:
```bash
bundle install
```

### 1.2 Основные возможности FastImage

#### Определение размеров изображения
```ruby
require 'fastimage'

# Локальный файл
FastImage.size('public/images/rails.png')
#=> [300, 200] (width, height)

# Удаленный URL
FastImage.size('https://example.com/image.jpg')
#=> [800, 600]
```

#### Определение типа изображения
```ruby
FastImage.type('public/images/rails.png')
#=> :png
```

#### Пример использования в модели
```ruby
# app/models/post.rb
class Post < ApplicationRecord
  def image_dimensions
    return unless image.attached?
    
    tempfile = image.download
    dimensions = FastImage.size(tempfile.path)
    tempfile.close
    dimensions
  rescue
    nil
  end
end
```

### 1.3 Практический пример: адаптивные изображения

```erb
<%# app/views/posts/_post.html.erb %>
<% if post.image.attached? %>
  <% width, height = post.image_dimensions %>
  <% aspect_ratio = width.to_f / height.to_f %>
  
  <div class="image-container" style="padding-bottom: <%= 100 / aspect_ratio %>%;">
    <%= image_tag post.image, 
                  class: "lazyload", 
                  data: { src: url_for(post.image) },
                  alt: post.title %>
  </div>
<% end %>
```

Стили для адаптивного контейнера:
```css
.image-container {
  position: relative;
  width: 100%;
  overflow: hidden;
}

.image-container img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

## 2. Сортировка элементов (acts_as_list)

### 2.1 Установка и настройка acts_as_list

Добавьте в Gemfile:
```ruby
gem 'acts_as_list'
```

Установите гем:
```bash
bundle install
```

### 2.2 Добавление сортировки к модели

```ruby
# app/models/menu_item.rb
class MenuItem < ApplicationRecord
  acts_as_list scope: :menu_id
  
  belongs_to :menu
  validates :name, presence: true
end
```

Миграция для добавления поля position:
```bash
rails g migration AddPositionToMenuItems position:integer
rails db:migrate
```

### 2.3 Реализация сортировки в интерфейсе

#### Контроллер:
```ruby
# app/controllers/menu_items_controller.rb
class MenuItemsController < ApplicationController
  def update_order
    ActiveRecord::Base.transaction do
      params[:order].each_with_index do |id, index|
        MenuItem.where(id: id).update_all(position: index + 1)
      end
    end
    
    head :ok
  end
end
```

#### JavaScript (Stimulus Controller):
```javascript
// app/javascript/controllers/sortable_controller.js
import { Controller } from "@hotwired/stimulus"
import Sortable from 'sortablejs'

export default class extends Controller {
  connect() {
    this.sortable = Sortable.create(this.element, {
      animation: 150,
      handle: '.handle',
      onEnd: this.updateOrder.bind(this)
    })
  }

  async updateOrder() {
    const items = Array.from(this.element.children)
    const order = items.map(item => item.dataset.id)
    
    await fetch(this.data.get('url'), {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': document.querySelector("[name='csrf-token']").content
      },
      body: JSON.stringify({ order: order })
    })
  }
}
```

#### Шаблон:
```erb
<%# app/views/menu_items/index.html.erb %>
<ul data-controller="sortable" 
    data-sortable-url="<%= update_order_menu_items_path %>">
  <% @menu_items.each do |item| %>
    <li data-id="<%= item.id %>" class="flex items-center p-2 border mb-2 bg-white">
      <span class="handle mr-2 cursor-move">≡</span>
      <%= item.name %>
    </li>
  <% end %>
</ul>
```

#### Маршруты:
```ruby
# config/routes.rb
resources :menu_items do
  patch :update_order, on: :collection
end
```

### 2.4 Дополнительные методы acts_as_list

```ruby
# Переместить элемент выше
item.move_higher

# Переместить элемент ниже
item.move_lower

# Переместить на конкретную позицию
item.insert_at(3)

# Получить элементы в порядке позиции
Menu.items.ordered
```

## 3. Комбинированный пример: Сортируемая галерея изображений

### 3.1 Модель GalleryImage
```ruby
# app/models/gallery_image.rb
class GalleryImage < ApplicationRecord
  acts_as_list
  has_one_attached :image
  
  validates :image, attached: true
  
  def dimensions
    return unless image.attached?
    
    tempfile = image.download
    dimensions = FastImage.size(tempfile.path)
    tempfile.close
    dimensions
  end
end
```

### 3.2 Контроллер
```ruby
# app/controllers/gallery_images_controller.rb
class GalleryImagesController < ApplicationController
  def index
    @gallery_images = GalleryImage.ordered
  end

  def update_order
    ActiveRecord::Base.transaction do
      params[:order].each_with_index do |id, index|
        GalleryImage.where(id: id).update_all(position: index + 1)
      end
    end
    
    head :ok
  end
end
```

### 3.3 Шаблон
```erb
<%# app/views/gallery_images/index.html.erb %>
<div class="grid grid-cols-1 md:grid-cols-3 gap-4"
     data-controller="sortable"
     data-sortable-url="<%= update_order_gallery_images_path %>">
  <% @gallery_images.each do |image| %>
    <div data-id="<%= image.id %>" 
         class="relative group rounded-lg overflow-hidden shadow-lg">
      <%= image_tag image.image.variant(resize_to_limit: [800, 800]), 
                   class: "w-full h-auto" %>
      
      <div class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity">
        <span class="handle text-white text-4xl cursor-move">≡</span>
      </div>
    </div>
  <% end %>
</div>
```

## Заключение

Мы рассмотрели два мощных инструмента для фронтенд-разработки в Rails:

1. **FastImage** - для эффективной работы с изображениями:
   - Определение размеров и типа изображений
   - Оптимизация загрузки и отображения
   - Создание адаптивных контейнеров

2. **acts_as_list** - для реализации сортировки:
   - Drag-and-drop интерфейс
   - Сохранение порядка на сервере
   - Гибкие методы управления порядком

### Домашнее задание

1. Реализуйте адаптивную галерею изображений с lazy loading
2. Добавьте возможность сортировки элементов через drag-and-drop
3. Оптимизируйте загрузку изображений с предварительным расчетом размеров
4. Напишите системные тесты для проверки функционала

### Полезные ссылки

- [Документация FastImage](https://github.com/sdsykes/fastimage)
- [Документация acts_as_list](https://github.com/brendon/acts_as_list)
- [SortableJS](https://sortablejs.github.io/Sortable/)
