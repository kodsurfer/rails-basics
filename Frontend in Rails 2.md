# Лекция: Фронтенд в Ruby on Rails (продолжение)

## Часть 1: Продвинутое использование Hotwire

### 1.1 Turbo Streams + Action Cable (Real-Time Updates)

Пример чата с мгновенными сообщениями:

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_room_#{params[:room_id]}"
  end

  def receive(data)
    message = Message.create!(content: data['message'], user: current_user)
    broadcast_to("chat_room_#{params[:room_id]}", 
      turbo_stream.append(:messages, partial: 'messages/message', locals: { message: message }))
  end
end
```

```erb
<%# app/views/chat/show.html.erb %>
<%= turbo_stream_from "chat_room_#{@room.id}" %>

<div id="messages">
  <%= render @room.messages %>
</div>

<%= form_with model: Message.new, url: messages_path, data: { controller: "reset-form" } do |f| %>
  <%= f.hidden_field :room_id, value: @room.id %>
  <%= f.text_field :content %>
  <%= f.submit "Send" %>
<% end %>
```

### 1.2 Custom Turbo Stream Actions

Создание кастомных действий:

```ruby
# app/helpers/turbo_streams_helper.rb
module TurboStreamsHelper
  def turbo_notify(message, type: :info)
    turbo_stream_action_tag(:notify, message: message, type: type)
  end
end
```

```javascript
// app/javascript/application.js
Turbo.StreamActions.notify = function() {
  const message = this.getAttribute('message');
  const type = this.getAttribute('type');
  showNotification(message, type); // Ваша функция нотификации
}
```

## Часть 2: Продвинутый Stimulus

### 2.1 Работа с внешними API

Пример поиска с автодополнением:

```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "results"]
  static values = { url: String }

  async search() {
    const response = await fetch(`${this.urlValue}?q=${this.inputTarget.value}`)
    this.resultsTarget.innerHTML = await response.text()
  }

  debounce(fn, delay = 300) {
    let timeout
    return (...args) => {
      clearTimeout(timeout)
      timeout = setTimeout(() => fn.apply(this, args), delay)
    }
  }

  initialize() {
    this.search = this.debounce(this.search)
  }
}
```

```erb
<div data-controller="search" data-search-url-value="<%= search_path %>">
  <input type="text" data-search-target="input" data-action="input->search#search">
  <div data-search-target="results"></div>
</div>
```

### 2.2 Интеграция с JavaScript-библиотеками

Пример интеграции с Chart.js:

```javascript
// app/javascript/controllers/chart_controller.js
import { Controller } from "@hotwired/stimulus"
import Chart from 'chart.js/auto'

export default class extends Controller {
  static values = { data: Array }

  connect() {
    this.chart = new Chart(this.element, {
      type: 'line',
      data: {
        labels: ['Jan', 'Feb', 'Mar'],
        datasets: [{
          label: 'Sales',
          data: this.dataValue
        }]
      }
    })
  }

  disconnect() {
    this.chart.destroy()
  }
}
```

```erb
<canvas data-controller="chart" data-chart-data-value="[10, 20, 30]"></canvas>
```

## Часть 3: Компонентный подход

### 3.1 ViewComponent + Tailwind

Пример кнопки с вариантами:

```ruby
# app/components/button_component.rb
class ButtonComponent < ViewComponent::Base
  VARIANTS = {
    primary: "bg-blue-500 text-white",
    secondary: "bg-gray-200 text-black",
    danger: "bg-red-500 text-white"
  }

  def initialize(variant: :primary, **options)
    @variant = variant
    @options = options
  end

  def css_classes
    [VARIANTS[@variant], "px-4 py-2 rounded"].join(" ")
  end
end
```

```erb
<%# app/components/button_component.html.erb %>
<%= content_tag :button, content, class: css_classes, **@options %>
```

Использование:
```erb
<%= render ButtonComponent.new(variant: :primary, data: { action: "click->modal#open" }) do %>
  Open Modal
<% end %>
```

### 3.2 Stimulus + ViewComponent

Интерактивный аккордеон:

```ruby
# app/components/accordion_component.rb
class AccordionComponent < ViewComponent::Base
  renders_many :items, AccordionItemComponent
end

class AccordionItemComponent < ViewComponent::Base
  def initialize(title:)
    @title = title
  end
end
```

```erb
<%# app/components/accordion_component.html.erb %>
<div data-controller="accordion">
  <%= items.each do |item| %>
    <%= item %>
  <% end %>
</div>

<%# app/components/accordion_item_component.html.erb %>
<div class="accordion-item" data-accordion-target="item">
  <button data-action="click->accordion#toggle">
    <%= @title %>
  </button>
  <div class="content hidden">
    <%= content %>
  </div>
