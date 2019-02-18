# The Elixir programming language: Reliable microservices made easy

This repository contains some examples used for the presenation of this topic at the microservice conference 2019 at FH-Dortmund.

## Getting started

Install erlang and elixir to execute the commands on this page.

```bash
# Just plain interactive elixir
iex

# Start named node alpha
iex --sname alpha --cookie secret

# ..or with longnames..
iex --name alpha@127.0.0.1 --cookie secret

# Staring node beta
iex --sname beta --cookie secret

# ..or..
iex --name beta@127.0.0.1 --cookie secret
```

## Elixir basics

```elixir
# Information about modules, variables, etc..
i System

# Built in help
h System

# Assign result of last command
a = v()

# Pattern matching
map = %{hello: %{a: 1, b: 2, c: 3}}
%{hello: %{c: c}} = map

# Runtime Information
runtime_info()

# Start Observer
:observer.start

# HTTP Requests with Erlang built in client
Application.ensure_all_started(:inets)
Application.ensure_all_started(:ssl)
:httpc.request('https://httpbin.org/uuid')
{:ok, {http_result, headers, body}} = :httpc.request('https://httpbin.org/uuid')
```

### Process basics

```elixir
# Spawn processes that output
Enum.each(1..1000, fn(i) -> spawn fn -> IO.puts "Hello #{i}" end end)

# Using tasks
task = Task.async(fn -> 41 + 1 end)
Task.await(task)

# Running background tasks
task = Task.async(fn ->
:httpc.request('http://httpbin.org/delay/7')
end)
Task.await(task, 60_000)

# Spawn a lot of processes
stream = Task.async_stream(1..1_000, fn(i) ->
Process.sleep(1_000)
i * 10
end)
Enum.each(stream, fn(i) -> IO.inspect(i) end)
Enum.map(stream, fn(i) -> IO.inspect(i) end)
```

### Distribution

Start named nodes and connect them via the Node module.

```elixir
# Listing connected nodes
Node.list

# Connecting nodes
Node.connect(:"alpha@HOSTNAME")

# Spawn a process on the other node
Node.spawn(:"alpha@HOSTNAME", fn() -> IO.puts("hello") end)

# Spawn a process and return the response (from alpha)
pid = Node.spawn_link(:beta@HOSTNAME -> 
  receive do
    {message, sender_pid} -> send(sender_pid, "Reply to: #{inspect message}")
  end
end)

send(pid, {:hello, self()})

flush()
```

### Distributed database with mnesia

Starting a distributed database with mnesia

Documentation:

- http://erlang.org/doc/man/mnesia.html
- https://elixirschool.com/en/lessons/specifics/mnesia/
- https://hex.pm/packages/ex2ms

```elixir
node_list = [node()] ++ Node.list()

:mnesia.create_schema(node_list)
:mnesia.start # On all nodes
:mnesia.system_info

:mnesia.create_table(:menu, [attributes: [:id, :name, :price]])

# Create a table with disc_copies on all nodes
:mnesia.create_table(:menu, [attributes: [:id, :name, :price], disc_copies: node_list])


# http://erlang.org/doc/man/mnesia.html#table_info-2
:mnesia.table_info(:menu, :all)

:mnesia.dirty_write({:menu, 1, "Tonno", 8})
:mnesia.dirty_read({:menu, 1})
:mnesia.dirty_delete({:menu, 1})

:mnesia.transaction(fn -> :mnesia.write({:menu, 1, "Tonno", 8}) end)
:mnesia.transaction(fn -> :mnesia.read({:menu, 1}) end)


```

## GenServer

Generic server behaviour implements a client/server relationship

```elixir
defmodule Demo do
  use GenServer

  def init(_args) do
    {:ok, %{}}
  end

  def handle_call(message, from, state) do
    IO.puts("handle_call(#{inspect message}, #{inspect from}, #{inspect state})")
    {:reply, :not_implemented, state}
  end

  def handle_cast(message, state) do
    IO.puts("handle_cast(#{inspect message}, #{inspect state})")
    {:noreply, state}
  end

  def handle_info(message, state) do
    IO.puts("handle_info(#{inspect message}, #{inspect state})")
    {:noreply, state}
  end

end
```

## Supervisor

Watch processes and restart them when they crash

```elixir
defmodule Demo do
  use GenServer

  def start_link(args) do
    GenServer.start_link(__MODULE__, args, name: __MODULE__)
  end

  def init(_args) do
    {:ok, %{}}
  end

  def something() do
    GenServer.call(__MODULE__, :something)
  end

  def die() do
    GenServer.call(__MODULE__, :die)
  end

  def handle_call(:die, _from, state) do
    raise "I am dying"
    {:bad}
  end
  def handle_call(message, from, state) do
    IO.puts("handle_call(#{inspect message}, #{inspect from}, #{inspect state})")
    {:reply, :not_implemented, state}
  end

  def handle_cast(message, state) do
    IO.puts("handle_cast(#{inspect message}, #{inspect state})")
    {:noreply, state}
  end

  def handle_info(message, state) do
    IO.puts("handle_info(#{inspect message}, #{inspect state})")
    {:noreply, state}
  end

end
```

## ETS

In memory database that runs as long as the node is alive

```elixir
:ets.new(:menu, [:set, :named_table, :public])
:ets.insert(:menu, {1, "Tonno", 9.99})
:ets.lookup(:menu, 1)
:ets.delete(:menu, 1)
:ets.info(:menu)
```



