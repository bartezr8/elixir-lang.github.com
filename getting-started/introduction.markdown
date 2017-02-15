---
layout: getting-started
title: Введение
redirect_from: /getting_started/1.html
---

# {{ page.title }}

{% include toc.html %}

Добро пожаловать!

В ходе данных уроков мы изучим синтаксис Elixir, основные типы данных, работу с модулями и многое другое. This chapter will focus on ensuring Elixir is installed and that you can successfully run Elixir's Interactive Shell, called IEx.

Основыне требования:

  * Elixir - версия 1.4.0 или выше
  * Erlang - версия 18.0 или выше

итак приступим!

> Если вы обнаружили ошибки или неточности на сайте, [напишите об этом или отправте pull реквест с исправлениями](https://github.com/etroynov/elixir-lang.github.com).

## Установка

Если у вас не установлен Elixir, ознакомтесь с [руководством по установке](/install.html). После того как вы установили Elixir, для проверки правильности установки в консоле выполните команду `elixir --version` для получения текущей версии Elixir.

## Интерактивный режим

После установки Elixir, вам станут доступны исполняемые файлы: `iex`, `elixir` и `elixirc`. Если вы собирали Elixir из исходников или через менеджер версий, данные файлы находятся в папке `bin`.

Для начала, выполним в консоле команду `iex` (или `iex.bat` если вы работаете в Windows) для запуска интерактивной обочке Elixir. В интерактивной оболчке, мы можем выполнять код Elixir в реальном времени, незамедлительно получая результат выполнения кода. Давайте попробуем выполнить пару простых операций в интерактивной оболочке.

Перейдем в интерактивную оболочку выполнив команду `iex` в консоле и введем следующий код:

```iex
Erlang/OTP 19 [erts-8.1] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.3.4) - нажмите Ctrl+C для выхода (введите h() нажмите ENTER для получения справки)
iex(1)> 40 + 2
42
iex(2)> "hello" <> " world"
"hello world"
```

Please note that some details like version numbers may differ a bit in your session, that's not important. From now on `iex` sessions will be stripped down to focus on the code. To exit `iex` press `Ctrl+C` twice.

It seems we are ready to go! We will use the interactive shell quite a lot in the next chapters to get a bit more familiar with the language constructs and basic types, starting in the next chapter.

> Примечание: если вы работаете в Windows, рекомендуем запускать оболочку используя команду `iex.bat --werl` для получения расширенных сообщений об ошибках.

## Выполнение скриптов

After getting familiar with the basics of the language you may want to try writing simple programs. This can be accomplished by putting the following Elixir code into a file:

```elixir
IO.puts "Hello world from Elixir"
```

Save it as `simple.exs` and execute it with `elixir`:

```bash
$ elixir simple.exs
Hello world from Elixir
```

Later on we will learn how to compile Elixir code (in [Chapter 8](/getting-started/modules-and-functions.html)) and how to use the Mix build tool (in the [Mix & OTP guide](/getting-started/mix-otp/introduction-to-mix.html)). For now, let's move on to [Chapter 2](/getting-started/basic-types.html).

## Возникли вопросы?

When going through this getting started guide, it is common to have questions, after all, that is part of the learning process! There are many places you could ask them to learn more about Elixir:

  * [#elixir-lang on freenode IRC](irc://irc.freenode.net/elixir-lang)
  * [Elixir on Slack](https://elixir-slackin.herokuapp.com/)
  * [Elixir Forum](http://elixirforum.com)
  * [elixir tag on StackOverflow](https://stackoverflow.com/questions/tagged/elixir)

When asking questions, remember these two tips:

  * Instead of asking "how to do X in Elixir", ask "how to solve Y in Elixir". In other words, don't ask how to implement a particular solution, instead describe the problem at hand. Stating the problem gives more context and less bias for a correct answer.

  * In case things are not working as expected, please include as much information as you can in your report, for example: your Elixir version, the code snippet and the error message alongside the error stacktrace. Use sites like [Gist](https://gist.github.com/) to paste this information.
