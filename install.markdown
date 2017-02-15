---
title: "Installing Elixir"
section: install
layout: default
---
{% assign stable = site.data.elixir-versions[site.data.elixir-versions.stable] %}

# {{ page.title }}

{% include toc.html %}

Самый быстрый способ установить Elixir это воспользоватся достпуными установщиками. Если таковых нету, мы рекомендуем предварительно собранные пакеты или вы можете самостоятельно скомпилировать нужную вам версию.

Для установки Elixir требуется версия Erlang 18.0 или выше. Большинство инсталяторов сами установят нужную версию Erlang. В данном случае, можно не читать раздел “Установка Erlang”.

## Дистрибутив

Рекомендуется устанавливать Elixir используя системные мереджеры пакетов.

Если в вашей версии содержится устаревшая версия Elixir/Erlang, смотрите раздел "установка Elixir/Erlang через мереджер версий или сборка из исходников".

### Mac OS X

  * Homebrew
    * Сначала обновите Homebrew: `brew update`
    * В консоли выполните: `brew install elixir`
  * Macports
    * В консоли выполните: `sudo port install elixir`

### Unix (and Unix-like)

  * Arch Linux (Репозиторий сообщества)
    * В консоли выполните: `pacman -S elixir`
  * openSUSE (and SLES 11 SP3+)
    * Подключите девелоперский репозиторий: `zypper ar -f http://download.opensuse.org/repositories/devel:/languages:/erlang/openSUSE_Factory/ erlang`
    * В консоли выполните: `zypper in elixir`
  * Gentoo
    * В консоли выполните: `emerge --ask dev-lang/elixir`
  * GNU Guix
    * В консоли выполните: `guix package -i elixir`
  * Fedora 21 (and older)
    * В консоли выполните: `yum install elixir`
  * Fedora 22 (and newer)
    * В консоли выполните `dnf install elixir`
  * FreeBSD
    * From ports: `cd /usr/ports/lang/elixir && make install clean`
    * From pkg: `pkg install elixir`
  * Ubuntu 12.04/14.04/16.04 or Debian 7
    * Add Erlang Solutions repo: `wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb && sudo dpkg -i erlang-solutions_1.0_all.deb`
    * В консоли выполните: `sudo apt-get update`
    * Установка платформы Erlang/OTP и ее зависемостей: `sudo apt-get install esl-erlang`
    * Установка Elixir: `sudo apt-get install elixir`

