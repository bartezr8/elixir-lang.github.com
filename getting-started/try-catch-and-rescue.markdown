---
layout: getting-started
title: try, catch, и rescue
---

# {{ page.title }}

{% include toc.html %}

Elixir has three error mechanisms: errors, throws, и exits. In this chapter we will explore each of them и include remarks about when each should be used.

## Ошибки

Errors (or *exceptions*) are used when exceptional things happen in the code. A sample error can be retrieved by trying to add a number into an atom:

```iex
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

A runtime error can be raised any time by using `raise/1`:

```iex
iex> raise "oops"
** (RuntimeError) oops
```

Other errors can be raised with `raise/2` passing the error name и a list of keyword arguments:

```iex
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

You can also define your own errors by creating a module и using the `defexception` construct inside it; this way, you'll create an error with the same name as the module it's defined in. The most common case is to define a custom exception with a message field:

```iex
iex> defmodule MyError do
iex>   defexception message: "default message"
iex> end
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

Errors can be **rescued** using the `try/rescue` construct:

```iex
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "oops"}
```

The example above rescues the runtime error и returns the error itself which is then printed in the `iex` session.

If you don't have any use for the error, you don't have to provide it:

```iex
iex> try do
...>   raise "oops"
...> rescue
...>   RuntimeError -> "Error!"
...> end
"Error!"
```

In practice, however, Elixir developers rarely use the `try/rescue` construct. For example, many languages would force you to rescue an error when a file cannot be opened successfully. Elixir instead provides a `File.read/1` function which returns a tuple containing information about whether the file was opened successfully:

```iex
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```

There is no `try/rescue` here. In case you want to handle multiple outcomes of opening a file, you can use pattern matching within the `case` construct:

```iex
iex> case File.read "hello" do
...>   {:ok, body}      -> IO.puts "Success: #{body}"
...>   {:error, reason} -> IO.puts "Error: #{reason}"
...> end
```

At the end of the day, it's up to your application to decide if an error while opening a file is exceptional or not. That's why Elixir doesn't impose exceptions on `File.read/1` and many other functions. Instead, it leaves it up to the developer to choose the best way to proceed.

For the cases where you do expect a file to exist (and the lack of that file is truly an *error*) you may use `File.read!/1`:

```iex
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:305: File.read!/1
```

Many functions in the standard library follow the pattern of having a counterpart that raises an exception instead of returning tuples to match against. The convention is to create a function (`foo`) which returns `{:ok, result}` или `{:error, reason}` tuples и another function (`foo!`, same name but with a trailing `!`) that takes the same arguments as `foo` but which raises an exception if there's an error. `foo!` should return the result (not wrapped in a tuple) if everything goes fine. The [`File` module](https://hexdocs.pm/elixir/File.html) is a good example of this convention.

В Elixir, we avoid using `try/rescue` because **we don't use errors for control flow**. We take errors literally: they are reserved for unexpected и/или exceptional situations. In case you actually need flow control constructs, *throws* should be used. That's what we are going to see next.

## Исключения

В Elixir, a value can be thrown и later be caught. `throw` и `catch` are reserved for situations where it is not possible to retrieve a value unless by using `throw` и `catch`.

Those situations are quite uncommon in practice except when interfacing with libraries that do not provide a proper API. For example, let's imagine the `Enum` module did not provide any API for finding a value и that we needed to find the first multiple of 13 in a list of numbers:

```iex
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

Since `Enum` *does* provide a proper API, in practice `Enum.find/2` is the way to go:

```iex
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## Exits

Все код в Elixir запускается внутри процессов которые могут взаимодействовать друг с другом. Когда процесс умирает по "естественным причинам" (в отличии, от необработанных исключений), он отправляет сигнал `exit`. Процесс может погибнуть отправив при этом сигнал завершения:

```iex
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```

В примере выше, процесс погибает отправив сигнал `exit` с значением 1. Оболочка Elixir автоматически обрабатывает сообщения и выводит его в терминал.

Сигнал `exit` можно "отловить" испльзуя `try/catch`:

```iex
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

Для подобных случаев конструкция `try/catch` используется не часто, для обработки завершения процесса ещё реже.

Сигнал `exit` один из способов дружественного взаимодействия с процессами в Erlang <abbr title="Virtual Machine">VM</abbr>. Processes usually run under supervision trees which are themselves processes that listen to `exit` signals from the supervised processes. Once an exit signal is received, the supervision strategy kicks in and the supervised process is restarted.

It is exactly this supervision system that makes constructs like `try/catch` and `try/rescue` so uncommon in Elixir. Instead of rescuing an error, we'd rather "fail fast" since the supervision tree will guarantee our application will go back to a known initial state after the error.

## After

Sometimes it's necessary to ensure that a resource is cleaned up after some action that could potentially raise an error. The `try/after` construct allows you to do that. For example, we can open a file and use an `after` clause to close it--even if something goes wrong:

```iex
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

The `after` clause will be executed regardless of whether or not the tried block succeeds. Note, however, that if a linked process exits,
this process will exit and the `after` clause will not get run. Thus `after` provides only a soft guarantee. Luckily, files in Elixir are also linked to the current processes and therefore they will always get closed if the current process crashes, independent of the
`after` clause. You will find the same to be true for other resources like ETS tables, sockets, ports and more.

Sometimes you may want to wrap the entire body of a function in a `try` construct, often to guarantee some code will be executed afterwards. In such cases, Elixir allows you to omit the `try` line:

```iex
iex> defmodule RunAfter do
...>   def without_even_trying do
...>     raise "oops"
...>   after
...>     IO.puts "cleaning up!"
...>   end
...> end
iex> RunAfter.without_even_trying
cleaning up!
** (RuntimeError) oops
```

Elixir will automatically wrap the function body in a `try` whenever one of `after`, `rescue` or `catch` is specified.

## Else

Providing an `else` block allows for pattern matching on the result of the expression executed.

```iex
x = 2
try do
  1 / x
rescue
  ArithmeticError ->
    :infinity
else
  y when y < 1 and y > -1 ->
    :small
  _ ->
    :large
end
```

The result is passed to `else` where it can be pattern matched. Please note that the `else` block is also executed if an exception has been caught, the result of the `catch/rescue` block will then be passed in.

Exceptions in the `else` block are not caught. If no pattern inside the `else` block matches an exception will be raised, this exception is not caught by the current `try/catch/rescue/after` block.

## Variables scope

It is important to bear in mind that variables defined inside `try/catch/rescue/after` blocks do not leak to the outer context. This is because the `try` block may fail and as such the variables may never be bound in the first place. In other words, this code is invalid:

```iex
iex> try do
...>   raise "fail"
...>   what_happened = :did_not_raise
...> rescue
...>   _ -> what_happened = :rescued
...> end
iex> what_happened
** (RuntimeError) undefined function: what_happened/0
```

Instead, you can store the value of the `try` expression:

```iex
iex> what_happened =
...>   try do
...>     raise "fail"
...>     :did_not_raise
...>   rescue
...>     _ -> :rescued
...>   end
iex> what_happened
:rescued
```

This finishes our introduction to `try`, `catch`, and `rescue`. You will find they are used less frequently in Elixir than in other languages, although they may be handy in some situations where a library or some particular code is not playing "by the rules".
