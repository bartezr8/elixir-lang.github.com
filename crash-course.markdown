---
title: "Erlang/Elixir синтаксис: обзорный курс"
section: home
layout: default
---

# {{ page.title }}

Данное руководство предназначено Erlang разработчикам, для быстро изучения синтаксиса Elixir. В данном руководстве собран абсолютный минимум знаний необходимый для понимания кода написанного на Elixir/Erlang, осуществления поддержки, чтения документации, примеров кода и т.д.

{% include toc.html %}

## Выполнения кода

### Erlang

Самый быстрый способ начать писать код, это запустить командную оболочку Erlang выполнив команду -- `erl`. Многие примеры с данной страницы можно скопировать и запустить в командной оболочке. Однако, когда вам нужно объявить именованную функцию, Erlang ожидает что она будет находится внутри модуля, и модуль должен быть скомпилирован. Ниже представлен шаблон простого модуля:

```erlang
% module_name.erl
-module(module_name).  % вы можете использовать любое имя
-compile(export_all).

hello() ->
  io:format("~s~n", ["Hello world!"]).
```

Добавьте в модуль свою функцию, сохраните файл на диск, выполните `erl` в той же директории и затем выполните команду `compile`:

```erl
Eshell V5.9  (abort with ^G)
1> c(module_name).
ok
1> module_name:hello().
Hello world!
ok
```

Вы можете держать оболочку активной, во время редактированния файла. Не забудте выполнить команду `c(module_name)` для подгрузки последних изменений. Учтите что имя файла должно совпадать с тем что указанно в директиве `-module()`, полюс расширение `.erl`.

### Elixir

У Elixir так же есть интерактивная оболочка называемая `iex`. Скомпилировать код Elixir можно выполнив команду `elixirc` (которая аналогична команде `erlc` в Erlang). Elixir так же предоставляет команду `elixir` для запуска кода Elixir. В Elixir модули объявляются следующим образом:

```elixir
# module_name.ex
defmodule ModuleName do
  def hello do
    IO.puts "Hello World"
  end
end
```

и компилируются через `iex`:

```iex
Interactive Elixir
iex> c("module_name.ex")
[ModuleName]
iex> ModuleName.hello
Hello world!
:ok
```

Однако в Elixir нет необходимости создавать на каждый модуль отдельный файл, модули в Elixir могут быть объявлены в интерактивной оболочке:

```elixir
defmodule MyModule do
  def hello do
    IO.puts "Another Hello"
  end
end
```


## Заметный отличия

В данном разделе представлены различия в синтаксисе этих языков.

### Имена операторов

Некоторые операторы пишутся по разному.

| Erlang         | Elixir         | Meaning                                 |
|----------------|----------------|-----------------------------------------|
| and            | недоступен     | Логическое 'и', вычисляет оба аргумента |
| andalso        | and            | Логическое 'и', краткая запись          |
| or             | недоступен     | Логическое 'или', вычисляет оба аргумента |
| orelse         | or             | Логическое 'или', краткая запись        |
| =:=            | ===            | Оператор равенства                      |
| =/=            | !==            | Оператор неравенства                    |
| /=             | !=             | не равно                                |
| =<             | <=             | меньше или равно                        |


### Разделители

Выражения в Erlang разделяются точкой `.` и запятой `,` это необходимо для вычисления нескольких выражений с одним контекстом (при объявлении функции, для экземпляров). В Elixir, выражения разделяются символом переноса строки или точкой с запятой `;`.

**Erlang**

```erlang
X = 2, Y = 3.
X + Y.
```

**Elixir**

```elixir
x = 2; y = 3
x + y
```

### Имена переменных

Значения переменных в Erlang могут быть присвоены только один раз. В интерактивной оболочке Erlang доступна специальная команда `f` которая позволяет стирать значение связанное с переменной или всех переменных за раз.

Elixir позволяет присваивать значение переменной многократно. Если вам нужно получить значение переменной которое у неё было до повторного присвоения, вы может воспользоватся оператором `^`:

**Erlang**

```erl
Eshell V5.9  (abort with ^G)
1> X = 10.
10
2> X = X + 1.
** exception error: no match of right hand side value 11
3> X1 = X + 1.
11
4> f(X).
ok
5> X = X1 * X1.
121
6> f().
ok
7> X.
* 1: variable 'X' is unbound
8> X1.
* 1: variable 'X1' is unbound
```

