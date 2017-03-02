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

To drive the point home, imagine you want to validate a certain attribute only if a given condition is met. We could easily achieve it with the first solution, by manipulating the data structure accordingly, or with the second solution by using conditionals (if/else) before invoking the function. However it is impossible to do so with the macros approach unless its DSL is augmented.

Другими словами:

    data > functions > macros

That said, there are still cases where using macros and modules to build domain specific languages is useful. Since we have explored data structures and function definitions in the Getting Started guide, this chapter will explore how to use macros and module attributes to tackle more complex DSLs.

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

Assuming we defined `TestCase` in a file named `tests.exs`, we can open it up by running `iex tests.exs` and define our first tests:

```iex
iex> defmodule MyTest do
...>   use TestCase
...>
...>   test "hello" do
...>     "hello" = "world"
...>   end
...> end
```

For now we don't have a mechanism to run tests, but we know that a function named "test hello" was defined behind the scenes. When we invoke it, it should fail:

```iex
iex> MyTest."test hello"()
** (MatchError) no match of right hand side value: "world"
```

## Хранение информации с атрибутами

In order to finish our `TestCase` implementation, we need to be able to access all defined test cases. One way of doing this is by retrieving the tests at runtime via `__MODULE__.__info__(:functions)`, which returns a list of all functions in a given module. However, considering that we may want to store more information about each test besides the test name, a more flexible approach is required.

When discussing module attributes in earlier chapters, we mentioned how they can be used as temporary storage. That's exactly the property we will apply in this section.

In the `__using__/1` implementation, we will initialize a module attribute named `@tests` to an empty list, then store the name of each defined test in this attribute so the tests can be invoked from the `run` function.

Here is the updated code for the `TestCase` module:

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

By starting a new IEx session, we can now define our tests and run them:

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

Although we have overlooked some details, this is the main idea behind creating domain specific modules in Elixir. Macros enable us to return quoted expressions that are executed in the caller, which we can then use to transform code and store relevant information in the target module via module attributes. Finally, callbacks such as `@before_compile` allow us to inject code into the module when its definition is complete.

Besides `@before_compile`, there are other useful module attributes like `@on_definition` and `@after_compile`, which you can read more about in [the docs for the `Module` module](https://hexdocs.pm/elixir/Module.html). You can also find useful information about macros and the compilation environment in the documentation for the [`Macro` module](https://hexdocs.pm/elixir/Macro.html) and [`Macro.Env`](https://hexdocs.pm/elixir/Macro.Env.html).
