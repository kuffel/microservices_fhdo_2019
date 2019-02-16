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
Process.sleep(30_000)
:done
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

## Distribution

Start named nodes and connect them via the Node module.

```elixir
# Listing connected nodes
Node.list

# Connecting nodes
Node.connect(:"alpha@HOSTNAME")

# Spawn a process on the other node
Node.spawn(:"alpha@HOSTNAME", fn() -> IO.puts("hello") end)
```

### Distributed database with mnesia

```elixir
:mnesia.create_schema([node(), :"alpha@Adams-MBP-10"])
:mnesia.start # On both nodes
:mnesia.system_info

:mnesia.create_table(:menu, [attributes: [:id, :name, :price]])

# http://erlang.org/doc/man/mnesia.html#table_info-2
:mnesia.table_info(:menu, :all)

:mnesia.dirty_write({:menu, 1, "Tonno", 8})
:mnesia.dirty_read({:menu, 1})

:mnesia.transaction(fn -> :mnesia.write({:menu, 1, "Tonno", 8}) end)
:mnesia.transaction(fn -> :mnesia.read({:menu, 1}) end)

# https://elixirschool.com/en/lessons/specifics/mnesia/
```

## GenServer

Generic server behaviour implements a client/server relationship

```elixir

```

## Supervisor

Watch processes and restart them when they crash

```elixir

```
