# Ecto.ULID Next+

An `Ecto.Type` implementation of [ULID](https://github.com/ulid/spec).

`Ecto.ULID` should be compatible anywhere that `Ecto.UUID` is supported. It has been confirmed to
work with PostgreSQL and MySQL on Ecto ~> 3.2.

ULID is a 128-bit universally unique lexicographically sortable identifier. ULID is
binary-compatible with UUID, so it can be stored in a `uuid` column in a database.

**This is a fork of [TheRealReal/ecto-ulid](https://github.com/TheRealReal/ecto-ulid), which doesn't seem to be maintained anymore. The aim is to ensure compatibility with current and future Ecto versions.**

Also a fork on [ulid-next](https://github.com/woylie/ecto-ulid), which includes prefix handling and builtin ULID downcasing.

## Features

- Generate ULID in Base32 or binary format.
- Generate ULID for a given timestamp.
- Autogenerate ULID when used as a primary key.
- Supports reading and writing ULID in a database backed by its native `uuid` type (no database
  extensions required).
- Supports Ecto ~> 3.2.
- Supports Elixir 1.7 and newer.
- Confirmed working on PostgreSQL and MySQL.
- Optimized for high throughput.

## Performance

Since one use case of ULID is to handle a large volume of events, `Ecto.ULID` is optimized to be as
fast as possible. It borrows techniques from `Ecto.UUID` to achieve sub-microsecond times for most
operations.

A benchmark suite is included. Download the repository and run `mix run bench/ulid_bench.exs` to test the performance
on your system.

## Usage

Usage is very similar to `Ecto.UUID`. The following example shows how to use `Ecto.ULID` as a
primary key in a database table, but it can be used for other columns just as easily.

[API documentation](https://hexdocs.pm/ecto_ulid_next) is available on hexdocs.

### Install

Install `ecto_ulid_next` from Hex by adding it to the dependencies in `mix.exs`:

```elixir
defp deps do
  [
    {:ecto_ulid_next, "~> 0.3.0", github: "sourdoughdev/ecto-ulid"}
  ]
end
```

### Migration

Since ULID is binary-compatible with UUID, the migrations look the same for both types. Use
`:binary_id` when defining a column in a migration:

```elixir
create table(:events, primary_key: false) do
  add :id, :binary_id, null: false, primary_key: true
  # more columns ...
end
```

Alternatively, if you plan to use ULID as the primary key type for all of your tables, you can set
`migration_primary_key` when configuring your `Repo`:

```elixir
config :my_app, MyApp.Repo, migration_primary_key: [name: :id, type: :binary_id]
```

and then you _do not_ need to specify the `id` column in your migrations:

```elixir
create table(:events) do
  # more columns ...
end
```

### Schema

When defining a model's schema, use `Ecto.ULID` as the `@primary_key` or `@foreign_key_type` as
appropriate for your schema. Here's an example of using both:

```elixir
defmodule MyApp.Event do
  use Ecto.Schema

  @primary_key {:id, Ecto.ULID, prefix: "evnt" autogenerate: false}
  @foreign_key_type Ecto.ULID
  schema "events" do
    # more columns ...
  end
end
```

`Ecto.ULID` supports `autogenerate: true` as well as `autogenerate: false` when used as the primary
key.

### Application Usage

A ULID can be generated in string or binary format by calling `generate/0` or `bingenerate/0`. This
can be useful when generating ULIDs to send to external systems:

```elixir
Ecto.ULID.generate()    #=> "01BZ13RV29T5S8HV45EDNC748P"
Ecto.ULID.bingenerate() #=> <<1, 95, 194, 60, 108, 73, 209, 114, 136, 236, 133, 115, 106, 195, 145, 22>>
```

To backfill old data, it may be helpful to pass a timestamp to `generate/1` or `bingenerate/1`. See
the [API documentation](https://hexdocs.pm/ecto_ulid) for more details.

## License

Copyright © 2018 The RealReal, Inc.

Distributed under the [MIT License](./LICENSE).
