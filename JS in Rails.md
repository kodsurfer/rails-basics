## Лекция: JavaScript в Rails в 2025

---

### 1. Webpacker has been retired

#### Почему Webpacker ушел?
Webpacker был популярным инструментом для управления JavaScript в Rails, но он стал слишком сложным для многих разработчиков. Его настройка требовала глубокого понимания Webpack, что часто было избыточным для небольших проектов. Кроме того, сообщество Rails стремилось упростить процесс работы с JavaScript, чтобы разработчики могли сосредоточиться на написании кода, а не на настройке инструментов.

#### Альтернативы Webpacker

1. **Import Maps**:
   - **Что это?** Import Maps — это стандарт браузера, который позволяет подключать JavaScript-модули напрямую из браузера без необходимости сборки.
   - **Как это работает?** Вместо того чтобы использовать сборщик, Rails генерирует JSON-файл (`importmap.rb`), который указывает браузеру, где искать модули.
   - **Пример настройки**:
     ```ruby
     # Gemfile
     gem "importmap-rails"
     ```
     После установки гема выполните:
     ```bash
     rails importmap:install
     ```
     Это создаст файл `config/importmap.rb`, где вы можете указать зависимости:
     ```ruby
     # config/importmap.rb
     pin "application", preload: true
     pin "stimulus", to: "https://cdn.jsdelivr.net/npm/stimulus@3.2.1/dist/stimulus.js"
     ```

2. **esbuild**:
   - **Что это?** esbuild — это быстрый и легковесный сборщик JavaScript, который компилирует код в несколько раз быстрее, чем Webpack.
   - **Как это работает?** esbuild интегрируется с Rails через гем `jsbundling-rails`.
   - **Пример настройки**:
     ```ruby
     # Gemfile
     gem "jsbundling-rails"
     ```
     Установите esbuild:
     ```bash
     rails javascript:install:esbuild
     ```
     Это создаст файл `app/javascript/application.js`, где вы можете импортировать модули:
     ```javascript
     // app/javascript/application.js
     import "stimulus"
     ```

---

### 2. Working with JavaScript in Rails

#### Stimulus
Stimulus — это минималистичный фреймворк для добавления интерактивности на стороне клиента. Он идеально подходит для небольших проектов или для добавления интерактивности в существующие Rails-приложения.

- **Основные концепции**:
  - **Контроллеры**: Stimulus использует контроллеры для управления поведением элементов на странице.
  - **Цели (Targets)**: Позволяют ссылаться на элементы DOM внутри контроллера.
  - **Действия (Actions)**: Обрабатывают события, такие как клики или отправка форм.

- **Пример**:
  ```javascript
  // app/javascript/controllers/hello_controller.js
  import { Controller } from "@hotwired/stimulus"

  export default class extends Controller {
    static targets = ["name"]

    greet() {
      alert(`Hello, ${this.nameTarget.value}!`)
    }
  }
  ```

  ```html
  <!-- app/views/posts/index.html.erb -->
  <div data-controller="hello">
    <input type="text" data-hello-target="name">
    <button data-action="click->hello#greet">Greet</button>
  </div>
  ```

#### Turbo
Turbo — это набор инструментов для ускорения навигации и работы с формами без перезагрузки страницы. Он состоит из трех основных компонентов:

1. **Turbo Drive**:
   - Ускоряет навигацию, загружая только содержимое страницы, а не весь HTML.
   - Пример:
     ```ruby
     # config/application.rb
     config.action_view.form_with_generates_remote_forms = true
     ```

2. **Turbo Frames**:
   - Позволяет обновлять только часть страницы.
   - Пример:
     ```erb
     <!-- app/views/posts/index.html.erb -->
     <%= turbo_frame_tag "post_1" do %>
       <%= render @post %>
     <% end %>
     ```

3. **Turbo Streams**:
   - Позволяет обновлять несколько частей страницы одновременно.
   - Пример:
     ```ruby
     # Контроллер
     def update
       @post = Post.find(params[:id])
       @post.update(post_params)

       respond_to do |format|
         format.turbo_stream
       end
     end
     ```

     ```erb
     <!-- app/views/posts/update.turbo_stream.erb -->
     <%= turbo_stream.replace @post do %>
       <%= render @post %>
     <% end %>
     ```

---

### 3. Realtime Web и WebSockets на Rails с Turbo

#### Action Cable
Action Cable — это встроенный в Rails инструмент для работы с WebSockets. Он позволяет создавать приложения с реальным временем, такие как чаты, уведомления и онлайн-игры.

- **Как это работает?**:
  - Сервер поддерживает постоянное соединение с клиентом.
  - Данные передаются в реальном времени без необходимости обновления страницы.

- **Пример: Чат в реальном времени**:
  1. **Настройка Action Cable**:
     ```yaml
     # config/cable.yml
     development:
       adapter: async
     ```

  2. **Создание канала**:
     ```bash
     rails generate channel Chat
     ```

  3. **Код канала**:
     ```ruby
     # app/channels/chat_channel.rb
     class ChatChannel < ApplicationCable::Channel
       def subscribed
         stream_from "chat_room"
       end
     end
     ```

  4. **Клиентская часть**:
     ```javascript
     // app/javascript/channels/chat_channel.js
     import consumer from "./consumer"

     consumer.subscriptions.create("ChatChannel", {
       received(data) {
         console.log("New message:", data)
       }
     })
     ```

  5. **Отправка сообщений**:
     ```ruby
     # Контроллер
     def create
       message = Message.create!(message_params)
       ActionCable.server.broadcast("chat_room", message)
     end
     ```

#### Turbo + Action Cable
Turbo Streams можно использовать вместе с Action Cable для обновления интерфейса в реальном времени. Например, при получении нового сообщения в чате можно обновить список сообщений на странице.

- **Пример**:
  ```ruby
  # app/channels/chat_channel.rb
  class ChatChannel < ApplicationCable::Channel
    def subscribed
      stream_from "chat_room"
    end

    def receive(data)
      ActionCable.server.broadcast("chat_room", turbo_stream: data)
    end
  end
  ```

  ```erb
  <!-- app/views/messages/_message.html.erb -->
  <%= turbo_stream.append "messages" do %>
    <%= render @message %>
  <% end %>
  ```

---

### Заключение
В 2025 году Rails предлагает мощные и простые инструменты для работы с JavaScript. Import Maps и esbuild упрощают управление зависимостями, Stimulus добавляет интерактивность, а Turbo и Action Cable позволяют создавать приложения с реальным временем. Эти инструменты делают Rails еще более привлекательным для современных веб-разработчиков.

#### Дополнительные ресурсы
- [Официальная документация по Import Maps](https://github.com/rails/importmap-rails)
- [Документация Stimulus](https://stimulus.hotwired.dev/)
- [Turbo Handbook](https://turbo.hotwired.dev/)
- [Action Cable Guide](https://guides.rubyonrails.org/action_cable_overview.html)

Удачи в изучении! 🚀
