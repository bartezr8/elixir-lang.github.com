---
layout: getting-started
title: Сопоставление с образцом
---

# {{ page.title }}<span hidden>.</span>

{% include toc.html %}

В данном уроке, мы рассмотрим как на самом деле работает оператор `=` в Elixir и как использовать сопоставление с образцом ( pattern match ) в отношении простых значений, структур и даже функций. В заключение, мы рассмотрим работу с "фиксирующим оператором"" `^` который используется для предотвращения перепресвоения переменных.

## Оператор сопоставления

Ранее мы использовали оператор `=` несколько раз для присвоения значений в Elixir:

```iex
iex> x = 1
1
iex> x
1
```

В языке Elixir, оператор `=` на самом деле является *оператором сопоставления* по аналогии со знаком равенства в алгебре. Давайте посмотрим почему:

```iex
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

Обратите внимание что выражение `1 = x` является валидным, и условие соблюдается потому что обе части выражения ( левая и правая ) равны 1. Если стороны не равны, возникнет ошибка `MatchError`.

Переменная используемая для присвоения должна распологаться с лева от оператора `=`:

```iex
iex> 1 = unknown
** (CompileError) iex:1: undefined function unknown/0
```

Since there is no variable `unknown` previously defined, Elixir imagined you were trying to call a function named `unknown/0`, but such a function does not exist.

## Сопоставление с образцом

Оператор сравнения используется для сравнения простых значений, так же он используется для деструктурирующего присваивания. Для примера, воспользуемся сопоставлением с образцом по отношению к кортежу:

```iex
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

Сопоставлением с образцом приводит к ошибке если стороны выражения неравны, в текущем примере кортежи имеют разный размер:

```iex
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

А также если тип данных различается:

```iex
iex> {a, b, c} = [:hello, "world", 42]
** (MatchError) no match of right hand side value: [:hello, "world", 42]
```

More interestingly, we can match on specific values. The example below asserts that the left side will only match the right side when the right side is a tuple that starts with the atom `:ok`:

```iex
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13

iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

Мы можем использовать *pattern match* на списках:

```iex
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

Списки так же поддерживают сопоставление с головой и хвостом:

```iex
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

Подобно функциям `hd/1` и `tl/1`, мы неможем сопостовлять пустой список используя шаблон головы и хоста:

```iex
iex> [h | t] = []
** (MatchError) no match of right hand side value: []
```

Шаблон `[head | tail]` может быть использован, не только для сопоставления с образом, так же его можно использовать для добавления элементов в список:

```iex
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0 | list]
[0, 1, 2, 3]
```

Сопоставление с образцом Pattern matching allows developers to easily destructure data types such as tuples and lists. As we will see in the following chapters, it is one of the foundations of recursion in Elixir and applies to other types as well, like maps and binaries.

## Фиксирующий оператор

Значение переменные в Elixir могут быть перепресвоены:

```iex
iex> x = 1
1
iex> x = 2
2
```

Фиксирующий оператор `^` используется если необходимо выполнить сравнение с текущим значением переменной, а не присвоить ей новое:

```iex
iex> x = 1
1
iex> ^x = 2
** (MatchError) правая сторона выражения не равна значению: 2
iex> {y, ^x} = {2, 1}
{2, 1}
iex> y
2
iex> {y, ^x} = {2, 2}
** (MatchError) правая сторона выражения не равна значению: {2, 2}
```

Поскольку мы присвоили значение 1 переменной x, последний пример можно переписать следующим образом:

```
iex> {y, 1} = {2, 2}
** (MatchError) правая сторона выражения не равна значению: {2, 2}
```
Если переменная упоминается несколько раз в шаблоне, все ссылки должны связываться по таму же шаблону:

```iex
iex> {x, x} = {1, 1}
{1, 1}
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

В некоторых случаях, вам ненужно беспокоится о текущем значении в шаблоне. Обычной в таких случаях связывают значение с `_`. Например, если нам нужна только голова списка, мы можем связать хвост списка с `_`:

```iex
iex> [h | _] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

Переменная `_` является специализированной, значение которой невозможно прочесть. При попытке прочтения значения данной переменной возникнет ошибка:

```iex
iex> _
** (CompileError) iex:1: unbound variable _
```

Хотя сопоставление с образцом позволяет создавать мощные конструкции, его использование ограничено. Например, вы не можете совершать вызовы функций слево от оператора. Следующий пример неверен:

```iex
iex> length([1, [2], 3]) = 3
** (CompileError) iex:1: illegal pattern
```

На этом мы заканчиваем урок по *сопоставлению с образцом*. Как мы увидим в последующих уроках, *сопоставление с образцом* очень часто используется при разработке.