**Elixir**

```iex
iex> a = 1
1
iex> a = 2
2
iex> ^a = 3
** (MatchError) no match of right hand side value: 3
```

### Вызов функций

Для вызова функции из модуля используется разный синстаксис. В Erlang, это делается так:

```erlang
lists:last([1, 2]).
```

для вызова функции `last` из модуля `List`. В Elixir, используется точка `.` вместо двоеточия `:`

```elixir
List.last([1, 2])
```

**Note**. Поскольку в Erlang модули представлены атомами, вы можете вызывать Erlang функции в Elixir:

```elixir
:lists.sort([3, 2, 1])
```

Все встороенные Erlang модули находятся в модуле `:erlang`.


## Типы данных

Erlang и Elixir имеют схожите типы данных, но есть пара отличий.

### Атомы

В Erlang, атомом `atom` является любой идентификатор который начинается с маленькой буквы, т.е. `ok`, `tuple`, `donut`. Идентификаторы начинающиеся с большой буквы считаются переменной. С другой стороны в Elixir, первый вариант используется для обозначения переменных, а второй - как псевдонимы атомов. В Elixir атомы всегда начинаются с двоеточия `:`.

**Erlang**

```erlang
im_an_atom.
me_too.

Im_a_var.
X = 10.
```

**Elixir**

```elixir
:im_an_atom
:me_too

im_a_var
x = 10

Module  # здесь мы вызываем псевдоним атома; который будет преобразован в :'Elixir.Module'
```

Так же можно создать атом который будет начинатся с буквы в нижнем регистре. Синтаксис в этих языках отличается:

**Erlang**

```erlang
is_atom(ok).                %=> true
is_atom('0_ok').            %=> true
is_atom('Multiple words').  %=> true
is_atom('').                %=> true
```

**Elixir**

```elixir
is_atom :ok                 #=> true
is_atom :'ok'               #=> true
is_atom Ok                  #=> true
is_atom :"Multiple words"   #=> true
is_atom :""                 #=> true
```

### Кортежи

Способ написания кортежей в данных языках одинаковый, но APIs разное. Elixir пытается нормализовать библиотеки Erlang следующим образом:

1. `subject` всегда является первым аргументом функции.
2. Все структуры данных имеют доступ нулевого уровня.

Тем неменее, Elixir по умолчанию не импортирует функции `element` и `setelement`, вместо них доступны функции `elem` и `put_elem`:

**Erlang**

```erlang
element(1, {a, b, c}).       %=> a
setelement(1, {a, b, c}, d). %=> {d, b, c}
```

**Elixir**

```elixir
elem({:a, :b, :c}, 0)         #=> :a
put_elem({:a, :b, :c}, 0, :d) #=> {:d, :b, :c}
```

### Списки и двоичные данные

В Elixir доступен краткий синтаксис для создания двоичных данных:

**Erlang**

```erlang
is_list('Hello').        %=> false
is_list("Hello").        %=> true
is_binary(<<"Hello">>).  %=> true
```

**Elixir**

```elixir
is_list 'Hello'          #=> true
is_binary "Hello"        #=> true
is_binary <<"Hello">>    #=> true
<<"Hello">> === "Hello"  #=> true
```

В Elixir, слово **string** обозначает двоичные данные в кодировке UTF-8 и модуль `String` предназначен для работы с такими данными. Elixir ожидает что все файлы с исходным кодом будут в кодировке UTF-8. С другой стороны, **string** в Erlang является списком символов с которыми работает модуль `:string`, не является UTF-8 и работает приемущественно с списками символов.

Elixir поодерживает многострочную нотацию создания строки (так же известную как *heredocs*):

```elixir
is_binary """
This is a binary
spanning several
lines.
"""
#=> true
```

### Списки ключевых слов

В Elixir доступен offers a literal syntax for creating a list of two-item tuples where the first item in the tuple is an atom and calls them keyword lists:

**Erlang**

```erlang
Proplist = [{another_key, 20}, {key, 10}].
proplists:get_value(another_key, Proplist).
%=> 20
```

