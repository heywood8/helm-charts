![Docker Pulls](https://img.shields.io/docker/pulls/prodrigestivill/postgres-backup-local)

# postgres-backup-local

Backup PostgresSQL to the local filesystem with periodic rotating backups, based
on [schickling/postgres-backup-s3](https://hub.docker.com/r/schickling/postgres-backup-s3/). Backup multiple databases
from the same host by setting the database names in `POSTGRES_DB` separated by commas or spaces.

Supports the following Docker architectures: `linux/amd64`, `linux/arm64`, `linux/arm/v7`, `linux/s390x`
, `linux/ppc64le`.

## Usage

Helm:

```sh
helm upgrade --install postgres-backup-local .
```

values:

```yaml
postgres-backup-local:
  auth:
    # must match release name
    existingPasswordSecret: postgres-backup-local-auth
  config:
    postgres_host: pg-auth-db-1414-1212-468f-1111-ae0.aivencloud.com
    postgres_port: 21699
    postgres_db: auth
    postgres_extra_opts: "-Z6"
    schedule: "@daily"
    backup_keep_days: 7
    backup_keep_weeks: 4
    backup_keep_months: 6
    healthcheck_port: 8080

  replicas: 1

  persistence:
    enabled: true
    storageClass: "longhorn-crypto-global"
    mountPath: /backups
    size: 8Gi
```

For security reasons it is recommended to run it as user `postgres:postgres`.

In case of running as `postgres` user, the system administrator must initialize the permission of the destination folder
as follows:

```sh
# for default images (debian)
mkdir -p /var/opt/pgbackups && chown -R 999:999 /var/opt/pgbackups
# for alpine images
mkdir -p /var/opt/pgbackups && chown -R 70:70 /var/opt/pgbackups
```

### Environment Variables

Most variables are the same as in the [official postgres image](https://hub.docker.com/_/postgres/).

| env variable            | description                                                                                                                                                                                                                                             |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| BACKUP_DIR              | Directory to save the backup at. Defaults to `/backups`.                                                                                                                                                                                                |
| BACKUP_SUFFIX           | Filename suffix to save the backup. Defaults to `.sql.gz`.                                                                                                                                                                                              |
| BACKUP_KEEP_DAYS        | Number of daily backups to keep before removal. Defaults to `7`.                                                                                                                                                                                        |
| BACKUP_KEEP_WEEKS       | Number of weekly backups to keep before removal. Defaults to `4`.                                                                                                                                                                                       |
| BACKUP_KEEP_MONTHS      | Number of monthly backups to keep before removal. Defaults to `6`.                                                                                                                                                                                      |
| BACKUP_KEEP_MINS        | Number of minutes for `last` folder backups to keep before removal. Defaults to `1440`.                                                                                                                                                                 |
| HEALTHCHECK_PORT        | Port listening for cron-schedule health check. Defaults to `8080`.                                                                                                                                                                                      |
| POSTGRES_DB             | Comma or space separated list of postgres databases to backup. Required.                                                                                                                                                                                |
| POSTGRES_DB_FILE        | Alternative to POSTGRES_DB, but with one database per line, for usage with docker secrets.                                                                                                                                                              |
| POSTGRES_EXTRA_OPTS     | Additional [options](https://www.postgresql.org/docs/12/app-pgdump.html#PG-DUMP-OPTIONS) for `pg_dump` (or `pg_dumpall` [options](https://www.postgresql.org/docs/12/app-pg-dumpall.html#id-1.9.4.13.6) if POSTGRES_CLUSTER is set). Defaults to `-Z6`. |
| POSTGRES_CLUSTER        | Set to `TRUE` in order to use `pg_dumpall` instead. Also set POSTGRES_EXTRA_OPTS to any value or empty since the default value is not compatible with `pg_dumpall`.                                                                                     |
| POSTGRES_HOST           | Postgres connection parameter; postgres host to connect to. Required.                                                                                                                                                                                   |
| POSTGRES_PASSWORD       | Postgres connection parameter; postgres password to connect with. Required.                                                                                                                                                                             |
| POSTGRES_PASSWORD_FILE  | Alternative to POSTGRES_PASSWORD, for usage with docker secrets.                                                                                                                                                                                        |
| POSTGRES_PASSFILE_STORE | Alternative to POSTGRES_PASSWORD in [passfile format](https://www.postgresql.org/docs/12/libpq-pgpass.html#LIBPQ-PGPASS), for usage with postgres clusters.                                                                                             |
| POSTGRES_PORT           | Postgres connection parameter; postgres port to connect to. Defaults to `5432`.                                                                                                                                                                         |
| POSTGRES_USER           | Postgres connection parameter; postgres user to connect with. Required.                                                                                                                                                                                 |
| POSTGRES_USER_FILE      | Alternative to POSTGRES_USER, for usage with docker secrets.                                                                                                                                                                                            |
| SCHEDULE                | [Cron-schedule](http://godoc.org/github.com/robfig/cron#hdr-Predefined_schedules) specifying the interval between postgres backups. Defaults to `@daily`.                                                                                               |
| TZ                      | [POSIX TZ variable](https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html) specifying the timezone used to evaluate SCHEDULE cron (example "Europe/Paris").                                                                               |

#### Special Environment Variables

This variables are not intended to be used for normal deployment operations:

| env variable                | description                                        |
|-----------------------------|----------------------------------------------------|
| POSTGRES_PORT_5432_TCP_ADDR | Sets the POSTGRES_HOST when the latter is not set. |
| POSTGRES_PORT_5432_TCP_PORT | Sets POSTGRES_PORT when POSTGRES_HOST is not set.  |

### Manual Backups

By default this container makes daily backups, but you can start a manual backup by running `/backup.sh`.

This script as example creates one backup as the running user and saves it the working folder.

```sh
docker run --rm -v "$PWD:/backups" -u "$(id -u):$(id -g)" -e POSTGRES_HOST=postgres -e POSTGRES_DB=dbname -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password  prodrigestivill/postgres-backup-local /backup.sh
```

### Automatic Periodic Backups

You can change the `SCHEDULE` environment variable in `-e SCHEDULE="@daily"` to alter the default frequency. Default
is `daily`.

More information about the scheduling can be
found [here](http://godoc.org/github.com/robfig/cron#hdr-Predefined_schedules).

Folders `daily`, `weekly` and `monthly` are created and populated using hard links to save disk space.

## Restore examples

Some examples to restore/apply the backups.

### Restore using the same container

To restore using the same backup container, replace `$BACKUPFILE`, `$CONTAINER`, `$USERNAME` and `$DBNAME` from the
following command:

```sh
docker exec --tty --interactive $CONTAINER /bin/sh -c "zcat $BACKUPFILE | psql --username=$USERNAME --dbname=$DBNAME -W"
```

### Restore using a new container

Replace `$BACKUPFILE`, `$VERSION`, `$HOSTNAME`, `$PORT`, `$USERNAME` and `$DBNAME` from the following command:

```sh
docker run --rm --tty --interactive -v $BACKUPFILE:/tmp/backupfile.sql.gz postgres:$VERSION /bin/sh -c "zcat /tmp/backupfile.sql.gz | psql --host=$HOSTNAME --port=$PORT --username=$USERNAME --dbname=$DBNAME -W"
```
