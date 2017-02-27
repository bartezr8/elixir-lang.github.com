---
layout: getting-started
title: Использование библиотек Erlang
---

# {{ page.title }}

{% include toc.html %}

Elixir обеспечивает отличную совместемость с библиотеками Erlang. По факту, Elixir предлагает работать с кодом Erlang напряму вместо создания оберток над библиотеками Erlang. В данном уроке мы разберем самые популярные и наиболее часто используемые фичи Erlang используемые в Elixir.

По мере роста опыта в работе с Elixir, вам возможно захочется изучить Erlang, вся основная документация по Erlang сосреготочена в [руководстве по STDLIB](http://erlang.org/doc/apps/stdlib/index.html).

## Двоичные модули

Встроенный модуль работы с строками в Elixir работает с кодировкой UTF-8. [Двоичный модуль](http://erlang.org/doc/man/binary.html) полезен когда у вас возникает необходимость работы с двоичными данными которые не обязательно в кодировке UTF-8.

```iex
iex> String.to_charlist "Ø"
[216]
iex> :binary.bin_to_list "Ø"
[195, 152]
```

В примере выше видна разница при использовании этих модулей; Модуль `String` возвращает список юникод символов, в то время как модуль `:binary` возвращает последовательность байтов.

## Форматирование вывода

В Elixir нет аналога функции `printf`, которая используется в C и других языках. К счастью, в стандартной библиотеке Erlang есть функция `:io.format/2` и
`:io_lib.format/2` которые мы можем использовать. Первая отвечает за форматирование данных при выводе в терминал, в то время как вторая форматирует данные для вывода в iolist. Параметры форматироватирования отличаются от `printf`, [см документацию по Erlang](http://erlang.org/doc/man/io.html#format-1).

```iex
iex> :io.format("Число Pi равно:~10.3f~n", [:math.pi])
Число Pi равно:     3.142
:ok
iex> to_string :io_lib.format("Число Pi равно:~10.3f~n", [:math.pi])
"Число Pi равно:     3.142\n"
```

Учтите использование функций форматирования Erlang's требует особого внимания при работе с символами Unicode.

## Модуль шифрования

[Модуль crypto](http://erlang.org/doc/man/crypto.html) содержит функции для хэширования, цифровой подписи, шифрования и много другого:

```iex
iex> Base.encode16(:crypto.hash(:sha256, "Elixir"))
"3315715A7A3AD57428298676C5AE465DADA38D951BDFAC9348A8A31E9C7401CB"
```

Модуль `:crypto` не является частью стандартной библиотеки Erlang, но входит в в пакет языка Erlang. Это значит что вам нужно добавить модуль `:crypto` в список зависимостей проекта. Для этого, отредактируйте файл `mix.exs` добавив в него:

```elixir
def application do
  [extra_applications: [:crypto]]
end
```

## Модуль графов

[модуль графов](http://erlang.org/doc/man/digraph.html) (так же как и [digraph_utils](http://erlang.org/doc/man/digraph_utils.html)) содержит функции для работы с ориентированными графами состоящими из вершин и ребер.

После построения графа, функции из модуля помогут найти кратчайший путь между двумя вершинами, петлями в графе.

Если передать 3 ребра, алгоритм найдет кратчайший путь от первого ребра к последнему.

```iex
iex> digraph = :digraph.new()
iex> coords = [{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
iex> [v0, v1, v2] = (for c <- coords, do: :digraph.add_vertex(digraph, c))
iex> :digraph.add_edge(digraph, v0, v1)
iex> :digraph.add_edge(digraph, v1, v2)
iex> :digraph.get_short_path(digraph, v0, v2)
[{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
```

Обратите внимание что функции `:digraph` изменяют структура графа на месте, это возможно благодаря тому что они выполнены в виде ETS таблиц, подробности далее.

## Erlang Term хранилище

Модуль [`ets`](http://erlang.org/doc/man/ets.html) и [`dets`](http://erlang.org/doc/man/dets.html) используются для хранения большого объема данных в памяти или на диске.

ETS позволяет создавать таблицы состоящие из кортежей. По умолчанию, ETS таблицы защищены, это означает что только владелец процесса может писать в таблицу однако други процессы могут её читать. ETS имеет функционал схожий с простой базой данных, используемой в качестве хранения пар ключ-значение или для реализации механизма кэширования.

Побочным эффектом использования функций модуля `ets` служит изменение состояния таблицы.

```iex
iex> table = :ets.new(:ets_test, [])
# Store as tuples with {name, population}
iex> :ets.insert(table, {"China", 1_374_000_000})
iex> :ets.insert(table, {"India", 1_284_000_000})
iex> :ets.insert(table, {"USA", 322_000_000})
iex> :ets.i(table)
<1   > {<<"India">>,1284000000}
<2   > {<<"USA">>,322000000}
<3   > {<<"China">>,1374000000}
```

## `Math` модуль

[модуль `math`](http://erlang.org/doc/man/math.html) содержит функции для выполнения основных математических, тригонометрических, экспоненциальных и логарифмитических операций.

```iex
iex> angle_45_deg = :math.pi() * 45.0 / 180.0
iex> :math.sin(angle_45_deg)
0.7071067811865475
iex> :math.exp(55.0)
7.694785265142018e23
iex> :math.log(7.694785265142018e23)
55.0
```

## Модуль очередей

[Структура данных `queue`](http://erlang.org/doc/man/queue.html) это эффективная реализация (симметричной) FIFO (метод "первым пришел - первым вышел") очереди:

```iex
iex> q = :queue.new
iex> q = :queue.in("A", q)
iex> q = :queue.in("B", q)
iex> {value, q} = :queue.out(q)
iex> value
{:value, "A"}
iex> {value, q} = :queue.out(q)
iex> value
{:value, "B"}
iex> {value, q} = :queue.out(q)
iex> value
:empty
```

## Модуль `rand`

Модуль [`rand`](http://erlang.org/doc/man/rand.html) возвращает рандомное значение и принимает значение для установки рандомности значения.

```iex
iex> :rand.uniform()
0.8175669086010815
iex> _ = :rand.seed(:exs1024, {123, 123534, 345345})
iex> :rand.uniform()
0.5820506340260994
iex> :rand.uniform(6)
6
```

## Модуль zip и zlib

[Модуль `zip`](http://erlang.org/doc/man/zip.html) позволяет читать и записывать ZIP файлы с диска и памяти, а так же для получения информации о файле.

Этот код считает количество файлов в ZIP архиве:

```iex
iex> :zip.foldl(fn _, _, _, acc -> acc + 1 end, 0, :binary.bin_to_list("file.zip"))
{:ok, 633}
```

[Модуль `zlib`](http://erlang.org/doc/man/zlib.html) работает с файлами в формате *zlib*, аналогичнен команде `gzip`.

```iex
iex> song = "
...> Mary had a little lamb,
...> His fleece was white as snow,
...> And everywhere that Mary went,
...> The lamb was sure to go."
iex> compressed = :zlib.compress(song)
iex> byte_size song
110
iex> byte_size compressed
99
iex> :zlib.uncompress(compressed)
"\nMary had a little lamb,\nHis fleece was white as snow,\nAnd everywhere that Mary went,\nThe lamb was sure to go."
```
