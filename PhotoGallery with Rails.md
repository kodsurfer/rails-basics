# **Реализация фото-галереи в Rails**  

## **1. Введение**  
Фото-галереи в виде карусели (слайдера) — популярный элемент интерфейса. В 2025 году актуальны:  
- **Быстрая загрузка** (ленивая подгрузка, оптимизация изображений).  
- **Адаптивность** (поддержка мобильных устройств).  
- **Доступность** (ARIA-атрибуты, клавиатурная навигация).  
- **Современные библиотеки** (Stimulus, Hotwire, React при необходимости).  

Сегодня реализуем галерею на **Rails 7+** с **Hotwire (Turbo + Stimulus)** и **lazy-loading**.  

---

## **2. Подготовка проекта**  
### **2.1. Создаем модель `Photo`**  
```bash
rails g model Photo title:string image:attachment
rails db:migrate
```  

### **2.2. Настраиваем Active Storage**  
Убедимся, что `config/storage.yml` использует облачное хранилище (например, **Amazon S3** или **Cloudinary**).  

```yaml
# config/storage.yml
cloudinary:
  service: Cloudinary
```  

```ruby
# config/environments/production.rb
config.active_storage.service = :cloudinary
```  

### **2.3. Добавляем валидации**  
```ruby
# app/models/photo.rb
class Photo < ApplicationRecord
  has_one_attached :image
  validates :title, presence: true
  validates :image, attached: true, content_type: ['image/png', 'image/jpg', 'image/jpeg']
end
```  

---

## **3. Реализация карусели**  
### **3.1. Устанавливаем Stimulus**  
Rails 7+ уже включает Stimulus, но можно обновить:  
```bash
yarn add @hotwired/stimulus
```  

### **3.2. Создаем контроллер Stimulus**  
```javascript
// app/javascript/controllers/carousel_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["slide"]
  static values = { index: { type: Number, default: 0 } }

  next() {
    this.indexValue = (this.indexValue + 1) % this.slideTargets.length
    this.showCurrentSlide()
  }

  prev() {
    this.indexValue = (this.indexValue - 1 + this.slideTargets.length) % this.slideTargets.length
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((slide, i) => {
      slide.classList.toggle("hidden", i !== this.indexValue)
    })
  }
}
```  

### **3.3. Разметка галереи (ERB + Turbo Frames)**  
```erb
<!-- app/views/photos/_carousel.html.erb -->
<div data-controller="carousel">
  <div class="carousel-container">
    <% photos.each do |photo| %>
      <div class="carousel-slide hidden" data-carousel-target="slide">
        <%= image_tag photo.image.variant(:large), 
                     loading: "lazy", 
                     alt: photo.title,
                     class: "w-full h-auto" %>
        <p class="text-center mt-2"><%= photo.title %></p>
      </div>
    <% end %>
  </div>

  <button data-action="carousel#prev" class="carousel-button">←</button>
  <button data-action="carousel#next" class="carousel-button">→</button>
</div>

<style>
  .carousel-container { position: relative; }
  .carousel-slide { transition: opacity 0.5s; }
  .hidden { display: none; }
  .carousel-button { background: rgba(0,0,0,0.5); color: white; }
</style>
```  

### **3.4. Подключаем в шаблоне**  
```erb
<!-- app/views/photos/index.html.erb -->
<%= render "carousel", photos: @photos %>
```  

---

## **4. Оптимизация и улучшения**  
### **4.1. Ленивая загрузка (`loading="lazy"`)**  
Уменьшает время загрузки страницы.  

### **4.2. Оптимизация изображений**  
```ruby
# app/models/photo.rb
def thumbnail
  image.variant(resize_to_limit: [300, 300])
end

def large
  image.variant(resize_to_limit: [1200, 800])
end
```  

### **4.3. Анимации (CSS/GSAP)**  
Можно добавить плавные переходы:  
```css
.carousel-slide {
  transition: transform 0.5s ease, opacity 0.3s ease;
}
```  

### **4.4. Доступность**  
Добавляем ARIA-атрибуты:  
```erb
<div role="region" aria-label="Фотогалерея">
  <button aria-label="Предыдущее фото">←</button>
  <button aria-label="Следующее фото">→</button>
</div>
```  

---

## **5. Альтернативные решения**  
- **Hotwire (Turbo Streams)** – если нужна динамическая подгрузка.  
- **React/Vue** – если требуется сложная логика (например, drag-and-drop).  
- **Swiper.js** – готовая библиотека для слайдеров.  

---

## **6. Заключение**  
Мы реализовали:  
✅ Модель `Photo` с Active Storage.  
✅ Карусель на Stimulus.  
✅ Ленивую загрузку и оптимизацию.  
✅ Доступность и адаптивность.  

**Домашнее задание:**  
- Добавить индикатор текущего слайда (1/5).  
- Реализовать автопрокрутку через `setInterval`.  
- Настроить Cloudinary для хостинга изображений.  
