# PostgreSQL archive driver for WeeWX

Use PostgreSQL through `user.postgresql`, implementing WeeWX's `weedb`
connection, transaction and schema interfaces. This is a database driver,
independent of your weather station hardware.

This repository mirrors [the upstream source](https://github.com/tkeffer/weewx-postgresql). The maintained
catalog archive is **1.0.1.post2**, with the configured port also passed during database initialization. Install that fixed archive below;
the mirrored default branch can differ from its repaired payload.

The catalog archive targets WeeWX `>=5.5.2,<6` and Python `>=3.10,<3.15`.

The driver remains a beta implementation. Its source requires WeeWX 5.3 or
later; the catalog compatibility range above is narrower.

## Prepare PostgreSQL

You need a PostgreSQL server and the Psycopg 3 client in WeeWX's Python
environment. The upstream tests use PostgreSQL 16. In a WeeWX virtual
environment, the binary distribution can be installed with:

```sh
python -m pip install 'psycopg[binary]>=3,<4'
```

Alternatively install `psycopg` with its system `libpq` dependencies. Create a
role with permission to create the WeeWX database. For example, enter `psql`
as an administrator and choose your own password:

```sh
sudo -u postgres psql
```

```sql
CREATE USER weewx WITH PASSWORD 'REPLACE_WITH_YOUR_PASSWORD';
ALTER USER weewx CREATEDB;
```

Server authentication rules must allow the connection you configure.

## Install

Use `weectl` from the Python environment running WeeWX. With stock WeeWX 5,
install the published archive:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/postgresql/v1.0.1.post2/postgresql-1.0.1.post2.tar.gz --config /path/to/weewx.conf
```

Replace `/path/to/weewx.conf` with your station configuration. You can also
download that URL and pass the local archive to the same command. Use elevated
permissions only when your station files require them.

## Configure the archive binding

The installer adds `[Databases] [[archive_postgresql]]` and
`[DatabaseTypes] [[PostgreSQL]]`. Replace its sample credentials and select the
new database in your existing `wx_binding`:

```ini
[DatabaseTypes]
    [[PostgreSQL]]
        driver = user.postgresql
        host = localhost
        port = 5432
        user = weewx
        password = REPLACE_WITH_YOUR_PASSWORD
        real_as_double = true
[Databases]
    [[archive_postgresql]]
        database_name = weewx_data
        database_type = PostgreSQL
[DataBindings]
    [[wx_binding]]
        database = archive_postgresql
```

Keep the binding's existing table, manager and schema settings. Restart WeeWX
yourself and inspect its database initialization log. A new target database
starts empty. Changing this binding does not copy an existing SQLite or MySQL
archive; keep the original database and plan migration separately.

## Driver options and limits

`real_as_double = true` maps schema `REAL` columns to PostgreSQL `DOUBLE
PRECISION`, preserving 8-byte floating-point storage. It defaults to true.
`maintenance_db` selects the administrative database, default `postgres`.
Empty host, user, password or database-name settings allow Psycopg/libpq's
normal environment defaults. Additional connection arguments reach `connect`;
the create/drop helpers use a smaller set of arguments.

The driver maintains `weewx_db__metadata` for mixed-case table and column names.
An arbitrary existing PostgreSQL table is not a ready WeeWX archive. The
catalog `.post2` archive fixes initialization on a nondefault PostgreSQL port;
its runtime version still reports `1.0.1`. Use the fixed archive rather than a
moving upstream ZIP. See [changes.md](changes.md) and
[the driver](bin/user/postgresql.py) for supported database operations.

Copyright 2009-2026 Tom Keffer and contributors. [GNU GPLv3](LICENSE.txt).
