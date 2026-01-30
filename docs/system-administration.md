System Administration
=====================

Philosophy
----------

CommCare Data Pipeline deployments use an
"[infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)"
philosophy which means all configuration is documented in code files.
See the sections below to access everything needed to configure the
system.


Services
--------

The complete list of services is available in the
[roles/commcare_sync/tasks/main.yml](https://github.com/dimagi/commcare-sync-ansible/blob/master/roles/commcare_sync/tasks/main.yml)
file.

The most important ones are summarized on the
[What's Installed](whats-installed.md) page.


Common Tasks
------------

Some of the common tasks needed to manage an environment.

### Enabling/Disabling Public Signups

> [!DANGER]
> Public Signups are a development or testing feature, and should never
> be enabled in production.

Signups are currently configured via the Django `ACCOUNT_ADAPTER`
setting. To enable anyone to sign up, you should set it to
`EmailAsUsernameAdapter`. To disable public account creation, set it to
`NoNewUsersAccountAdapter`. This behavior is controlled by the
`django_allow_public_signups` Ansible variable.

If public signups are disabled, then only superusers can create new
accounts, via the Django Admin UI or command line.

### Create A Superuser

Change into the CommCare Data Pipeline directory, activate the virtual
environment, and run the "createsuperuser" management command.

```shell
sudo -iu ansible
cd ~/www/commcare-sync/code_root/
source .venv/bin/activate
./manage.py createsuperuser
```

Then follow the prompts.

### Add A CommCare HQ Server

By default, CommCare Data Pipeline is preconfigured with
"www.commcarehq.org" as the default site from which to pull data. To
pull data from a different instance of CommCare HQ, you will need to add
the server parameters.

1. Log into the CommCare Data Pipeline server using the superuser
   account you created above.

2. Click the "Admin" link in the top right of the page.

3. Click "CommCare Server" > "Add CommCare Server".

4. Fill in the name of the server and the server URL.

5. Click "Save"

### Deploying Changes

To deploy updates to the server, log in as your user on the remote
server:

```shell
ssh ubuntu@cc-sync.example.com
```

On the remote server, add your SSH key to the SSH agent, and run the
Ansible playbook with `--tags=deploy`:

```shell
eval `ssh-agent`
ssh-add ~/.ssh/id_ed25519_example
ansible-playbook -i inventories/your-environment \
    commcare_sync.yml \
    --vault-password-file /path/to/password/file \
    --tags=deploy \
    -vv
```

### Activating the Virtual Environment

To activate the virtual environments for CommCare Data Pipeline and
Superset, run the following commands:

```
source .venv/bin/activate
source .venv/bin/postactivate
```

"postactivate" is required to set project-specific environment
variables.

The CommCare Data Pipeline virtual environment is in
`~ansible/www/commcare-sync/code_root/.venv/` and the Superset virtual
environment is in `~ansible/www/.virtualenvs/superset`.

### Other Useful Commands

| Task                             | Command                                     |
|----------------------------------|---------------------------------------------|
| See running Supervisor processes | `sudo supervisorctl status`                 |
| Restart a Supervisor process     | `sudo supervisorctl restart [process name]` |
| Connect to the local database    | `sudo -u postgres psql`                     |
| Restart nginx                    | `sudo service nginx restart`                |


### Checking for Stuck Exports

The logs for running exports don't show up in the UI until they
complete. The easiest way to see if an export is still running or if it
is stuck or was killed is to log into the server and just run a command
to see what "commcare-export" processes are running:

```shell
ps -ef | grep commcare-export
```


Database Management
-------------------

CommCare Data Pipeline uses a PostgreSQL system database, configured in
the Django settings file, `commcare_sync/settings_local.py`. You can
open a shell to it using `./manage.py dbshell` in the CommCare Data
Pipeline virtual environment.

CommCare Data Pipeline can export data to databases that are served by
the same database server, or by different servers. Use an appropriate
database client to access them. [DBeaver](https://dbeaver.io/) for SQL
Server and PostgreSQL, and [pgAdmin](https://www.pgadmin.org/) for
PostgreSQL are popular choices.

To access the local database when logged into the server, you can use
`psql` with the credentials from the vault file:

```shell
psql -U [postgres user] -h localhost -p 5432
```


Database Backup
---------------

If you are using Superset you might be interested in enabling the
backups for Superset "meta" database which stores the Superset
application metadata. You can enable these backups by setting
`postgresql_backup_enabled: true` and setting S3-related secrets in
your vault file to send backups to an AWS S3 account.

If S3 is enabled, backups are created once a week and are uploaded to S3
immediately. You may create an AWS S3 Lifecycle Policy to delete old
logs on a schedule suitable to you.

Backups for the CommCare Data Pipeline database can also be done, but it
is not required as the data is already present in CommCare HQ. If you
want to add the CommCare Data Pipeline database as well, you may update
`postgresql_backup_db_list` to 'superset commcare_sync'.

To deploy the changes, either do a full stack deploy, or a one-off
deploy using:

```shell
ansible-playbook -i inventories/your-environment \
    commcare_sync.yml \
    --vault-password-file path-to-vault-password \
    --tags=postgres_backup \
    --diff
```
