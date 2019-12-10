# postgres-backup

This image runs pg_dump to backup data using cronjob to folder `/backup`. Backups are named with date and time, the latest backup is linked to `/backup/latest.psql.gz`.

Uses Alpine Linux for a small image.

PS.: this is a fork from stffabi/postgres-backup image, with some updates and
modifications for my needs, including Postgresql 11.6.

## Usage:

    docker run -d \
        --env PG_HOST=postgresdb.host \
        --env PG_PORT=5432 \
        --env PG_USER=admin \
        --env PG_PASS=password \
        --volume host.folder:/backup
        victorouttes/postgres-backup

Moreover, if you link `victorouttes/postgres-backup` to a postgres container with an alias named postgres, this image has defaults that will connect to `postgres` on port 5432 with user `root` and no password.

    docker run -d -e PG_ALLOW_EMPTY_PASSWORD=true --name postgresdb postgres
    docker run -d --link postgresdb:postgresdb -v host.folder:/backup victorouttes/postgres-backup

## Compose:

Use with a Postgresql image like the example:

```
version: "3"

services:
  db:
    image: postgres:11.6
    restart: always
    env_file:
      - .env
    environment:
      POSTGRES_USER: "${POSTGRES_USER}"
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD}"
    volumes:
      - /host/folder/data:/var/lib/postgresql/data

  backup:
    image: victorouttes/postgres-backup
    restart: always
    env_file:
      - .env
    environment:
      PG_HOST: db
      PG_PORT: 5432
      PG_USER: "${POSTGRES_USER}"
      PG_PASS: "${POSTGRES_PASSWORD}"
      EXTRA_OPTS: "-C"
      MAX_BACKUPS: 10
      CRON_TIME: "0 0 * * *"
    volumes:
      - /host/folder/backups:/backup
    depends_on:
      - db
```

## Parameters

    UID             the user id, default: 65534
    GID             the group id, default: 65534
    PG_HOST         the host/ip of your postgres database
    PG_PORT         the port number of your postgres database
    PG_USER         the username of your postgres database
    PG_PASS         the password of your postgres database
    PG_DB           the database name to dump
    EXTRA_OPTS      the extra options to pass to pg_dump command
    CRON_TIME       the interval of cron job to run pg_dump. "0 0 * * *" by default, which is every day at 00:00
    MAX_BACKUPS     the number of backups to keep. When reaching the limit, the old backup will be discarded. No limit by default
    INIT_BACKUP     if set, create a backup when the container starts
    INIT_RESTORE_LATEST if set, restores latest backup
    NO_CRON         if set, do not start cron. Must be used with INIT_BACKUP to run a single backup and then exit

## Restore from a backup

See the list of backups, you can run:

    docker exec postgres-backup ls /backup

To restore database from a certain backup, simply run:

    docker exec postgres-backup /restore.sh /backup/2019.12.10.171901.psql.gz
    
You may need to drop the database before restore.

## Support

Add a [GitHub issue](https://github.com/victorouttes/docker-postgres-backup/issues).
