---
layout: getting-started
title: Comprehensions (Абстракция списков)
---

# {{ page.title }}

{% include toc.html %}

В Elixir, обычной задачей является перебор списков, кортежей и т.д, так же часто возникает необходимость отфильтровать список по какому то критерию, а из данных полученых при этом создать новый список. *Comprehensions* является синтаксическим сахаром позволяя сгруппировать схожие задачи в специальную форму для использования совместно с оператором `for`.

Например, мы можем отобразить список целых чисел в их значения в квадрате:

```iex
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```

Абстракция списков состоит из трех частей: генераторы, фильтры и коллекции.

## Генераторы и фильтры

В выражении выше , `n <- [1, 2, 3, 4]` используются **генераторы**. В данном примере мы буквально генерируем значения которые будут использоватся в *comprehension*. Справа от выражения в котором используется генератор можно использовать любой перечисляемый тип данных:

```iex
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```

В выражениях использующих генераторы можно использовать *pattern matching* в левой части выражения; Все паттерны не являющиеся *pattern matching* игнорируются *ignored*. Представте, что вместо диапозонов, мы используем список ключевых слов, в котором в качестве ключа выступает атом `:good` и `:bad` и мы хотим вычислить только квадрат из значения `:good`:

```iex
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

Альтернативой *pattern matching*, выступают фильтры которые могут быть использованны для получения конкретных значений. Например, мы можем выбрать 3 из них и отказатся от других:

```iex
iex> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
iex> for n <- 0..5, multiple_of_3?.(n), do: n * n
[0, 9]
```

*Comprehensions* отбрасывает все элементы для которых выражение возвращает `false` или `nil`; Все остальные значения остаются выбранными.

Comprehensions generally provide a much more concise representation than using the equivalent functions from the `Enum` and `Stream` modules. Furthermore, comprehensions also allow multiple generators and filters to be given. Here is an example that receives a list of directories and gets the size of each file in those directories:

```elixir
dirs = ['/home/mikey', '/home/james']
for dir  <- dirs,
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  File.stat!(path).size
end
```

Multiple generators can also be used to calculate the cartesian product of two lists:

```iex
iex> for i <- [:a, :b, :c], j <- [1, 2], do:  {i, j}
[a: 1, a: 2, b: 1, b: 2, c: 1, c: 2]
```

A more advanced example of multiple generators and filters is Pythagorean triples. A Pythagorean triple is a set of positive integers such that `a*a + b*b = c*c`, let's write a comprehension in a file named `triple.exs`:

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n,
        b <- 1..n,
        c <- 1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

Now on terminal:

```bash
iex triple.exs
```

```iex
iex> Triple.pythagorean(5)
[]
iex> Triple.pythagorean(12)
[{3, 4, 5}, {4, 3, 5}]
iex> Triple.pythagorean(48)
[{3, 4, 5}, {4, 3, 5}, {5, 12, 13}, {6, 8, 10}, {8, 6, 10}, {8, 15, 17},
 {9, 12, 15}, {12, 5, 13}, {12, 9, 15}, {12, 16, 20}, {15, 8, 17}, {16, 12, 20}]
```

The code above is quite expensive when the range of search is a large number. Additionally, since the tuple `{b, a, c}` represents the same Pythagorean triple as `{a, b, c}`, our function yields duplicate triples. We can optimize the comprehension and eliminate the duplicate results by referencing the variables from previous generators in the following ones, for example:

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n-2,
        b <- a+1..n-1,
        c <- b+1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

Finally, keep in mind that variable assignments inside the comprehension, be it in generators, filters or inside the block, are not reflected outside of the comprehension.

## Генераторы битовых строк

Bitstring generators are also supported and are very useful when you need to comprehend over bitstring streams. The example below receives a list of pixels from a binary with their respective red, green and blue values and converts them into tuples of three elements each:

```iex
iex> pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>
iex> for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
[{213, 45, 132}, {64, 76, 32}, {76, 0, 0}, {234, 32, 15}]
```

A bitstring generator can be mixed with "regular" enumerable generators, and supports filters as well.

## Опция `:into`

In the examples above, all the comprehensions returned lists as their result. However, the result of a comprehension can be inserted into different data structures by passing the `:into` option to the comprehension.

For example, a bitstring generator can be used with the `:into` option in order to easily remove all spaces in a string:

```iex
iex> for <<c <- " hello world ">>, c != ?\s, into: "", do: <<c>>
"helloworld"
```

Sets, maps and other dictionaries can also be given to the `:into` option. In general, `:into` accepts any structure that implements the `Collectable` protocol.

A common use case of `:into` can be transforming values in a map, without touching the keys:

```iex
iex> for {key, val} <- %{"a" => 1, "b" => 2}, into: %{}, do: {key, val * val}
%{"a" => 1, "b" => 4}
```

Let's make another example using streams. Since the `IO` module provides streams (that are both `Enumerable`s and `Collectable`s), an echo terminal that echoes back the upcased version of whatever is typed can be implemented using comprehensions:

```iex
iex> stream = IO.stream(:stdio, :line)
iex> for line <- stream, into: stream do
...>   String.upcase(line) <> "\n"
...> end
```

Now type any string into the terminal and you will see that the same value will be printed in upper-case. Unfortunately, this example also got your IEx shell stuck in the comprehension, so you will need to hit `Ctrl+C` twice to get out of it. :)