**Elixir**

```elixir
kw = [another_key: 20, key: 10]
kw[:another_key]
#=> 20
```

### Maps

Erlang R17 introduced maps, a key-value store, with no ordering. Keys and values can be any term. Creating, updating and matching maps in both languages is shown below:

**Erlang**

```erlang
Map = #{key => 0}.
Updated = Map#{key := 1}.
#{key := Value} = Updated.
Value =:= 1.
%=> true
```

**Elixir**

```elixir
map = %{:key => 0}
map = %{map | :key => 1}
%{:key => value} = map
value === 1
#=> true
```

If the keys are all atoms, Elixir allows developers to use `key: 0` for defining the map as well as using `.key` for accessing fields:

```elixir
map = %{key: 0}
map = %{map | key: 1}
map.key === 1
```

### Регулярные выражения

Elixir supports a literal syntax for regular expressions. Such syntax allows regexes to be compiled at compilation time instead of runtime and does not require you to double escape special regex characters:

**Erlang**

```erlang
{ ok, Pattern } = re:compile("abc\\s").
re:run("abc ", Pattern).
%=> { match, ["abc "] }
```

**Elixir**

```elixir
Regex.run ~r/abc\s/, "abc "
#=> ["abc "]
```

Regexes are also supported in heredocs, which is convenient when defining multiline regexes:

```elixir
Regex.regex? ~r"""
This is a regex
spanning several
lines.
"""
#=> true
```


## Модули

Модули в Erlang распологаются в отдельных файлах и имеют следующую структуру:

```erlang
-module(hello_module).
-export([some_fun/0, some_fun/1]).

% A "Hello world" function
some_fun() ->
  io:format('~s~n', ['Hello world!']).

% This one works only with lists
some_fun(List) when is_list(List) ->
  io:format('~s~n', List).

% Non-exported functions are private
priv() ->
  secret_info.
```

Here we create a module named ``hello_module``. In it we define three functions, the first two are made available for other modules to call via the ``export`` directive at the top. It contains a list of functions, each of which is written in the format ``<function name>/<arity>``. Arity stands for the number of arguments.

An Elixir equivalent to the Erlang above:

```elixir
defmodule HelloModule do
  # A "Hello world" function
  def some_fun do
    IO.puts "Hello world!"
  end

  # This one works only with lists
  def some_fun(list) when is_list(list) do
    IO.inspect list
  end

  # A private function
  defp priv do
    :secret_info
  end
end
```

In Elixir, it is also possible to have multiple modules in one file, as well as nested modules:

```elixir
defmodule HelloModule do
  defmodule Utils do
    def util do
      IO.puts "Utilize"
    end

    defp priv do
      :cant_touch_this
    end
  end

  def dummy do
    :ok
  end
end

defmodule ByeModule do
end

HelloModule.dummy
#=> :ok

HelloModule.Utils.util
#=> "Utilize"

HelloModule.Utils.priv
#=> ** (UndefinedFunctionError) undefined function: HelloModule.Utils.priv/0
```


## Синтаксис функций

[This chapter][3] from the Erlang book provides a detailed description of pattern matching and function syntax in Erlang. Here, I'm briefly covering the main points and provide sample code both in Erlang and Elixir.

[3]: http://learnyousomeerlang.com/syntax-in-functions

### Сопоставление с образцом (Pattern matching)

Pattern matching in Elixir is based on Erlang's implementation and in general is very similar:

**Erlang**

```erlang
loop_through([H | T]) ->
  io:format('~p~n', [H]),
  loop_through(T);

loop_through([]) ->
  ok.
```

**Elixir**

```elixir
def loop_through([h | t]) do
  IO.inspect h
  loop_through t
end

def loop_through([]) do
  :ok
end
```

When defining a function with the same name multiple times, each such definition is called a **clause**. In Erlang, clauses always go side by side and are separated by a semicolon ``;``. The last clause is terminated by a dot ``.``.

Elixir doesn't require punctuation to separate clauses, but they must be grouped together.

### Identifying functions

In both Erlang and Elixir, a function is not identified only by its name, but by its name and arity. In both examples below, we are defining four different functions (all named `sum`, but with different arity):

**Erlang**

