System Administration
=====================

# Philosophy

CommCare Sync deployments use an "[infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)"
philosophy which means all configuration is documented in code files.
See the sections below to access everything needed to configure the system.

# Services

The complete list of services 
[is available in the roles/commcare_sync/tasks/main.yml](https://github.com/dimagi/commcare-sync-ansible/blob/master/roles/commcare_sync/tasks/main.yml) file.

The most important ones are summarized on the [what's installed page](/whats-installed/).

# Common Tasks

Some of the common tasks needed to manage an environment.

## Enabling/Disabling Public Sign Ups

Sign ups are currently configured via the Django `ACCOUNT_ADAPTER` setting.
To enable anyone to sign up, you should set it to `EmailAsUsernameAdapter`.
To disable public account creation, set it to `NoNewUsersAccountAdapter`.
This behavior is controlled by the `django_allow_public_signups` Ansible variable.

If public sign ups are disabled, then only superusers can create new accounts, via the Django admin UI or command line.

## Deploying Changes

You may wish to deploy updates to the server, for example to pull the latest changes from the CommCare Sync code.

The command to deploy updates is:

```
ansible-playbook -i inventories/yourapp commcare_sync.yml --vault-password-file /path/to/password/file -vv --tags=deploy
```

Changes can also be deployed from the [CommCare Sync codebase](https://github.com/dimagi/commcare-sync)
itself by following the [instructions in the README](https://github.com/dimagi/commcare-sync#deployment).


## Accessing the Virtual Environment

To access the virtual environments for commcare sync and superset, run the following commands:

```
source <venv>/bin/activate
source <venv>/bin/postactivate
```

On most servers the commcare-sync <venv> is at `~/www/.virtualenvs/commcare-sync` and
the superset <venv> is at `~/www/.virtualenvs/superset`. 

The second command is required to set the project-specific environment variables to the correct values.

## Other Useful commands

| Task                              | Command |
| --------------------------------- | ----------- |
| See running supervisor processes  | `sudo supervisorctl status` |
| Restart a supervisor processes    | `sudo supervisorctl restart [process name]` |
| Connect to database               | `sudo -u postgres psql` |
| Restart Nginx                     | `sudo service nginx restart |


## Checking for Stuck Exports

The logs for running exports don't show up in the UI until they complete.
The easiest way to see if an export is still running or if it is stuck/was killed is to
login to the server and just run a command to see what "commcare-export" processes are running. E.g.

```
ps -ef | grep commcare-export
```

# Database Management

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
