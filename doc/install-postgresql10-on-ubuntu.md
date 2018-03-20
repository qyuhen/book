# Install PostgreSQL 10 on Ubuntu

This is a quick guide to install PostgreSQL 10 - tested on Ubuntu 16.04 but likely can be used for Ubuntu 14.04 and 17.04 as well, with one minor modification detailed below.

## (Optional) Uninstall other versions of postgres

To make life simple, remove all other versions of Postgres. Obviously not required, but again, makes life simple.

```
dpkg -l | grep postgres
```

returned for me:

```
ii  postgresql                                  9.5+173                                                     all          object-relational SQL database (supported version)
ii  postgresql-9.5                              9.5.8-0ubuntu0.16.04.1                                      amd64        object-relational SQL database, version 9.5 server
ii  postgresql-client-9.5                       9.5.8-0ubuntu0.16.04.1                                      amd64        front-end programs for PostgreSQL 9.5
ii  postgresql-client-common                    173                                                         all          manager for multiple PostgreSQL client versions
ii  postgresql-common                           173                                                         all          PostgreSQL database-cluster manager
ii  postgresql-contrib-9.5                      9.5.8-0ubuntu0.16.04.1                                      amd64        additional facilities for PostgreSQL
```

... therefore I ran:

```sh
sudo apt-get --purge remove postgresql postgresql-9.5 postgresql-client-9.5 postgresql-client-common  postgresql-common postgresql-contrib-9.5
```


## Add PostgreSQL apt repository

The current default Ubuntu apt repositories only have up to postgresql-9.6. To get 10, we'll add the official postgres apt repository.

* Ubuntu 14.04: `sudo add-apt-repository 'deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main'`
* Ubuntu 16.04: `sudo add-apt-repository 'deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main'`
* Ubuntu 17.04: `sudo add-apt-repository 'deb http://apt.postgresql.org/pub/repos/apt/ zesty-pgdg main'`

Now import the repository signing key, followed by an update to the package lists:

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -
sudo apt-get update
```

## Install PostgreSQL

```sh
sudo apt-get install postgresql-10
```

Ensure that the server is started by switching to the postgres user.

```sh
sudo su - postgres
/usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main -l logfile start
```

If that fails, the server might be running, so restart it to be safe: `/usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main -l logfile restart`

That should return something like:

```sh
postgres@computer-name:~$ /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main -l logfile restart
waiting for server to shut down.... done
server stopped
waiting for server to start.... done
server started
```

Which means your PostgreSQL 10 is up and running!

## (Optional) Create a user for yourself

Since we're still logged in as the postgres user, now is a good time to create your own user account.
This allows you to use operating system level authentication locally, greatly simplifying access to the database. I also like to create a database with my username, as this is the default database that `psql` will connect to, and if the database doesn't exist `psql` throws a pesky error (which gets me every time).

```
psql
CREATE ROLE <username> SUPERUSER LOGIN REPLICATION CREATEDB CREATEROLE;
CREATE DATABASE <username> OWNER <username>;
\q
```

Now we can exit the postgres user and query normally!

```sh
postgres@computer:~$ logout
```
