---
title: Elixir Documentation
section: docs
layout: default
---

## Документация

На данной странице предоставлена документация по последним версиям языка.

{% assign stable = site.data.elixir-versions[site.data.elixir-versions.stable] %}

<h4 id="stable">Стабильная
  {% if stable.docs_zip == true %}
    <small>(<a href="https://github.com/elixir-lang/elixir/releases/download/v{{ stable.version }}/Docs.zip">Скачать</a>)</small>
  {% endif %}
</h4>

* [Elixir](https://hexdocs.pm/elixir/) - Стандартная библиотека
* [EEx](https://hexdocs.pm/eex/) - Шаблонизатор
* [ExUnit](https://hexdocs.pm/ex_unit/) - Библиотека для тестирования
* [IEx](https://hexdocs.pm/iex/) - Интерактивная оболочка
* [Logger](https://hexdocs.pm/logger/) - Логирование
* [Mix](https://hexdocs.pm/mix/) - Инструменты сборки

#### Мастер ветка

* [Elixir](https://hexdocs.pm/elixir/master/) - Стандартная библиотека
* [EEx](https://hexdocs.pm/eex/master/) - Шаблонизатор
* [ExUnit](https://hexdocs.pm/ex_unit/master/) - Библиотека для тестирования
* [IEx](https://hexdocs.pm/iex/master/) - Интерактивная оболочка
* [Logger](https://hexdocs.pm/logger/master/) - Логирование
* [Mix](https://hexdocs.pm/mix/master/) - Инструменты сборки

{% for version in site.data.elixir-versions reversed %}
  {% if version[0] == 'stable' %}
    {% continue %}
  {% endif %}

<h4 id="{{ version[1].name }}">{{ version[1].name }}
  {% if version[1].docs_zip == true %}<small>(<a href="https://github.com/elixir-lang/elixir/releases/download/v{{ version[1].version }}/Docs.zip">Скачать</a>)</small>{% endif %}
</h4>

* [Elixir](https://hexdocs.pm/elixir/{{ version[1].version }}/) - Стандартная библиотека
* [EEx](https://hexdocs.pm/eex/{{ version[1].version }}/) - Шаблонизатор
* [ExUnit](https://hexdocs.pm/ex_unit/{{ version[1].version }}/) - Библиотека для тестирования
* [IEx](https://hexdocs.pm/iex/{{ version[1].version }}/) - Интерактивная оболочка
* [Logger](https://hexdocs.pm/logger/{{ version[1].version }}/) - Логирование
* [Mix](https://hexdocs.pm/mix/{{ version[1].version }}/) - Инструменты сборки
{% endfor %}
