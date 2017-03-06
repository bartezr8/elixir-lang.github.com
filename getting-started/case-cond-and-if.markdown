---
layout: getting-started
title: Операторы case, cond, и if
---

# {{ page.title }}

{% include toc.html %}

В данном уроке, мы изучим работу с операторами `case`, `cond`, и `if` которые используются для создания условий в программе.

## `case`

Оператор `case` позволяет проводить сравнение передаваемого значения с несколькими условиями, до тех пор пока не найдет соотвествие:

```iex
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "Данное условие не выполняется"
...>   {1, x, 3} ->
...>     "Данное условие выполняется благодаря оператору x"
...>   _ ->
...>     "Данное условие соотвествует любому значению"
...> end
```

Для того что бы использовать в качестве условия существующую переменную, используют оператор `^`:

```iex
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Не соотвествует условию"
...>   _  -> "Соотвествует условию"
...> end
"Соотвествует условию"
```

Условия можно расширять испольщуя операторы:

```iex
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Соотвествует условию"
...>   _ ->
...>     "Данный блок выполняется если значение не соотвествует ни одному условию"
...> end
"Соотвествует условию"
```

Первое условие соблюдается если `x` является положительным числом.

## Выражения используемые в качестве условий

В Elixir можно использовать следующие конструкции в качестве условий:

* операторы сравнения (`==`, `!=`, `===`, `!==`, `>`, `>=`, `<`, `<=`)
* булевы операторы (`and`, `or`, `not`)
* арифмитические операторы (`+`, `-`, `*`, `/`)
* унарные арифмитические операторы (`+`, `-`)
* оператор объеденения двоичных данных `<>`
* оператор `in` в том случае если левой частью является список или диапозон
* Все функции проверки типа данных:
    * `is_atom/1`
    * `is_binary/1`
    * `is_bitstring/1`
    * `is_boolean/1`
    * `is_float/1`
    * `is_function/1`
    * `is_function/2`
    * `is_integer/1`
    * `is_list/1`
    * `is_map/1`
    * `is_nil/1`
    * `is_number/1`
    * `is_pid/1`
    * `is_port/1`
    * `is_reference/1`
    * `is_tuple/1`
* и в добавок:
    * `abs(number)`
    * `binary_part(binary, start, length)`
    * `bit_size(bitstring)`
    * `byte_size(bitstring)`
    * `div(integer, integer)`
    * `elem(tuple, n)`
    * `hd(list)`
    * `length(list)`
    * `map_size(map)`
    * `node()`
    * `node(pid | ref | port)`
    * `rem(integer, integer)`
    * `round(number)`
    * `self()`
    * `tl(list)`
    * `trunc(number)`
    * `tuple_size(tuple)`

В дополнение к существующим условиям, пользователи могут создавать свои "условия". Например, модуль `Bitwise`
позволяет в качестве условий использовать операторы и функции: `bnot`, `~~~`, `band`,
`&&&`, `bor`, `|||`, `bxor`, `^^^`, `bsl`, `<<<`, `bsr`, `>>>`.

Обратите внимание что в качестве условий могут выступать булевы операторы `and`, `or`, `not`, однако обычные операторы `&&`, `||`, и `!` не могут быть использованы.

Если условие содержит ошибку, программа не завершится ошибкой, вместо этого условие всегда будет ложным:

```iex
iex> hd(1)
** (ArgumentError) argument error
iex> case 1 do
...>   x when hd(x) -> "Не соотвествует условию"
...>   x -> "Got #{x}"
...> end
"Got 1"
```

Если выражение не соотвествует ни одному условия то возникнет ошибка:

```iex
iex> case :ok do
...>   :error -> "Не соотвествует условию"
...> end
** (CaseClauseError) no case clause matching: :ok
```

Обратите внимание что у анонимных функций может быть несколько условий и *guards*:

```iex
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```

Количество аргументов в каждой анонимной функции должно быть одинаковым, иначе возникнет ошибка.

```iex
iex> f2 = fn
...>   x, y when x > 0 -> x + y
...>   x, y, z -> x * y + z
...> end
** (CompileError) iex:1: cannot mix clauses with different arities in function definition
```

