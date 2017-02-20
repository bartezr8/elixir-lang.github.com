---
layout: getting-started
title: Бинарные данные, строки, символьные списки
---

# {{ page.title }}

{% include toc.html %}

В уроке "Типы", мы познакомились со строками и научились использовать функцию `is_binary/1` для проверки типа данных:

```iex
iex> string = "hello"
"hello"
iex> is_binary(string)
true
```

В данном уроке, мы будем изучать бинарные данные, то как они связанны с строками и чем в Elixir являются данные, в одинарных ковычках.

## UTF-8 и Unicode

Строки кодируются UTF-8 кодировкой. Для того что бы понять что это означает, нам нужно понять в чем разница между байтами и символьными кодами.

Стандарт Unicode свзявает знакомые нам символы алфавита с символьными кодами. Например, буква `a` имеет символьный код `97` а буква `ł` имеет код `322`. Для записи строки `"hełło"` на диск, нам нужно преобразовать символьные коды в байты. Если бы для кодирования одного символьного кода использовался один байт мы бы несмогли написать слово `"hełło"`, потому что для записи одного символа `ł` используется три символа `322`, при этом один байт может хранить только одно число от `0` до `255`. Но, конечно же, поскольку вы можете прочитать `"hełło"` на вашем экране, так как нам нужно это представить *как нибудь*. В этом нам может помочь кодировка.

Для преобразования символьных кодов в байты, нам нужно их как то закодировать. В Elixir используется кодировка UTF-8 по умолчанию. Когда мы говорим что строка представлена в кодировке UTF-8, мы имеем в виду что строка это набор байтов который содержит набор символьных кодов, указанных в кодировке UTF-8.

Так как у нас есть сиволы такие как `ł` имеющий символьный код `322`, нам нужно больше одно байта для кодирования таких символов. В этом и заключается разница в работе функций `byte_size/1` и `String.length/1`, рассмотрим пример:

```iex
iex> string = "hełło"
"hełło"
iex> byte_size(string)
7
iex> String.length(string)
5
```

Где, функция `byte_size/1` считает количество байтов, а функция `String.length/1` считает количество символов.

> Примечание: Если вы работаете в ос Windows, есть вероятность что ваша консоль не поддерживает UTF-8 по умолчанию. Вы можете указать кодировку для текущей сессии выполнив команду `chcp 65001` перед тем как запустить интерактивную оболочку `iex` (`iex.bat`).

Кодировка UTF-8 требует одного байта для кодировки символа `h`, `e`, и `o`, и два байта для кодирования символа `ł`. в Elixir, вы можете получить символ кода воспользовавшись оператором `?`:

```iex
iex> ?a
97
iex> ?ł
322
```

Так же вы можете использовать функцию из [модуля `String`](https://hexdocs.pm/elixir/String.html) для разбиения строки на отдельные символы, длина каждого из них равна 1:

```iex
iex> String.codepoints("hełło")
["h", "e", "ł", "ł", "o"]
```

You will see that Elixir has excellent support for working with strings. It also supports many of the Unicode operations. In fact, Elixir passes all the tests showcased in the article ["The string type is broken"](http://mortoray.com/2013/11/27/the-string-type-is-broken/).

However, strings are just part of the story. If a string is a binary, and we have used the `is_binary/1` function, Elixir must have an underlying type empowering strings. And it does! Let's talk about binaries.

## Бинарные данные (и бинарные строки)

В Elixir, вы можете объявить бинарные данные используя оператор `<<>>`:

```iex
iex> <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> byte_size(<<0, 1, 2, 3>>)
4
```

A binary is a sequence of bytes. Those bytes can be organized in any way, even in a sequence that does not make them a valid string:

```iex
iex> String.valid?(<<239, 191, 191>>)
false
```

The string concatenation operation is actually a binary concatenation operator:

```iex
iex> <<0, 1>> <> <<2, 3>>
<<0, 1, 2, 3>>
```

A common trick in Elixir is to concatenate the null byte `<<0>>` to a string to see its inner binary representation:

```iex
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

Each number given to a binary is meant to represent a byte and therefore must go up to 255. Binaries allow modifiers to be given to store numbers bigger than 255 or to convert a code point to its UTF-8 representation:

```iex
iex> <<255>>
<<255>>
iex> <<256>> # truncated
<<0>>
iex> <<256 :: size(16)>> # use 16 bits (2 bytes) to store the number
<<1, 0>>
iex> <<256 :: utf8>> # the number is a code point
"Ā"
iex> <<256 :: utf8, 0>>
<<196, 128, 0>>
```

If a byte has 8 bits, what happens if we pass a size of 1 bit?

```iex
iex> <<1 :: size(1)>>
<<1::size(1)>>
iex> <<2 :: size(1)>> # truncated
<<0::size(1)>>
iex> is_binary(<<1 :: size(1)>>)
false
iex> is_bitstring(<<1 :: size(1)>>)
true
iex> bit_size(<< 1 :: size(1)>>)
1
```

The value is no longer a binary, but a bitstring -- a bunch of bits! So a binary is a bitstring where the number of bits is divisible by 8.

```iex
iex>  is_binary(<<1 :: size(16)>>)
true
iex>  is_binary(<<1 :: size(15)>>)
false
```

We can also pattern match on binaries / bitstrings:

```iex
iex> <<0, 1, x>> = <<0, 1, 2>>
<<0, 1, 2>>
iex> x
2
iex> <<0, 1, x>> = <<0, 1, 2, 3>>
** (MatchError) no match of right hand side value: <<0, 1, 2, 3>>
```

Note each entry in the binary pattern is expected to match exactly 8 bits. If we want to match on a binary of unknown size, it is possible by using the binary modifier at the end of the pattern:

```iex
iex> <<0, 1, x :: binary>> = <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> x
<<2, 3>>
```

Similar results can be achieved with the string concatenation operator `<>`:

```iex
iex> "he" <> rest = "hello"
"hello"
iex> rest
"llo"
```

A complete reference about the binary / bitstring constructor `<<>>` can be found [in the Elixir documentation](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#%3C%3C%3E%3E/1). This concludes our tour of bitstrings, binaries and strings. A string is a UTF-8 encoded binary and a binary is a bitstring where the number of bits is divisible by 8. Although this shows the flexibility Elixir provides for working with bits and bytes, 99% of the time you will be working with binaries and using the `is_binary/1` and `byte_size/1` functions.

## Список символов

Список символов это ничто иное как список символьных кодов. Список символов можно создать используя литерал строки в одинарных ковычках:

```iex
iex> 'hełło'
[104, 101, 322, 322, 111]
iex> is_list 'hełło'
true
iex> 'hello'
'hello'
iex> List.first('hello')
104
```

You can see that, instead of containing bytes, a char list contains the code points of the characters between single-quotes (note that by default IEx will only output code points if any of the integers is outside the ASCII range). So while double-quotes represent a string (i.e. a binary), single-quotes represent a char list (i.e. a list).

In practice, char lists are used mostly when interfacing with Erlang, in particular old libraries that do not accept binaries as arguments. You can convert a char list to a string and back by using the `to_string/1` and `to_charlist/1` functions:

```iex
iex> to_charlist "hełło"
[104, 101, 322, 322, 111]
iex> to_string 'hełło'
"hełło"
iex> to_string :hello
"hello"
iex> to_string 1
"1"
```

Note that those functions are polymorphic. They not only convert char lists to strings, but also integers to strings, atoms to strings, and so on.

With binaries, strings, and char lists out of the way, it is time to talk about key-value data structures.