```erlang
sum() -> 0.
sum(A) -> A.
sum(A, B) -> A + B.
sum(A, B, C) -> A + B + C.
```

**Elixir**

```elixir
def sum, do: 0
def sum(a), do: a
def sum(a, b), do: a + b
def sum(a, b, c), do: a + b + c
```

Guard expressions provide a concise way to define functions that accept a limited set of values based on some condition.

**Erlang**

```erlang
sum(A, B) when is_integer(A), is_integer(B) ->
  A + B;

sum(A, B) when is_list(A), is_list(B) ->
  A ++ B;

sum(A, B) when is_binary(A), is_binary(B) ->
  <<A/binary,  B/binary>>.

sum(1, 2).
%=> 3

sum([1], [2]).
%=> [1, 2]

sum("a", "b").
%=> "ab"
```

**Elixir**

```elixir
def sum(a, b) when is_integer(a) and is_integer(b) do
  a + b
end

def sum(a, b) when is_list(a) and is_list(b) do
  a ++ b
end

def sum(a, b) when is_binary(a) and is_binary(b) do
  a <> b
end

sum 1, 2
#=> 3

sum [1], [2]
#=> [1, 2]

sum "a", "b"
#=> "ab"
```

### Значение по умолчанию

In addition, Elixir allows for default values for arguments, whereas Erlang does not.

```elixir
def mul_by(x, n \\ 2) do
  x * n
end

mul_by 4, 3 #=> 12
mul_by 4    #=> 8
```

### Анонимные функции

Anonymous functions are defined in the following way:

**Erlang**

```erlang
Sum = fun(A, B) -> A + B end.
Sum(4, 3).
%=> 7

Square = fun(X) -> X * X end.
lists:map(Square, [1, 2, 3, 4]).
%=> [1, 4, 9, 16]
```

**Elixir**

```elixir
sum = fn(a, b) -> a + b end
sum.(4, 3)
#=> 7

square = fn(x) -> x * x end
Enum.map [1, 2, 3, 4], square
#=> [1, 4, 9, 16]
```

It is possible to use pattern matching when defining anonymous functions, too.

**Erlang**

```erlang
F = fun(Tuple = {a, b}) ->
        io:format("All your ~p are belong to us~n", [Tuple]);
        ([]) ->
        "Empty"
    end.

F([]).
%=> "Empty"

F({a, b}).
%=> "All your {a, b} are belong to us"
```

**Elixir**

```elixir
f = fn
      {:a, :b} = tuple ->
        IO.puts "All your #{inspect tuple} are belong to us"
      [] ->
        "Empty"
    end

f.([])
#=> "Empty"

f.({:a, :b})
#=> "All your {:a, :b} are belong to us"
```


### Функции высшего порядка

Anonymous functions are first-class values, so they can be passed as arguments to other functions and also can serve as a return value. There is a special syntax to allow named functions be treated in the same manner.

**Erlang**

```erlang
-module(math).
-export([square/1]).

square(X) -> X * X.

lists:map(fun math:square/1, [1, 2, 3]).
%=> [1, 4, 9]
```

**Elixir**

```elixir
defmodule Math do
  def square(x) do
    x * x
  end
end

Enum.map [1, 2, 3], &Math.square/1
#=> [1, 4, 9]
```


### Partials and function captures in Elixir

Elixir supports partial application of functions which can be used to define anonymous functions in a concise way:

```elixir
Enum.map [1, 2, 3, 4], &(&1 * 2)
#=> [2, 4, 6, 8]

List.foldl [1, 2, 3, 4], 0, &(&1 + &2)
#=> 10
```

We use the same `&` operator to capture a function, allowing us to pass named functions as arguments.

```elixir
defmodule Math do
  def square(x) do
    x * x
  end
end

Enum.map [1, 2, 3], &Math.square/1
#=> [1, 4, 9]
```

The above would be equivalent to Erlang's `fun math:square/1`.

## Управления ходом выполнения

The constructs `if` and `case` are actually expressions in both Erlang and Elixir, but may be used for control flow as in imperative languages.

### Case

The ``case`` construct provides control flow based purely on pattern matching.

**Erlang**

```erlang
case {X, Y} of
  {a, b} -> ok;
  {b, c} -> good;
  Else -> Else
end
```