### Windows

  * Установочный пакет
    * [Скачайте установочный пакет](https://repo.hex.pm/elixir-websetup.exe)
    * Жмем далее, далее, ..., завершить
  * Chocolatey
    * `cinst elixir`

### Raspberry Pi

Заметите “jessie” на название вашего релиза.
  
  * Репозиторий Erlang содержит пердварительно скомпилированную версию armhf.
    Это сэкономит времня которое вы потратите на сборку.
  
  * Get Erlang key   
  
    * `echo "deb http://packages.erlang-solutions.com/debian jessie contrib" | sudo tee /etc/apt/sources.list.d/erlang-solutions.list`    
    * В консоли выполните: `wget http://packages.erlang-solutions.com/debian/erlang_solutions.asc`  
    * Добавление ключа в связку:: `sudo apt-key add erlang_solutions.asc`   
     
  * Install Elixir
    * Обновляем состояние репозитория: `sudo apt update`     
    * В консоли выполните: `sudo apt install elixir`   

### Docker

  Если у вас есть опыт работы с Docker вы можете воспользоватся готовым образом для того что бы сразу начать работать с Elixir.

  * Enter interactive mode
    * В консоли выполните: `docker В консоли выполните -it --rm elixir`
  * Enter bash within container with installed `elixir`
    * В консоли выполните: `docker В консоли выполните -it --rm elixir bash`

Так же автоматически будет установлен Erlang. Если этого не произошло, смотрите раздел [Установка Erlang](/install.html#installing-erlang).

## Предварительно собранные пакеты

Elixir предоставляет предварительно собранные пакеты для каждой версии. Сначала  [Установите Erlang](/install.html#installing-erlang) затем скачайте zip архив [Precompiled.zip необходимой версии](https://github.com/elixir-lang/elixir/releases/download/v{{ stable.version }}/Precompiled.zip).

После распаковки архива, вам будут доступны исполняемые файлы `elixir` и `iex` которые находятся в папке `bin`, но мы рекомендуем вам [добавить путь к исполняемым файлам в переменную PATH ](#setting-path-environment-variable) для упращения дальнейшей работы с ними.

## Установка через менеджер контроля версий

Существует множество инструментов для установки и контроля версий Erlang и Elixir. Они могут быть полезны если у вас неполучилось установить Elang/Elixir штатными средствами вашего дистрибутива или если в репозитории вашего дистрибутива находится устаревшая версия. Вот некоторые из них:

  * [asdf](https://github.com/asdf-vm/asdf) -  позволяет устанавливать и управлять версиями Elixir и Erlang
  * [exenv](https://github.com/mururu/exenv) -  позволяет устанавливать и управлять версиями Elixir
  * [kiex](https://github.com/taylor/kiex) -  позволяет устанавливать и управлять версиями Elixir
  * [kerl](https://github.com/yrashk/kerl) -  позволяет устанавливать и управлять версиями Erlang

## Ручная сборка (Unix и MinGW)

Для ручной сборки Elixir вам нужно выполнить несколько шагов. Для начала вам нужно [Установка Erlang](/install.html#installing-erlang).

Затем скачиваете исходники ([.zip](https://github.com/elixir-lang/elixir/archive/v{{ stable.version }}.zip), [.tar.gz](https://github.com/elixir-lang/elixir/archive/v{{ stable.version }}.tar.gz)) [последней версии](https://github.com/elixir-lang/elixir/releases/tag/v{{ stable.version }}), разархивираете его и выполняете команду `make` внутри папки с разархивированной папки (Примечание: если вы работаете в Windows, [ознакомтись с особенностями компиляции Elixir в среде Windows](https://github.com/elixir-lang/elixir/wiki/Windows)).

После того как Elixir будет скомпилирован, вам будут доступны исполняемые файлы elixir и `iex` распологающиеся в папке `bin`.Так же рекомендуем дабавить путь к папке с Elixir [в переменную PATH](#setting-path-environment-variable) для упрощения разработки.

Если вам нужны фичи недоступные в последнем релизе но доступные в мастер ветке, вы можете собрать Elixir из мастер ветки, для этого в консоли выполните следующие команды:

```bash
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```

Если тесты завершились без ошшибок, вы можете приступать к сборке. В противном случае, ознакомтесь с [списком возможных проблем на Github](https://github.com/elixir-lang/elixir).

## Установка Erlang

Для установки Elixir необходима версия Erlang 18.0 или выше, которую можно легоко установить из [предварительно собранных пакетов.](https://www.erlang-solutions.com/resources/download.html). В случае если вы хотите собрать Erlang вручную, все необходимые файлы вы можете скачать на [официальном сайте Erlang](http://www.erlang.org/download.html){: rel="nofollow"} или следуя отличному руководству доступному по адресу  [https://docs.basho.com/riak/latest/ops/building/installing/erlang/](https://docs.basho.com/riak/latest/ops/building/installing/erlang/).

For Windows developers, we recommend the precompiled packages. Those on a Unix platform can probably get Erlang installed via one of the many package distribution tools.

After Erlang is installed, you should be able to open up the command line (or command prompt) and check the Erlang version by typing `erl`. You will see some information as follows:

    Erlang/OTP 18 (erts-7) [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]

Notice that depending on how you installed Erlang, Erlang binaries might not be available in your PATH. Be sure to have Erlang binaries in your [PATH](https://en.wikipedia.org/wiki/Environment_variable), otherwise Elixir won't work!

## Настройка переменной PATH

Для упращения разработки рекомендуется прописать путь до исполняемых файлов Elixir’s, в переменную PATH.

На **Windows**, следуйте инструкциям [из руководства](http://www.computerhope.com/issues/ch000549.htm) по установке.

На **системах Unix**, вам необходимо [найти файл настроек профиля ( shell )](https://unix.stackexchange.com/a/117470/101951) и добавить в конец файла путь до директории в которой установлен Elixir:

```bash
export PATH="$PATH:/path/to/elixir/bin"
```

## Проверка установленной версии Elixir

После того как Elixir будет установлен, вы можете проверить его версию выполнив команду `elixir --version`.
