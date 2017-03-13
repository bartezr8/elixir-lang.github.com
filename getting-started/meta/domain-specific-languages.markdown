---
layout: getting-started
title: Предметно-ориентированный язык
---

# {{ page.title }}

{% include toc.html %}

## Предисловие

[Предметно-ориентированный язык (DSL)](https://en.wikipedia.org/wiki/Domain-specific_language) позволяет разработчикам адаптировать свои приложения приложение для конкретной предметной области. Для работы с DSL макросы не требуются: любая структура данных или функция объявленная в модуле является частью предметно-ориентированный язык (DSL).

К примеру, представте что нам нужно написать валидатор который будет проверять данные предметно-ориентированного языка (DSL). Мы можем сделать это используя структуры данных, функции или макросы. Давайте посмотрим на что будут похожи разные DSLs:

```elixir
# 1. Структуры данных
import Validator
validate user, name: [length: 1..100],
               email: [matches: ~r/@/]

# 2. Функции
import Validator
user
|> validate_length(:name, 1..100)
|> validate_matches(:email, ~r/@/)

# 3. Макросы + модули
defmodule MyValidator do
  use Validator
  validate_length :name, 1..100
  validate_matches :email, ~r/@/
end

MyValidator.validate(user)
```

Из все способов описанных выше, первый определенно является самым гибким. Если конфигурацию нашей области можно будет закодировать с структурами данных, данная реализация , в стандартной библиотеке Elixir's достаточно функций для работы с различными типами данных.

Второй способ использует вызов функций, такой способ лучше всего подходит для работы с различными API ( например,  если вам необходимо передать несколько параметров) и читать удобно, такая возможность стала доступной в Elixir благодаря оператору *пайп*.

Третий способ, использует макросы, и безусловно является самым сложным. Для написания аналогичного функционала понадобится написать гораздо больше кода, сложно в тестировании (по сравнения с тестированием обычных функций), так же это ограничивает использования данной библиотеки пользователем, так как все проверки должны быть реализованны в модуле.

Для подтверждения данной точки зрения, представте что вы хотите проверить конкретный аттрибут, только в том случае если этот условие соблюдается. Мы легко могли бы это сделать используя первый способ, путем контроля данных, или используя условия (if/else) из второго способа перед вызовом функции. Однако, это невозможно сделать используя просто макросы, без расширения их возможностей через DSL.

Другими словами:

    data > functions > macros

Мы сказали, что есть случаи где использование макросов и модулей где использование придметно-ориентированного языка быает полезно. Так как мы изучили структуры данных и объявление функций ранее, в этом уроке мы сосреготочимся на использовании макросов и модулей для написания более сложных DSLs.

## Проектируем тест кейс

Главной целью данного урока разработка модуля `TestCase`, который мы будем использовать в разработке:

```elixir
defmodule MyTest do
  use TestCase

  test "arithmetic operations" do
    4 = 2 + 2
  end

  test "list operations" do
    [1, 2, 3] = [1, 2] ++ [3]
  end
end

MyTest.run
```

В примере выше, мы подключаем модуль `TestCase`, из него мы будем пользоваться макросом `test` для написания наших тестов, затем мы выполняем функцию `run` для автоматического запуска всех тестов. Наш прототип будет опиратся на использование оператора (`=`), в качестве механизма подтверждения.

## `test` макрос

Для теста создадим модуль в которм объявим и импортируем макрос `test` и затем им воспользуемся:

```elixir
defmodule TestCase do
  # Обратный вызов выполняется `use`.
  #
  # For now it returns a quoted expression that
  # imports the module itself into the user code.
  @doc false
  defmacro __using__(_opts) do
    quote do
      import TestCase
    end
  end

  @doc """
  Defines a test case with the given description.

  ## Примеры

      test "arithmetic operations" do
        4 = 2 + 2
      end

  """
  defmacro test(description, do: block) do
    function_name = String.to_atom("test " <> description)
    quote do
      def unquote(function_name)(), do: unquote(block)
    end
  end
end
```

В примере выше мы объявили модуль `TestCase` в файле `tests.exs`, давайте выполним код выполнив в консоле команду `iex tests.exs` затем напишем наш первый тест:

```iex
iex> defmodule MyTest do
...>   use TestCase
...>
...>   test "hello" do
...>     "hello" = "world"
...>   end
...> end
```

На данный момент у нас нет механизма для запуска тестов, но мы знаем что функция "test hello" была объявлена негласно. Если мы попытаемся вызвать ее, она должна завершится с ошибкой:

```iex
iex> MyTest."test hello"()
** (MatchError) no match of right hand side value: "world"
```

## Хранение информации с атрибутами

Для завершения реализации модуля `TestCase`, нам необходимо обеспечить доступ ко всем тестовым сценариям. Как вариант мы можем вызывать тесты по ходу выполнения через `__MODULE__.__info__(:functions)`, который возвращает список всех функций в данном модуле. Однако, мы можем хранить информацию о каждом тесте после имени теста, это предоставляет более гибкий подход.

Мы рассматривали модули в прошлой главе, при этом упоминали тот факт что их можно использовать в качестве временного хранилища.

В `__using__/1`, мы инициализируем атрибут модуля `@tests` который будет пустым списком, в котором мы будем хранить имя каждого теста, которые затем мы будем вызвать функцией `run`.

Изменим модуль `TestCase` следующим образом:

```elixir
defmodule TestCase do
  @doc false
  defmacro __using__(_opts) do
    quote do
      import TestCase

      # Initialize @tests to an empty list
      @tests []

      # Invoke TestCase.__before_compile__/1 before the module is compiled
      @before_compile TestCase
    end
  end

  @doc """
  Defines a test case with the given description.

  ## Examples

      test "arithmetic operations" do
        4 = 2 + 2
      end

  """
  defmacro test(description, do: block) do
    function_name = String.to_atom("test " <> description)
    quote do
      # Prepend the newly defined test to the list of tests
      @tests [unquote(function_name) | @tests]
      def unquote(function_name)(), do: unquote(block)
    end
  end

  # This will be invoked right before the target module is compiled
  # giving us the perfect opportunity to inject the `run/0` function
  @doc false
  defmacro __before_compile__(env) do
    quote do
      def run do
        Enum.each @tests, fn name ->
          IO.puts "Running #{name}"
          apply(__MODULE__, name, [])
        end
      end
    end
  end
end
```

После того как мы запустим новую сессию IEx, мы можем объявить наши тесты и выполнить их:

```iex
iex> defmodule MyTest do
...>   use TestCase
...>
...>   test "hello" do
...>     "hello" = "world"
...>   end
...> end
iex> MyTest.run
Running test hello
** (MatchError) no match of right hand side value: "world"
```

Несмотря на то что мы рассмотрели не все аспекты связанные с DSL, данных знаний хватит для эффективного использования DSL в Elixir. Макросы позволяют возвращать выражения в ковычках, которые выполняются в вызывающей их функции, которые затем мы можем использовать для преобразования кода и хранения релевантной информации в модуле через атрибуты. Наконец, обратный вызов такой как `@before_compile` позволяет нам инжектить код в модуль.

Помимо атрибута`@before_compile`, у данного модуля есть полезные атрибуты такие как `@on_definition` и `@after_compile`, подробнее о них вы можете прочитать в документации [ модуля `Module`](https://hexdocs.pm/elixir/Module.html). Так же вы можете полезную информацию о макросах и окружении для сборки из  документации по [ модулю `Macro`](https://hexdocs.pm/elixir/Macro.html) и [`Macro.Env`](https://hexdocs.pm/elixir/Macro.Env.html).