</div>
```

## Часть 4: Оптимизация производительности

### 4.1 Ленивая загрузка

Использование lazy-load для Turbo Frames:

```erb
<%= turbo_frame_tag "lazy_comments", src: comments_path(post), loading: :lazy %>
```

### 4.2 Предзагрузка данных

```ruby
# app/controllers/posts_controller.rb
def show
  @post = Post.find(params[:id])
  @comments = @post.comments
  preload_for @comments, :user
end
```

### 4.3 Кэширование компонентов

```erb
<% cache @post do %>
  <%= render PostComponent.new(post: @post) %>
<% end %>
```

## Часть 5: Тестирование

### 5.1 Системные тесты

Пример теста для Turbo Stream:

```ruby
# test/system/messages_test.rb
require "application_system_test_case"

class MessagesTest < ApplicationSystemTestCase
  test "creating message updates list" do
    visit chat_room_path
    
    fill_in "Message", with: "Hello world"
    click_button "Send"

    assert_selector ".message", text: "Hello world"
    assert_equal current_path, chat_room_path
  end
end
```

### 5.2 Тестирование Stimulus контроллеров

Использование Stimulus Testing Library:

```javascript
// test/javascript/controllers/search_controller.test.js
import { Application } from "@hotwired/stimulus"
import SearchController from "controllers/search_controller"

describe("SearchController", () => {
  beforeEach(() => {
    document.body.innerHTML = `
      <div data-controller="search">
        <input data-search-target="input">
        <div data-search-target="results"></div>
      </div>
    `
    
    const application = Application.start()
    application.register("search", SearchController)
  })

  it("updates results on input", async () => {
    const input = document.querySelector("[data-search-target='input']")
    input.value = "test"
    input.dispatchEvent(new Event('input'))
    
    await new Promise(resolve => setTimeout(resolve, 300))
    
    const results = document.querySelector("[data-search-target='results']")
    expect(results.innerHTML).toContain("Search results")
  })
})
```

## Полезные ресурсы

1. **[Hotwire Handbook](https://hotwired.dev/)** - официальная документация
2. **[Stimulus Components](https://stimulus-components.netlify.app/)** - коллекция готовых компонентов
3. **[ViewComponent](https://viewcomponent.org/)** - документация компонентов
4. **[Turbo Testing](https://turbo.hotwired.dev/handbook/testing)** - руководство по тестированию
5. **[Rails Frontend Checklist](https://github.com/ledermann/rails-frontend-checklist)** - чеклист лучших практик

## Лучшие практики

1. **Инкрементальная загрузка**: Используйте Turbo Frames для частичных обновлений
2. **Прогрессивное улучшение**: Обеспечьте базовую функциональность без JavaScript
3. **Декларативный подход**: Используйте data-атрибуты вместо кастомных классов
4. **Оптимизация сборки**: Разделяйте код на чанки с помощью dynamic imports
5. **Безопасность**: Всегда санируйте HTML при использовании `render html: ...`

**Пример продвинутой оптимизации:**

```javascript
// app/javascript/controllers/lazy_load_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["content"]
  static values = { url: String }

  connect() {
    if ('IntersectionObserver' in window) {
      this.observer = new IntersectionObserver(entries => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            this.loadContent()
            this.observer.unobserve(this.element)
          }
        })
      })
      
      this.observer.observe(this.element)
    } else {
      this.loadContent()
    }
  }

  async loadContent() {
    const response = await fetch(this.urlValue)
    this.contentTarget.innerHTML = await response.text()
  }
}
```

```erb
<div data-controller="lazy-load" data-lazy-load-url-value="<%= heavy_content_path %>">
  <div data-lazy-load-target="content">
    Loading...
  </div>
</div>
```

## Заключение

Современный фронтенд в Rails предлагает богатые возможности для создания интерактивных приложений без сложных SPA-решений. Ключевые моменты:

- **Гибкая архитектура**: Комбинируйте Hotwire, Stimulus и компоненты
- **Производительность**: Используйте ленивую загрузку и кэширование
- **Тестирование**: Полноценное покрытие всех сценариев
- **Экосистема**: Используйте богатый набор инструментов Rails-сообщества

**Финальное задание:**
1. Реализуйте интерактивную форму с валидацией через Turbo Streams
2. Создайте компонент дашборда с графиками на Stimulus
3. Настройте ленивую загрузку для тяжелых компонентов
4. Напишите системные тесты для всех сценариев

**Дополнительные материалы:**
- [Hotwire Tutorial](https://www.hotrails.dev/turbo-rails)
- [Stimulus Cheat Sheet](https://stimulus.hotwired.dev/reference/cheatsheet)
- [Rails Frontend Patterns](https://frontend.rails-frontend.com/)