---

# Объектно-Ориентированное Программирование в Ruby

---

## Что такое ООП?

* **Парадигма программирования:** 
    * Организация кода вокруг объектов.
    * Объекты – экземпляры классов, обладающие состоянием и поведением.
* **Основные принципы:**
    * Инкапсуляция
    * Наследование
    * Полиморфизм

---

## Классы и объекты в Ruby

* **Класс:** 
    * Шаблон для создания объектов.
    * Определяет состояние (атрибуты) и поведение (методы).
* **Объект:** 
    * Экземпляр класса.
    * Имеет собственные значения атрибутов.

```ruby
class Dog
  def initialize(name)
    @name = name
  end

  def bark
    puts "#{@name} says woof!"
  end
end

my_dog = Dog.new("Rex")
my_dog.bark  # Rex says woof!
```

---

## Инкапсуляция

* **Скрытие деталей реализации:** 
    * Доступ к атрибутам и методам ограничен.
* **Доступ через методы:** 
    * `attr_reader`, `attr_writer`, `attr_accessor`

```ruby
class Dog
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def bark
    puts "#{@name} says woof!"
  end
end

my_dog = Dog.new("Rex")
puts my_dog.name  # Rex
```

---

## Наследование

* **Создание иерархии классов:** 
    * Подкласс наследует атрибуты и методы суперкласса.
* **Переопределение методов:** 
    * Подкласс может изменять поведение методов суперкласса.

```ruby
class Animal
  def speak
    puts "Some generic sound"
  end
end

class Dog < Animal
  def speak
    puts "Woof!"
  end
end

my_dog = Dog.new
my_dog.speak  # Woof!
```

---

## Полиморфизм

* **Одинаковый интерфейс, разная реализация:** 
    * Разные классы могут реализовывать один и тот же метод по-разному.

```ruby
class Animal
  def speak
    puts "Some generic sound"
  end
end

class Dog < Animal
  def speak
    puts "Woof!"
  end
end

class Cat < Animal
  def speak
    puts "Meow!"
  end
end

animals = [Dog.new, Cat.new]
animals.each { |animal| animal.speak }  # Woof! Meow!
```

---

## Модули

* **Группировка методов:** 
    * Похоже на класс, но не может создавать объекты.
* **Использование:** 
    * Включение в классы (`include`), расширение классов (`extend`).

```ruby
module Flyable
  def fly
    puts "I'm flying!"
  end
end

class Bird
  include Flyable
end

my_bird = Bird.new
my_bird.fly  # I'm flying!
```

---

## Заключение

* **Ruby – язык с сильной поддержкой ООП:** 
    * Все – объекты.
    * Гибкость и выразительность.
* **ООП – мощный инструмент:** 
    * Повторное использование кода.
    * Упрощение структуры программы.

---

## Вопросы?

Спасибо за внимание!

---
