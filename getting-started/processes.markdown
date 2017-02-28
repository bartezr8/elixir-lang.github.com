---
layout: getting-started
title: Процессы
---

# {{ page.title }}

{% include toc.html %}

В Elixir, всь код запускается внутри процессов. Процессы изолированны друг от друга, работают паралелльно друг другу и общаются между собой посредством сообщений. Процессы используются не только для реализации параллелизма в Elixir, но так же они являются отличным средством для построения паралелльных отказоустойчивых систем.

Процессы в Elixir's не используют процессы операционной системы. Процессы в Elixir являются легковесными с точки зрения потребления оперативной памяти и ресурсов процессора (в отличии от потоков в других языках программирования). Благодаря этому возможно на одной машине одновременно запускать десятки а то и сотни тысяч процессов одновременно.

В данном уроке, мы научимся создавать новые процессы, а так же пересылать сообщения между ними.

## `spawn`

Основным механизмом для создания нового процесса служит автоматически импортируемая функция `spawn/1`:

```iex
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

`spawn/1` в качестве аргумента принимает функцию, которая будет выполнятся в отдельном процессе.

Обратите внимание что в качестве результата функция `spawn/1` возвращает PID (идентификатор процесса). При этом порожденный процесс вероятнее всего быстро станет мертвым. Созданный нами процесс начнет выполнение переданной функции и завершится после того как она будет выполнена:

```iex
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

> Примечание: скорее всего ваш идентификатор будет отличатся от идентификатора в этом примере.

Вы можете получить PID текущего процесса используя функцию `self/0`:

```iex
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

Процессы становятся намного более интересными, когда мы в состоянии получать и передавать сообщения между ними.

## `send` и `receive`

Мы можем отправить сообщение процессу используя функцию `send/2` и получить его с помощью функции `receive/1`:

```iex
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

Когда сообщение отправляют процессу, оно хранитсся в почтовом ящике процесса. Функция `receive/1` блокирует поток и выполняет поиск сообщения в почтовом ящике, по переданному ей шаблону. Функция `receive/1` может использовать *guards* и несколько возможных вариантов, по аналогии с функцией `case/2`.

Процесс который отправляет сообщение с помощью функции `send/2` не блокируется, после того как сообщение будет доставлено в почтовый ящик, выполнение продолжится. На практике, процесс может послать сообщение самому себе.

Если в почтовом ящике не будет сообщений удолетворяющих шаблону, процесс будет ожить сообщение то тех пор пока оно не придет, при необходимости мы можем задать интервал ожидания:

```iex
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

Таймаут равный 0 можно указать если вы уже ожидаете доставку сообщений в почтовый ящик.

Давайте воспользуемся получеными знаниями и попробуем осуществить пересылку между процессами:

```iex
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

Функция наблюдатель используется для преобразовния, внутрених данных в строку, как правило для вывода на печать. Обратите внимание, что к тому моменту как блок `receive` будет выполнен, процес может быть уже завершен, поскольку его единственной задачей является отправка сообщения.

При работе в оболочке, полезной будет функция `flush/0`. Она очищает почтовый ящик и выводит все собщения из него на печать.

```iex
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## Ссылки

Большую часть времени в Elixir мы будем создавать процессы, созданные процессы по большей части будут связанными. Перед тем как мы рассмотрим пример с использованием функции `spawn_link/1`, давайте посмотрим что произойдет когда в процессе созданном функцией `spawn/1` возникает ошибка:

```iex
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Process #PID<0.58.00> raised an exception
** (RuntimeError) oops
    :erlang.apply/2
```

Он просто выводит ошибку при этом родительский процесс продолжает работать. Это потому что процесс является изолированным. Если мы хотим что бы ошибка в одном процессе, возникала в другом процессе, нам нужно их связать. Это можно сделать воспользовавшись функцией `spawn_link/1`:

```iex
iex> spawn_link fn -> raise "oops" end
#PID<0.41.0>

** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