**Elixir**

```elixir
case {x, y} do
  {:a, :b} -> :ok
  {:b, :c} -> :good
  other -> other
end
```

### If

**Erlang**

```erlang
Test_fun = fun (X) ->
  if X > 10 ->
       greater_than_ten;
     X < 10, X > 0 ->
       less_than_ten_positive;
     X < 0; X =:= 0 ->
       zero_or_negative;
     true ->
       exactly_ten
  end
end.

Test_fun(11).
%=> greater_than_ten

Test_fun(-2).
%=> zero_or_negative

Test_fun(10).
%=> exactly_ten
```

**Elixir**

```elixir
test_fun = fn(x) ->
  cond do
    x > 10 ->
      :greater_than_ten
    x < 10 and x > 0 ->
      :less_than_ten_positive
    x < 0 or x === 0 ->
      :zero_or_negative
    true ->
      :exactly_ten
  end
end

test_fun.(44)
#=> :greater_than_ten

test_fun.(0)
#=> :zero_or_negative

test_fun.(10)
#=> :exactly_ten
```

There are two important differences between Elixir's `cond` and Erlang's `if`:

1) `cond` allows any expression on the left side while Erlang allows only guard clauses;

2) `cond` uses Elixir's concepts of truthy and falsy values (everything is truthy except `nil` and `false`), Erlang's `if` expects strictly a boolean;

Elixir also provides an `if` function that resembles more imperative languages and is useful when you need to check if one clause is true or false:

```elixir
if x > 10 do
  :greater_than_ten
else
  :not_greater_than_ten
end
```

### Отправка и получение сообщений

The syntax for sending and receiving differs only slightly between Erlang and Elixir.

**Erlang**

```erlang
Pid = self().

Pid ! {hello}.

receive
  {hello} -> ok;
  Other -> Other
after
  10 -> timeout
end.
```

**Elixir**

```elixir
pid = Kernel.self

send pid, {:hello}

receive do
  {:hello} -> :ok
  other -> other
after
  10 -> :timeout
end
```


## Добавление Elixir в существующие Erlang программы

Elixir compiles into BEAM byte code (via Erlang Abstract Format). This means that Elixir code can be called from Erlang and vice versa, without the need to write any bindings. All Elixir modules start with the `Elixir.` prefix followed by the regular Elixir name. For example, here is how to use the UTF-8 aware `String` downcase from Elixir in Erlang:

```erlang
-module(bstring).
-export([downcase/1]).

downcase(Bin) ->
  'Elixir.String':downcase(Bin).
```

### Интеграция с Rebar

Если вы используете rebar, вам нужно добавить репозиторий Elixir в список зависемостей:

    https://github.com/elixir-lang/elixir.git

Elixir построен по сохожему с Erlang's OTP принципу. It is divided into applications that are placed inside the `lib` directory, as seen in its [source code repository](https://github.com/elixir-lang/elixir). Since rebar does not recognize such structure, we need to explicitly add to our `rebar.config` which Elixir apps we want to use, for example:

```erlang
{lib_dirs, [
  "deps/elixir/lib"
]}.
```

Этого должно быть достаточно для вызова Elixir функций в коде Erlang. Если вам нужно писать код на Elixir, вы можете [установить Elixir's rebar плагин для автоматической компиляции](https://github.com/yrashk/rebar_elixir_plugin).

### Интеграция вручную

Если вы не используете rebar, самым простым способом использования Elixir кода в текущем Erlang коде, это установка Elixir одним из способов описанных в [руководстве для начинающих](/getting-started/introduction.html), затем добавить директорию `lib` в `ERL_LIBS`.


## Полезно почитать

У языка Erlang's есть очень хорошая документация [на сайте][4] которая содержит подробное описание API и примеры кода. В качестве обучения данные примеры можно переписать на на Elixir. [Erlang cookbook][5] содержит более полезные примеры кода.

У Elixir есть [руквоводство для начинающих][6] и [онлайн документация][7].

[4]: http://www.erlang.org/doc/programming_examples/users_guide.html
[5]: http://schemecookbook.org/Erlang/TOC
[6]: /getting-started/introduction.html
[7]: /docs.html
