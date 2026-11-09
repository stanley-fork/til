# Connect To App Database With `dbconsole`

For about as long as I've been a Rails developer, I've been comfortable with the
command line and Postgres tooling. Because of that, whenever I've needed to
connect to a local or remote database, I typically invoke `psql` directly.

```bash
❯ psql still_development
psql (18.4)
Type "help" for help.

::1 postgres@still_development=#
```

Because of that, I somehow completely missed the existence of
[`dbconsole`](https://guides.rubyonrails.org/command_line.html#bin-rails-dbconsole).
It's a Rails command that will look at your database adapter and database config
for the relevant environment and connect you directly to the database.

```bash
❯ bin/rails dbconsole
psql (18.4)
Type "help" for help.

::1 postgres@still_development=#
```

When it sees that your app is using the Postgres adapter, it is going to:
- Use the values from `config/database.yml` for the current environment to set
env vars like `PGHOST`, `PGUSER`, etc.
- Invoke the `psql` it finds on your path which will pick up those env vars

I can also use this to connect to an app's database in a remote environment,
e.g. my staging env. Because I don't have actually staging values set in
`config/database.yml`, I would first need to `ssh` onto that remote server, load
the env, and then run `bin/rails dbconsole`.