## `cond`

Оператор `case` является полезным когда вам нужно проверить несколько условий одновременно. Тем неменее, в большинстве случаев, возникает необходимость проверки различных условий и найти первое совпавшее. Для таких случаев, мы можем использовать оператор `cond`:

```iex
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

Данная конструкция аналогична конструкции `else if`, которая часто используется в других языках программирования (однако не часто используется в данном языке).

Если небудет найдено значение соотвествующие условию, возникнет ошибка (`CondClauseError`). По этой причине, необходимо добавить заключительное условие, эквивалентное `true`, которое всегда будет выполнятся:

```iex
iex> cond do
...>   2 + 2 == 5 ->
...>     "This is never true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   true ->
...>     "This is always true (equivalent to else)"
...> end
"This is always true (equivalent to else)"
```

И последнее, обратите внимание оператор `cond` считает любое значение истиным кроме `nil` и `false`:

```iex
iex> cond do
...>   hd([1, 2, 3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

## `if` и `unless`

Помимо `case` и `cond`, в Elixir доступны макросы `if/2` и `unless/2` которые могут быть полезны когда вам нужно проверить только одно условие:

```iex
iex> if true do
...>   "Это работает!"
...> end
"Это работает!"
iex> unless true do
...>   "Это никогда не выведется"
...> end
nil
```

Если условие переданное в `if/2` возвращает `false` или `nil`, содержимое между `do/end` не выполнится и в качестве результата будет возвращено `nil`. Оператор `unless/2` является противоположным `if\2`.

Так же можно использовать `else`:

```iex
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```

> Примечание: Интересная особенность касается `if/2` и `unless/2` которые в Elixir реализованны как макросы; Они не являются специальными языковыми конструкциями, какими они являются в других языках. Вы можете убедится в этом, изучив документацию и исходный код макроса `if/2` и [документацию по модулю `Kernel`](https://hexdocs.pm/elixir/Kernel.html). В модуле `Kernel` объявлены такие операторы как `+/2` и функции подобные `is_function/2`, все они импортируются автоматически и доступны по умолчанию.

## Блоки `do/end`

Мы уже изучили, четыре управляющие конструкции: `case`, `cond`, `if`, и `unless`, и все они в качестве обертки используют блоки `do/end`. Используя их мы можем записать инструкцию `if` следующим образом:

```iex
iex> if true, do: 1 + 2
3
```

Обратите внимание на запятую между `true` и `do:`, она там используется потому что в Elixir's каждый аргумент разделяется запятой. Мы сказали что данный синтаксис использует *список ключевых слов*. Мы можем передать `else` используя ключевое слово:

```iex
iex> if false, do: :this, else: :that
:that
```

Блок `do/end` blocks are a syntactic convenience built on top of the keywords one. That's why `do/end` blocks do not require a comma between the previous argument and the block. They are useful exactly because they remove the verbosity when writing blocks of code. These are equivalent:

```iex
iex> if true do
...>   a = 1 + 2
...>   a + 10
...> end
13
iex> if true, do: (
...>   a = 1 + 2
...>   a + 10
...> )
13
```

Запомните одну вещь, используя блок `do/end` всегда связаны с вызовом стороней функции. Например, следующее выражение:

```iex
iex> is_number if true do
...>  1 + 2
...> end
** (CompileError) неизвестная функция: is_number/2
```

Будет преобрзованно в:

```iex
iex> is_number(if true) do
...>  1 + 2
...> end
** (CompileError) неизвестная функция: is_number/2
```

which leads to an undefined function error because that invocation passes two arguments, and `is_number/2` does not exist. The `if true` expression is invalid in itself because it needs the block, but since the arity of `is_number/2` does not match, Elixir does not even reach its evaluation.

Adding explicit parentheses is enough to bind the block to `if`:

```iex
iex> is_number(if true do
...>  1 + 2
...> end)
true
```

Keyword lists play an important role in the language and are quite common in many functions and macros. We will explore them a bit more in a future chapter. Now it is time to talk about "Binaries, strings, and char lists".
