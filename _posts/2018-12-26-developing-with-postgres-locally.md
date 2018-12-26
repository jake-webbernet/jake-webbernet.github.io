Developing with Postgres locally I always needed to specify a username and password to connect. This was a bit of pain because I needed to keep a different version of `config/database.yml` to the rest of my development team. Not a huge deal most of the time as you can ask git to forget the file (different to gitignore). This seemed to cause issues though working with stashes.

I also only started having this issue working on Windows. Developing on a Mac, I was just able to always connect to my postgres installation without a username and password.

The reason this was the case was because of some configuration with the Postgres installation. The answer was in the `pg_hba` configuration file. In Windows, I found mine in the Postgres data directory `C:\Program Files\PostgreSQL\10\data\pg_hba`


The `pg_hba` file describes itself as

```
# This file controls: which hosts are allowed to connect, how clients
# are authenticated, which PostgreSQL user names they can use, which
# databases they can access.  Records take one of these forms:
```

At the bottom of the file, there was a table that looks like this.

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

You can see above that my configuration has `md5` for each entry which indicates that every connection is going to need a password. `MD5` would mean that it's encrypted.

**Note** You should only make this change in your development environment.

I replaced the above table with this:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host all             all                127.0.0.1/32            trust
```

This will allow all hosts to be able to access postgres with `trust`. Meaning they can connect without credentials.

After editing this file, restart the Postgres service to apply the changes.