Поскольку процессы связаны между собой, мы можем видеть сообщения от родительского процесса, поскольку это процесс выполняется в интерактивной оболочке, при получении сигнала EXIT от другого процесса оболочка завершит свою работу. IEx видит данную ситуацию и запускает оболочку с новой сессией.

Процессы можно связать вручную используя функцию `Process.link/1`. Мы рекомендуем вам почитать документацию по [модулю `Process`](https://hexdocs.pm/elixir/Process.html) для ознакомления с другими возможностями данного модуля.

Процесс и связи между ними играют важную роль в построении отказоустойчивых систем. В Elixir процессы являются изолированными и по умолчанию ничего не разделяют. Таким образом, сбой в одном процессе не влияет на другие процессы. Ссылки, однако, позволяют устанавливать связь после сбоя. Часто связывание процессов используется We often link our processes to supervisors which will detect when a process dies and start a new process in its place.

While other languages would require us to catch/handle exceptions, in Elixir we are actually fine with letting processes fail because we expect supervisors to properly restart our systems. "Failing fast" is a common philosophy when writing Elixir software!

`spawn/1` and `spawn_link/1` are the basic primitives for creating processes in Elixir. Although we have used them exclusively so far, most of the time we are going to use abstractions that build on top of them. Let's see the most common one, called tasks.

## Задачи

Задачи работают поверх порожденных функций для обеспечения расширеных сообщений об ошибках:

```iex
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
** (RuntimeError) oops
    (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
    (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []
```

Вместо функции `spawn/1` и `spawn_link/1`, мы используем `Task.start/1` и `Task.start_link/1` которые возвращают кортеж `{:ok, pid}` содержаший PID. Данные функции позволяют организовать наблюдение за ходом выполнения. Кроме того, `Task` предоставляет набор удобных функций, таких как `Task.async/1` и `Task.await/1`, и функционал для упрощения разработки распределенных систем.

Более подробно данный функционал будет описан в ***руководстве по Mix и OTP***, на данный момент просто запомните что мы используем `Task` для получения расширенной информации над ошибками.

## Состояние

We haven't talked about state so far in this guide. If you are building an application that requires state, for example, to keep your application configuration, or you need to parse a file and keep it in memory, where would you store it?

Processes are the most common answer to this question. We can write processes that loop infinitely, maintain state, and send and receive messages. As an example, let's write a module that starts new processes that work as a key-value store in a file named `kv.exs`:

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end
```

Note that the `start_link` function starts a new process that runs the `loop/1` function, starting with an empty map. The `loop/1` function then waits for messages and performs the appropriate action for each message. In the case of a `:get` message, it sends a message back to the caller and calls `loop/1` again, to wait for a new message. While the `:put` message actually invokes `loop/1` with a new version of the map, with the given `key` and `value` stored.

Let's give it a try by running `iex kv.exs`:

```iex
iex> {:ok, pid} = KV.start_link
{:ok, #PID<0.62.0>}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
nil
:ok
```

At first, the process map has no keys, so sending a `:get` message and then flushing the current process inbox returns `nil`. Let's send a `:put` message and try it again:

```iex
iex> send pid, {:put, :hello, :world}
{:put, :hello, :world}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
:world
:ok
```

Notice how the process is keeping a state and we can get and update this state by sending the process messages. In fact, any process that knows the `pid` above will be able to send it messages and manipulate the state.

It is also possible to register the `pid`, giving it a name, and allowing everyone that knows the name to send it messages:

```iex
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
:world
:ok
```

Using processes to maintain state and name registration are very common patterns in Elixir applications. However, most of the time, we won't implement those patterns manually as above, but by using one of the many abstractions that ship with Elixir. For example, Elixir provides [agents](https://hexdocs.pm/elixir/Agent.html), which are simple abstractions around state:

```iex
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

A `:name` option could also be given to `Agent.start_link/2` and it would be automatically registered. Besides agents, Elixir provides an API for building generic servers (called `GenServer`), tasks, and more, all powered by processes underneath. Those, along with supervision trees, will be explored with more detail in the ***руководство по Mix и OTP*** which will build a complete Elixir application from start to finish.

For now, let's move on and explore the world of I/O in Elixir.
