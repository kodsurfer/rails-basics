## Основы Ruby

---

### Что такое Ruby?

* **Ruby:** 
    * Динамический, объектно-ориентированный язык программирования.
    * Создан Юкихиро Мацумото (Matz) в 1995 году.
* **Основные характеристики:**
    * Простой и читаемый синтаксис
    * Выразительность и гибкость
    * Поддержка ООП и функционального программирования

---

### Установка Ruby

* **Установка на macOS:**

```bash
brew install ruby
```

* **Установка на Linux:**

```bash
sudo apt-get install ruby-full
```

* **Установка на Windows:**
    * Используйте RubyInstaller: [rubyinstaller.org](https://rubyinstaller.org/)

---

### Основные типы данных

* **Числа:** 
    * Целые числа (`Integer`)
    * Числа с плавающей точкой (`Float`)
* **Строки:** 
    * Текстовые данные (`String`)
* **Логические значения:** 
    * `true` и `false`
* **Массивы:** 
    * Упорядоченные коллекции элементов (`Array`)
* **Хэши:** 
    * Коллекции пар ключ-значение (`Hash`)

```ruby
# Примеры
number = 42
float = 3.14
string = "Hello, Ruby!"
boolean = true
array = [1, 2, 3]
hash = { name: "John", age: 30 }
```

---

### Операторы

* **Арифметические операторы:** 
    * `+`, `-`, `*`, `/`, `%`, `**`
* **Операторы сравнения:** 
    * `==`, `!=`, `>`, `<`, `>=`, `<=`
* **Логические операторы:** 
    * `&&`, `||`, `!`
* **Операторы присваивания:** 
    * `=`, `+=`, `-=`, `*=`, `/=`, `%=`

```ruby
# Примеры
sum = 10 + 5
is_equal = (10 == 5)
is_true = (true && false)
x = 10
x += 5
```

---

### Управляющие структуры

* **Условные операторы:** 
    * `if`, `elsif`, `else`, `unless`
* **Циклы:** 
    * `while`, `until`, `for`, `each`
* **Операторы выбора:** 
    * `case`

```ruby
# Примеры
if x > 10
  puts "x is greater than 10"
elsif x == 10
  puts "x is equal to 10"
else
  puts "x is less than 10"
end

array.each do |element|
  puts element
end

case x
when 10
  puts "x is 10"
when 20
  puts "x is 20"
else
  puts "x is something else"
end
```

---

### Методы

* **Методы:** 
    * Группировка кода для повторного использования.
* **Определение метода:** 
    * `def` ... `end`
* **Аргументы:** 
    * Передача данных в метод.

```ruby
def greet(name)
  puts "Hello, #{name}!"
end

greet("Ruby")  # Hello, Ruby!
```

---

### Классы и объекты

* **Класс:** 
    * Шаблон для создания объектов.
* **Объект:** 
    * Экземпляр класса.
* **Атрибуты и методы:** 
    * Состояние и поведение объекта.

```ruby
class Dog
  attr_accessor :name

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

### Модули

* **Модули:** 
    * Группировка методов и констант.
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

### Заключение

* **Ruby – мощный язык:** 
    * Простой и выразительный синтаксис.
    * Поддержка ООП и функционального программирования.
* **Основные компоненты:** 
    * Типы данных, операторы, управляющие структуры, методы, классы, модули.

---

### Вопросы?

Спасибо за внимание!

---
