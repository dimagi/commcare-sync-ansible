Migrating a server
==================

Here are the approximate steps to migrate a server from one environment to another.

1. Stand up a new server with ansible. For ease of migration, 
   it is recommended to use the same secrets as the previous environment unless there 
   is any reason to believe there was a compromise/breach. 
2. Back up the commcare-sync, superset, and data export tool databases to pgdump files.
3. Transfer the pgdump files to the new server.
4. Restore the commcare-sync and superset databases to the new server.
5. Copy the export config files from the old server to the new server.
6. Test.

**Backup commands**

(These commands are run on the old server.)

```
pg_dump -U commcare_sync -h localhost -p 5432 commcare_sync > ~/sync-db-backup.pgdump
pg_dump -U commcare_sync -h localhost -p 5432 superset > ~/superset-db-backup.pgdump
# add any other DBs created where the data might actually be housed
```

**Copy commands**

(These commands are run on the new server.)

```
scp oldserver.dimagi.com:sync-db-backup.pgdump ./
scp oldserver.dimagi.com:supeset-db-backup.pgdump ./
# other DBs here
```

**Restore commands**

(These commands are run on the new server.)

First you have to delete and recreate the databases that were created by ansible:

```
sudo -u postgres dropdb commcare_sync
sudo -u postgres dropdb superset
createdb -U commcare_sync -h localhost -p 5432 commcare_sync
createdb -U commcare_sync -h localhost -p 5432 superset
# need to create other DBs here
```

If you get errors about active connections when dropping databases try stopping all processes 
with `supervisorctl stop all` and killing any other active connections in a `psql` shell with something like the below:

```
SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE pg_stat_activity.datname = 'superset' AND pid <> pg_backend_pid();
```

Next you can restore them individually:

```
psql -U commcare_sync -h localhost -p 5432 commcare_sync < sync-db-backup.pgdump 
psql -U commcare_sync -h localhost -p 5432 superset < superset-db-backup.pgdump 
# restore other DBs here
```

*Copying config files*

On the old server, in the `~www/commcare-sync/code_root` folder:
```
tar -zcf ~/commcare-sync-media.tar.gz media/
```

On the new server, in the `~www/commcare-sync/code_root` folder:

```
tar -xzf ~/commcare-sync-media.tar.gz
```
