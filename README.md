# SQL migrations for Golang and PostgreSQL

[![Build Status](https://travis-ci.org/go-pg/migrations.svg)](https://travis-ci.org/go-pg/migrations)
[![GoDoc](https://godoc.org/github.com/go-pg/migrations?status.svg)](https://godoc.org/github.com/go-pg/migrations)

This package allows you to run migrations on your PostgreSQL database using [Golang Postgres client](https://github.com/go-pg/pg). See [example](example) for details.


# Example

You need to create database `pg_migrations_example` before running this example.

```bash
> psql -c "CREATE DATABASE pg_migrations_example"
CREATE DATABASE

> go run *.go version
version is 0

> go run *.go
creating table my_table...
adding id column...
seeding my_table...
migrated from version 0 to 3

> go run *.go version
version is 3

> go run *.go reset
truncating my_table...
dropping id column...
dropping table my_table...
migrated from version 3 to 0

> go run *.go up 2
creating table my_table...
adding id column...
migrated from version 0 to 2

> go run *.go
seeding my_table...
migrated from version 2 to 3

> go run *.go down
truncating my_table...
migrated from version 3 to 2

> go run *.go version
version is 2

> go run *.go set_version 1
migrated from version 2 to 1

> go run *.go create add email to users
created migration 4_add_email_to_users.go
```

## Registering Migrations

### `migrations.RegisterTx` and `migrations.MustRegisterTx`

Registers migrations to be executed inside transactions.


### `migrations.Register` and `migrations.MustRegister`

Registers migrations to be executed without any transaction.


## Transactions

By default, the migrations are executed outside without any transactions. Individual migrations can however be marked to be executed inside transactions by using the `RegisterTx` function instead of `Register`. 

### Global Transactions

```go
var oldVersion, newVersion int64

err := db.RunInTransaction(func(tx *pg.Tx) (err error) {
    oldVersion, newVersion, err = migrations.Run(tx, flag.Args()...)
    return
})
if err != nil {
    exitf(err.Error())
}
```
