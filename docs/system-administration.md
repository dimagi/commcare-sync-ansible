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

### Create superuser
To create a CommCare Sync superuser you need to do the following:
1. Activate virtual environment of commare-sync. If you are not sure where the virtualenv path is you can use the following command to find it,
   ```
   find $HOME -name "*activate" -type f`
   ```

   An example of the output can be seen below (assuming the user is named `pc`),
   ```
   /home/pc/www/.virtualenvs/commcare-sync/bin/postactivate
   /home/pc/www/.virtualenvs/commcare-sync/bin/activate
   /home/pc/www/.virtualenvs/superset/bin/postactivate
   /home/pc/www/.virtualenvs/superset/bin/activate
   ```
   
   Finally, to activate the virtual environment you can run the following command,
   ```       
   source /home/pc/www/.virtualenvs/commcare-sync/bin/activate
   ```

2. Change directory into the installation directory 
   ```
   cd ~/www/commcare-sync/code_root/ 
   ```
3. Create the superuser by running the command,
   ```
   ./manage.py createsuperuser
   ```

### Add commare server

By default, commcare-sync is preconfigured with `commcarehq.org` as the default site from which to pull data. If you want to 
pull data from a local instance of commcare, you should add the server parameters.

Steps: 
1. Login to commcare-sync server using your superuser account created with the step above
2. Append `admin` to the URL to see the admin panels, i.e 
   ```
   http://<IP>/admin
   ```
3. Click on the comcare server link 
   - Click on "ADD COMM CARE SERVER"
   - Add the name and your server URL
   - Click "Save"

### Enabling/Disabling Public Sign Ups

Sign ups are currently configured via the Django `ACCOUNT_ADAPTER` setting.
To enable anyone to sign up, you should set it to `EmailAsUsernameAdapter`.
To disable public account creation, set it to `NoNewUsersAccountAdapter`.
This behavior is controlled by the `django_allow_public_signups` Ansible variable.

If public sign ups are disabled, then only superusers can create new accounts, via the Django admin UI or command line.

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
