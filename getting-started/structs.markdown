---
layout: getting-started
title: Структуры
redirect_from: /getting-started/struct.html
---

# {{ page.title }}

{% include toc.html %}

В [7 уроке](/getting-started/keywords-and-maps.html) мы изучили работу с функцией `map`:

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

Структуры являются надстройкой над *maps* обеспечивают проверку на этапе компиляции и значения по умолчанию.

## Объявление структуры

Для объявления структуры, используется оператор `defstruct`:

```iex
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

Ключевое слово *list* используется с `defstruct` для указания набора полей структуры и присвоения этим полям значений по умолчанию.

В качестве имени структура использует имя модуля в котором она была объявлена. В примере выше, объявленная структура будет иметь имя `User`.

Теперь мы можем создать структуру `User` используя синтаксис похожий на синтаксис который используется для создания maps:

```iex
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

Структура использует механизм *compile-time* который гарантирует что в структуре будут присутствовать только те поля (и *все* из них) которые были объявлены через `defstruct`:

```iex
iex> %User{oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "John"}
```

## Доступ к структурам и их обновление

Когда мы изучали *maps*, мы рассматривали как можно получать данные из них, и как обновлять в них данные. В структурах действует тот же принцип (и синтаксис) для работы с данными:

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "Meg"}
```

Когда для обновления используется (`|`), Erlang <abbr title="Virtual Machine">VM</abbr> гарантирует что в структуру небудут добавлены новые данные, это позволяет оптимально использовать память для хранения структур. В примере выше, значения `john` и `meg` будут хранится единой структурой в памяти.

Структуры можно использовать совместно с pattern matching, для сапоставления значений конкретных ключей, так и для обеспечения соотвествия значений структуры того же типа, что и согласованное значение.

```iex
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## Структуры - это простые maps

В примере выше, *pattern matching* работает потому что на поверку структуры являются стирильными *maps* с фиксированным набором полей. Так же как и у maps, у структур есть специальное поле `__struct__` которое хранит имя данной структуры:

```iex
iex> is_map(john)
true
iex> john.__struct__
User
```

Мы уже сказали что структуры это **стирильные** maps, потому что к ним не применимы функции предназначеные для работы с maps. Например, пвы неможете и получить доступ к данным или итерировать структуру:

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (UndefinedFunctionError) function User.fetch/2 is undefined (User does not implement the Access behaviour)
             User.fetch(%User{age: 27, name: "John"}, :name)
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

Однако, поскольку структуры по факту являются просто *maps* для работы с ними можно использовать функции из модуля `Map`:

```iex
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

Структуры наряду с протоколами обеспечивают одну из наиболее важных функций для разработчиков Elixir: полиморфизм данных. Это мы и рассмотрим в следующей главе.

## Значения по умолчанию и обязательные ключи

Если вы не указали значение ключа по умолчанию во время объявления структуры, в качестве значения будет использовано `nil`:

```iex
iex> defmodule Product do
...>   defstruct [:name]
...> end
iex> %Product{}
%Product{name: nil}
```

Вы можете указать обязательные ключи, которые должны быть обязательно указаны при создании структуры:

```iex
iex> defmodule Car do
...>   @enforce_keys [:make]
...>   defstruct [:model, :make]
...> end
iex> %Car{}
** (ArgumentError) the following keys must also be given when building struct Car: [:make]
    expanding struct: Car.__struct__/1
```
