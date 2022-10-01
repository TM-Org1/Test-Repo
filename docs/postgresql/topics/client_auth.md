Move your `{{ ca_cert }}` certificate to your PostgreSQL data directory—often at `/var/lib/pgsql/data` or `/usr/local/pgsql/data`—and name it `root.crt` (the usual convention, though other paths are possible).

```shell-session
$ sudo cp {{ ca_cert }} /var/lib/pgsql/data/root.crt
```

Make sure PostgreSQL has access to the file.

```shell-session
$ sudo chown postgres:postgres /var/lib/pgsql/data/root.crt
```

Configure `postgresql.conf` to point to your root CA certificate. PostgreSQL will use this certificate to verify certificates presented by clients.

```ini
# ...
ssl_ca_file = 'root.crt'
# ...
```

Configure `pg_hba.conf`, creating `hostssl` records with the `clientcert=1` option for all relevant connections. It might look something like this:

```ini
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# ...

# IPv4 remote connections for authenticated users
hostssl all             myuser          0.0.0.0/0               md5 clientcert=1
```

Alternatively, if you'd like to disable password authentication and lean exclusively on client certificates for authentication, change from the `md5` authentication method and use the `cert` method instead. By default, this requires that the identity used as the Common Name in the certificate (when issued by your CA, eg. `{{ server_name }}`) exactly matches the PostgreSQL database user specified in connections from clients. User name mapping can be used to allow the Common Name to be different from the database user name. See more in the PostgreSQL [docs on certificate authentication](https://www.postgresql.org/docs/current/auth-cert.html).

```ini
# ...
hostssl all             myuser          0.0.0.0/0               cert clientcert=1
```

Finally, restart your PostgreSQL server for your changes to take effect.
