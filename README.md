# Crown

Crown is a [heir](http://erlang.org/doc/man/ets.html#heir) for your
ETS tables. Nothing more, nothing less. It provides a simple
[gen_server](http://erlang.org/doc/man/gen_server.html) that
deliberately provides no funtionality other acting as ETS heir by
responding to the `'ETS_TRANSFER'` message.


# Usage

There are two mechanisms to transfer ownership of a table from one
process to another: transferring on process termination, or by giving
the table away.

## Transfer on process termination

A simple way of creating a new ETS table and transferring ownership is
to use the `on_load`
[hook](http://erlang.org/doc/reference_manual/code_loading.html#id88557)
so that the table is created when your module is first referenced.

```erlang
-module(haystack_class).

-on_load(on_load/0).

-record(?MODULE, {
           id :: pos_integer(),
           name :: atom(),
           description :: string()
          }).

on_load() ->
    crown_table:new(?MODULE),
    add(1, in, <<"the internet">>),
    add(2, cs, <<"the CSNET class">>),
    add(3, ch, <<"the CHAOS class">>),
    add(4, hs, <<"Hesiod">>),
    add(254, none),
    add(255, any),
    ok.
```

In the above example the `haystack_class` module uses the `on_load`
hook to create a new table (in this case named after the `?MODULE`)
and adding some static reference data. The `on_load` function is run
by a new process which terminates at the end of the function. The
`crown_table:new(?MODULE)` function ensures that the newly created
table is correctly setup with its heir pointing to the
`crown_table_owner`.

Alternatively when creating a
[new ETS table](http://erlang.org/doc/man/ets.html#new-2), a
convenience function may be used instead to transfer ownership when
the creating process dies as follows:

```erlang
ets:new(abc, [crown_table_owner:heir()])
```

## Giveaway

Ownership may also be transfered by
[giving](http://www.erlang.org/doc/man/ets.html#give_away-3) the table
away.

```erlang
ets:new(abc, [named_table, public, {keypos, 2}]).
ets:give_away(abc, whereis(crown_table_owner), []).
```

In the above code the `abc` table is given away to
`crown_table_owner`.
