# crystal-pg
A Postgres driver for Crystal
[![Build Status](https://travis-ci.org/will/crystal-pg.svg?branch=master)](https://travis-ci.org/will/crystal-pg)
[![docrystal.org](http://www.docrystal.org/badge.svg?style=round)](http://www.docrystal.org/github.com/will/crystal-pg)


## usage

### connecting

``` crystal
DB = PG.connect("postgres://...")
```

### typed querying

The preferred way to send queries is to send a tuple of the types you expect back along with the query. `#rows` will then be an array of tuples with each element properly casted. You can also use parameterized queries for injection-safe server-side interpolation.

``` crystal
result = DB.exec({Int32, String}, "select id, email from users")
result.fields  #=> [PG::Result::Field, PG::Result::Field]
result.rows    #=> [{1, "will@example.com"}], …]
result.to_hash #=> [{"field1" => value, …}, …]

result = DB.exec({String}, "select $1::text || ' ' || $2::text", ["hello", "world"])
result.rows #=> [{"hello world"}]
```

Out of the box, crystal-pg supports 1-32 types. If you need more, you can reopen `PG::Result` and use the `generate_gather_rows` macro. If your field can return nil, you should use `PG::Nilable{{Type}}` for the type, which is a union of the type and `Nil`.

### untyped querying

If you do not know the types beforehand you can omit them. However you will get back an array of arrays of PGValue. Since it is a union type of amost every type, you will probably have to manually cast later on in your program.

``` crystal
result = DB.exec("select * from table")
result.fields  #=> [PG::Result::Field, …]
result.rows    #=> [[value, …], …]
result.to_hash #=> [{"field1" => value, …}, …]

result = DB.exec("select $1::text || ' ' || $2::text", ["hello", "world"])
result.rows #=> [["hello world"]]
```

## Requirements

Crystal-pg is [tested on](https://travis-ci.org/will/crystal-pg) Postgres versions 9.1 through 9.4. Since it is based on libpq, older versions probably also work but are not guaranteed.

Linking requires that the `pg_config` binary is in your `$PATH` and returns correct results for `pg_config --includedir` and `pg_config --libdir`.

## Supported Datatypes

- text
- boolean
- int8, int2, int4
- float4, float8
- timestamptz, date, timestamp (but no one should use ts when tstz exists!)
- json and jsonb


## Todo

- more datatypes (ranges, hstore)
- more info in postgres exceptions
- transaction help
- a lot more


