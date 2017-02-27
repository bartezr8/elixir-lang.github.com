---
layout: getting-started
title: Where to go next
---

# {{ page.title }}

{% include toc.html %}

Готовы узнать больше? Продолжай читать!

## Создаем ваш первый Elixir проект

Для создания нашего первого проекта, мы воспользуемся `Mix` это система сборки входящая в состав Elixir. Для создания шаблона проекта в консоли наберите:

```bash
$ mix new path/to/new/project
```

Мы написали руководство по написанию простого приложения на Elixir включая контроль кода, конфигурацию, тесты и многое другое. Приложение будет выполнять роль хранилища пар ключ-значение, хранилища будут находится на отдельных нодах:

* [Mix и OTP](/getting-started/mix-otp/introduction-to-mix.html)

## Метапрограммирование

Elixir is an extensible and very customizable programming language thanks to its meta-programming support. Most meta-programming in Elixir is done through macros, which are very useful in several situations, especially for writing DSLs. We have written a short guide that explains the basic mechanisms behind macros, shows how to write macros, and how to use macros to create DSLs:

* [Метапрограммирование в Elixir](/getting-started/meta/quote-and-unquote.html)

## Сообщество и другие ресурсы

We have a [Learning](/learning.html) section that suggests books, screencasts, and other resources for learning Elixir and exploring the ecosystem. There are also plenty of Elixir resources out there, like conference talks, open source projects, and other learning material produced by the community.

Don't forget that you can also check the [source code of Elixir itself](https://github.com/elixir-lang/elixir), which is mostly written in Elixir (mainly the `lib` directory), or [explore Elixir's documentation](/docs.html).

## Взаимодействие с Erlang

Elixir работает поверх виртуальной машины Erlang и, рано или поздно, Elixir разработчику понадобится механизм для работы с библиотеками Erlang. Мы собрали для вас список онлайн ресурсов по Erlang's которые содержат много полезной информации:

* This [Синтаксис Erlang: Экстремальный курс](/crash-course.html) provides a concise intro to Erlang's syntax. Each code snippet is accompanied by equivalent code in Elixir. This is an opportunity for you to not only get some exposure to Erlang's syntax but also review some of the things you have learned in this guide.

* На официальном сайте по Erlang's есть краткое [руководство](http://www.erlang.org/course/concurrent_programming.html) с иллюстрациями по Erlang's и основам параллельного программирования.

* [Изучай Erlang во имя добра!](http://learnyousomeerlang.com/) отличное пособие по изучению Erlang, принципах проектирования, стандартной библиотеки, лучших практиках, и моногому другому. Once you have read through the crash course mentioned above, you'll be able to safely skip the first couple of chapters in the book that mostly deal with the syntax. When you reach [руководство по параллельному программированию от Hitchhiker's Guide](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency) chapter, that's where the real fun starts.
