System Administration
=====================

## Philosophy

CommCare Sync deployments use an "[infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)"
philosophy which means all configuration is documented in code files.
See the sections below to access everything needed to configure the system.

## Services

The complete list of services 
[is available in the roles/commcare_sync/tasks/main.yml](https://github.com/dimagi/commcare-sync-ansible/blob/master/roles/commcare_sync/tasks/main.yml) file.

The most important ones are summarized on the [what's installed page](/whats-installed/).

## Common Tasks

Some of the common tasks needed to manage an environment.

### Enabling/Disabling Public Sign Ups

Sign ups are currently configured via the Django `ACCOUNT_ADAPTER` setting.
To enable anyone to sign up, you should set it to `EmailAsUsernameAdapter`.
To disable public account creation, set it to `NoNewUsersAccountAdapter`.
This behavior is controlled by the `django_allow_public_signups` Ansible variable.

If public sign ups are disabled, then only superusers can create new accounts, via the Django admin UI or command line.


## Create superuser
Activate virtual environment of commare-sync. If you are not sure where the virtualenv path is, use the following command to find it:
    `find $HOME -name "*activate" -type f`

example output

`/home/pc/www/.virtualenvs/commcare-sync/bin/postactivate`

`/home/pc/www/.virtualenvs/commcare-sync/bin/activate`

`/home/pc/www/.virtualenvs/superset/bin/postactivate`

`/home/pc/www/.virtualenvs/superset/bin/activate`

To activate use the following command:
    `source /home/pc/www/.virtualenvs/commcare-sync/bin/activate`

And cd to installation directory: that is `~/www/commcare-sync/code_root/` and Create super user using the following command, 
 `./manage.py createsuperuser`  and follow the wizard

## Add commare server

By default, commcare-sync is preconfigured with commcarehq.org as the default site from which to pull data. If you are pulling data from a local instance of commcare, you should add the server parameters.

Steps: 
    Login to commcare-sync server using your superuser account created on the above step
    On the url append admin to see the admin panels: i.e `http://<IP>/admin`

    Then click on the comcare server link ---> Click on ADD COMM CARE SERVER ---> Add the name and your server URL and save.

## superset initialization steps
run  superset db upgrade 

run superset fab create-admin

run superset load_examples (optional. this command loads sample dashboard)

run superset init

### Deploying Changes

You may wish to deploy updates to the server, for example to pull the latest changes from the CommCare Sync code.

The command to deploy updates is:

```
ansible-playbook -i inventories/yourapp commcare_sync.yml --vault-password-file /path/to/password/file -vv --tags=deploy
```

Changes can also be deployed from the [CommCare Sync codebase](https://github.com/dimagi/commcare-sync)
itself by following the [instructions in the README](https://github.com/dimagi/commcare-sync#deployment).


### Accessing the Virtual Environment

To access the virtual environments for commcare sync and superset, run the following commands:

```
source <venv>/bin/activate
source <venv>/bin/postactivate
```

On most servers the commcare-sync <venv> is at `~/www/.virtualenvs/commcare-sync` and
the superset <venv> is at `~/www/.virtualenvs/superset`. 

The second command is required to set the project-specific environment variables to the correct values.

### Other Useful commands

| Task                              | Command |
| --------------------------------- | ----------- |
| See running supervisor processes  | `sudo supervisorctl status` |
| Restart a supervisor processes    | `sudo supervisorctl restart [process name]` |
| Connect to database               | `sudo -u postgres psql` |
| Restart Nginx                     | `sudo service nginx restart |


### Checking for Stuck Exports

The logs for running exports don't show up in the UI until they complete.
The easiest way to see if an export is still running or if it is stuck/was killed is to
login to the server and just run a command to see what "commcare-export" processes are running. E.g.

```
ps -ef | grep commcare-export
```

## Database Management

Because CommCare Sync does not provide direct access to the underlying databases,
it is common to have to perform database management tasks, for example, creating new databases,
 or dropping existing databases or tables.

Database management can be done by logging into the server and getting a postgres shell:

```
psql -U [postgres user] -h localhost -p 5432
```

You will need the postgres username and password from the secrets file,
or by finding it on the server e.g. in the django commcare_sync/local.py file.
You can also run `./manage.py dbshell` in the CommCare Sync virtual environment.


## Database Backup

If you are using Superset you might be interested in enabling the backups for Superset meta
database which stores the superset application side data. You can enable these backups by 
setting `postgresql_backup_enabled` to true and updating s3 related secrets in your vault file 
to send backups to an AWS S3 account.

Backups are created once a week and are uploaded to S3 immediately if S3 is enabled. You may create AWS S3 Lifecycle policy to delete old logs on a schedule suitable to you.

Backups for CommCare Sync DB can also be done, but it is not required as the data is already 
present in CommCareHQ. If you want to add commcare_sync DB as well, you may update 
`postgresql_backup_db_list` to 'superset commcare_sync'.

To deploy the changes, either do a full stack deploy or as a one-off command as below

```
ansible-playbook -i inventory_dir commcare_sync.yml --vault-password-file path-to-vault-password  --tags=postgres_backup --diff
```
