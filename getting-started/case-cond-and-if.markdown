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

Note anonymous functions can also have multiple clauses and guards:

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

The number of arguments in each anonymous function clause needs to be the same, otherwise an error is raised.

```iex
iex> f2 = fn
...>   x, y when x > 0 -> x + y
...>   x, y, z -> x * y + z
...> end
** (CompileError) iex:1: cannot mix clauses with different arities in function definition
```

## `cond`

`case` is useful when you need to match against different values. However, in many circumstances, we want to check different conditions and find the first one that evaluates to true. In such cases, one may use `cond`:

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

This is equivalent to `else if` clauses in many imperative languages (although used way less frequently here).

If none of the conditions return true, an error (`CondClauseError`) is raised. For this reason, it may be necessary to add a final condition, equal to `true`, which will always match:

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

Finally, note `cond` considers any value besides `nil` and `false` to be true:

```iex
iex> cond do
...>   hd([1, 2, 3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

## `if` and `unless`

Besides `case` and `cond`, Elixir also provides the macros `if/2` and `unless/2` which are useful when you need to check for only one condition:

```iex
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

If the condition given to `if/2` returns `false` or `nil`, the body given between `do/end` is not executed and instead it returns `nil`. The opposite happens with `unless/2`.

They also support `else` blocks:

```iex
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```

> Note: An interesting note regarding `if/2` and `unless/2` is that they are implemented as macros in the language; they aren't special language constructs as they would be in many languages. You can check the documentation and the source of `if/2` in [the `Kernel` module docs](https://hexdocs.pm/elixir/Kernel.html). The `Kernel` module is also where operators like `+/2` and functions like `is_function/2` are defined, all automatically imported and available in your code by default.

## `do/end` blocks

At this point, we have learned four control structures: `case`, `cond`, `if`, and `unless`, and they were all wrapped in `do/end` blocks. It happens we could also write `if` as follows:

```iex
iex> if true, do: 1 + 2
3
```

Notice how the example above has a comma between `true` and `do:`, that's because it is using Elixir's regular syntax where each argument is separated by comma. We say this syntax is using *keyword lists*. We can pass `else` using keywords too:

```iex
iex> if false, do: :this, else: :that
:that
```

`do/end` blocks are a syntactic convenience built on top of the keywords one. That's why `do/end` blocks do not require a comma between the previous argument and the block. They are useful exactly because they remove the verbosity when writing blocks of code. These are equivalent:

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

One thing to keep in mind when using `do/end` blocks is they are always bound to the outermost function call. For example, the following expression:

```iex
iex> is_number if true do
...>  1 + 2
...> end
** (CompileError) undefined function: is_number/2
```

Would be parsed as:

```iex
iex> is_number(if true) do
...>  1 + 2
...> end
** (CompileError) undefined function: is_number/2
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
