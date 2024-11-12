## Основы создания API на Rails

### Введение

В современном мире веб-разработки API (Application Programming Interface) играют ключевую роль в обеспечении взаимодействия между различными сервисами и приложениями. Ruby on Rails, мощный фреймворк для разработки веб-приложений, также предоставляет удобные инструменты для создания API. В этой лекции мы рассмотрим основы создания API на Rails, используя видеоурок по ссылке: [Основы создания API на Rails](https://www.youtube.com/watch?v=GZPgCVnFSQ4).

### 1. Настройка проекта

#### 1.1. Создание нового Rails приложения

Для начала создадим новое Rails приложение, которое будет служить основой для нашего API:

```bash
rails new my_api --api
```

Опция `--api` указывает Rails создать приложение в режиме API, что означает, что будут исключены компоненты, не нужные для API, такие как представления (views) и middleware для обработки HTML.

#### 1.2. Настройка базы данных

По умолчанию Rails использует SQLite, но для реальных проектов рекомендуется использовать более мощные базы данных, такие как PostgreSQL или MySQL. Настроим PostgreSQL:

1. Установите PostgreSQL на вашу машину.
2. Откройте файл `config/database.yml` и настройте параметры подключения к базе данных.

### 2. Создание моделей и миграций

#### 2.1. Генерация модели

Создадим модель `Article` с атрибутами `title` и `content`:

```bash
rails generate model Article title:string content:text
```

Эта команда создаст файл модели `app/models/article.rb` и файл миграции для создания соответствующей таблицы в базе данных.

#### 2.2. Выполнение миграции

Примените миграцию для создания таблицы в базе данных:

```bash
rails db:migrate
```

### 3. Создание контроллеров

#### 3.1. Генерация контроллера

Создадим контроллер `ArticlesController`, который будет обрабатывать запросы к API:

```bash
rails generate controller Articles
```

#### 3.2. Определение эндпоинтов

В файле `app/controllers/articles_controller.rb` определим основные эндпоинты для работы с ресурсами:

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :update, :destroy]

  # GET /articles
  def index
    @articles = Article.all
    render json: @articles
  end

  # GET /articles/:id
  def show
    render json: @article
  end

  # POST /articles
  def create
    @article = Article.new(article_params)

    if @article.save
      render json: @article, status: :created, location: @article
    else
      render json: @article.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /articles/:id
  def update
    if @article.update(article_params)
      render json: @article
    else
      render json: @article.errors, status: :unprocessable_entity
    end
  end

  # DELETE /articles/:id
  def destroy
    @article.destroy
  end

  private

  def set_article
    @article = Article.find(params[:id])
  end

  def article_params
    params.require(:article).permit(:title, :content)
  end
end
```

### 4. Маршрутизация

#### 4.1. Настройка маршрутов

В файле `config/routes.rb` настроим маршруты для нашего API:

```ruby
Rails.application.routes.draw do
  resources :articles
end
```

Эта строка создаст все стандартные маршруты для ресурса `articles`, включая `index`, `show`, `create`, `update` и `destroy`.

### 5. Тестирование API

#### 5.1. Использование Postman

Для тестирования API можно использовать инструмент Postman. Создайте запросы к различным эндпоинтам и убедитесь, что они работают корректно.

#### 5.2. Автоматизированное тестирование

Rails также предоставляет инструменты для автоматизированного тестирования. Создайте файл `test/controllers/articles_controller_test.rb` и напишите тесты для проверки работы API:

```ruby
require 'test_helper'

class ArticlesControllerTest < ActionDispatch::IntegrationTest
  setup do
    @article = articles(:one)
  end

  test "should get index" do
    get articles_url, as: :json
    assert_response :success
  end

  test "should create article" do
    assert_difference('Article.count') do
      post articles_url, params: { article: { title: 'New Article', content: 'Content' } }, as: :json
    end

    assert_response :created
  end

  test "should show article" do
    get article_url(@article), as: :json
    assert_response :success
  end

  test "should update article" do
    patch article_url(@article), params: { article: { title: 'Updated Article' } }, as: :json
    assert_response :success
  end

  test "should destroy article" do
    assert_difference('Article.count', -1) do
      delete article_url(@article), as: :json
    end

    assert_response :no_content
  end
end
```

### 6. Заключение

В этой лекции мы рассмотрели основы создания API на Rails. Мы создали новое Rails приложение в режиме API, настроили базу данных, создали модели и контроллеры, определили маршруты и протестировали API. Rails предоставляет мощные инструменты для быстрой и эффективной разработки API, что делает его отличным выбором для разработчиков.

### Дополнительные ресурсы

- [Официальная документация Rails](https://guides.rubyonrails.org/)
- [Postman](https://www.postman.com/)
- [RSpec для тестирования в Rails](https://rspec.info/)

Надеюсь, эта лекция помогла вам понять основы создания API на Rails. Удачи в ваших проектах!
