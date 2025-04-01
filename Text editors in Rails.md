# Интеграция текстовых редакторов в Rails приложения

## Введение

Текстовые редакторы - важный компонент современных веб-приложений, позволяющий пользователям создавать и форматировать контент. В экосистеме Ruby on Rails существует несколько популярных решений для реализации WYSIWYG-редакторов. Сегодня мы рассмотрим:

1. TinyMCE и его интеграцию через `tinymce-rails`
2. Trix Editor (используемый в Action Text)
3. Кастомизацию редакторов
4. Action Text - стандартное решение Rails

---

## 1. TinyMCE и tinymce-rails

### Что такое TinyMCE?
TinyMCE - это популярный WYSIWYG-редактор с богатым функционалом:
- Поддержка таблиц, изображений, медиа
- Расширяемость через плагины
- Современный интерфейс

### Интеграция в Rails

1. Добавляем гем:
```ruby
# Gemfile
gem 'tinymce-rails'
```

2. Устанавливаем:
```bash
bundle install
rails generate tinymce:install
```

3. Подключаем в application.js:
```javascript
//= require tinymce
```

4. Инициализируем для textarea:
```javascript
$(document).ready(function() {
  tinymce.init({
    selector: 'textarea.tinymce',
    plugins: 'image media link paste',
    toolbar: 'bold italic | link image',
    paste_data_images: true
  });
});
```

5. В форме:
```erb
<%= form.text_area :content, class: 'tinymce' %>
```

### Плюсы TinyMCE
- Богатый функционал
- Много плагинов
- Хорошая документация

### Минусы
- Большой размер бандла
- Проприетарная лицензия для некоторых функций

---

## 2. Trix Editor и Action Text

### Что такое Trix?
Trix - это редактор от Basecamp (создателей Rails) с минималистичным дизайном, но мощными возможностями.

### Action Text
Action Text - стандартное решение Rails для работы с богатым текстом, использующее Trix.

Установка:
```bash
rails action_text:install
```

Это добавит:
1. Миграции для хранения контента
2. Конфигурацию Active Storage
3. Подключение Trix в application.js

Использование:
```ruby
# Модель
has_rich_text :content
```

```erb
<!-- Форма -->
<%= form.rich_text_area :content %>
```

### Основные возможности Trix:
- Вставка файлов и изображений
- Форматирование текста
- Цитаты, списки, ссылки
- Простое API для расширения

---

## 3. Кастомизация редакторов

### Кастомизация TinyMCE

Добавление кнопок:
```javascript
tinymce.init({
  toolbar: 'customButton',
  setup: function(editor) {
    editor.ui.registry.addButton('customButton', {
      text: 'Моя кнопка',
      onAction: function() {
        editor.insertContent('Кастомный текст');
      }
    });
  }
});
```

### Кастомизация Trix

1. Добавление кнопок в панель инструментов:

Создаем файл `app/javascript/trix-custom.js`:
```javascript
addEventListener("trix-initialize", function(event) {
  const buttonHTML = `<button type="button" class="trix-button" data-trix-action="custom">Custom</button>`
  
  const buttonGroup = event.target.toolbarElement.querySelector(".trix-button-group--block-tools")
  buttonGroup.insertAdjacentHTML("beforeend", buttonHTML)
  
  addEventListener("trix-action-invoke", function(event) {
    if (event.actionName === "custom") {
      event.target.editor.insertString("Кастомный текст")
    }
  })
})
```

2. Добавление стилей:
```css
.trix-button[data-trix-action="custom"]::after {
  content: "C";
  font-weight: bold;
}
```

---

## 4. Продвинутая интеграция

### Загрузка файлов в TinyMCE

1. Настройка загрузчика:
```javascript
tinymce.init({
  images_upload_handler: function(blobInfo, success, failure) {
    const formData = new FormData();
    formData.append('file', blobInfo.blob(), blobInfo.filename());
    
    fetch('/uploads', {
      method: 'POST',
      body: formData,
      headers: {
        'X-CSRF-Token': document.querySelector("[name='csrf-token']").content
      }
    })
    .then(response => response.json())
    .then(data => success(data.location))
    .catch(() => failure('Upload failed'));
  }
});
```

2. Контроллер Rails:
```ruby
def create
  upload = Upload.create!(file: params[:file])
  render json: { location: url_for(upload.file) }
end
```

### Расширение Action Text

Добавление кастомных атрибутов:
```javascript
document.addEventListener("trix-initialize", function(event) {
  const { toolbarElement } = event.target
  
  // Добавляем кнопку для выделения текста
  const highlightButton = document.createElement("button")
  highlightButton.setAttribute("type", "button")
  highlightButton.setAttribute("class", "trix-button")
  highlightButton.setAttribute("data-trix-attribute", "highlight")
  highlightButton.setAttribute("title", "Highlight")
  highlightButton.textContent = "H"
  
  const blockTools = toolbarElement.querySelector(".trix-button-group--block-tools")
  blockTools.appendChild(highlightButton)
  
  // Обработка стиля
  Trix.config.textAttributes.highlight = {
    style: { backgroundColor: "yellow" },
    parser: function(element) {
      return element.style.backgroundColor === "yellow"
    },
    inheritable: true
  }
})
```

---

## Сравнение решений

| Критерий          | TinyMCE               | Trix (Action Text)     |
|-------------------|-----------------------|------------------------|
| Функционал        | Богатый               | Минималистичный        |
| Кастомизация      | Гибкая                | Ограниченная           |
| Размер            | Большой               | Компактный             |
| Интеграция с Rails| Через гем             | Нативная (Action Text) |
| Хранение данных   | HTML                  | HTML + Active Storage  |
| Лицензия          | Проприетарные функции | MIT                    |

---

## Заключение

1. **Для сложных редакторов** с множеством функций - выбирайте TinyMCE
2. **Для стандартных задач** - используйте Action Text с Trix
3. **Для максимальной кастомизации** - рассмотрите создание собственного решения на основе Trix

### Домашнее задание

1. Создайте модель Article с rich text полем
2. Интегрируйте Action Text
3. Добавьте кастомную кнопку в панель инструментов
4. Реализуйте загрузку изображений через Active Storage

### Полезные ссылки

- [Документация TinyMCE](https://www.tiny.cloud/docs/)
- [Trix GitHub](https://github.com/basecamp/trix)
- [Action Text Guide](https://guides.rubyonrails.org/action_text_overview.html)

Удачной разработки! 🚀
