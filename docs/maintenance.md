Maintenance
===========

This shows you how to deploy CommCare Data Pipeline in a steady state as well as some other useful tasks.

### Steady-State Deploy

For existing environments you should get the relevant `myproject-ansible-vault` and `myproject.pem`
files from a project team member and jump straight to deployment.

To deploy, run the following *from your local machine*.

```bash
ansible-playbook -i inventories/myproject commcare_sync.yml --limit myserver --vault-password-file ~/myproject-ansible-vault -vv --tags=deploy
```

You can also modify the fabric example in the [app repository](https://github.com/dimagi/commcare-sync) to deploy.


### Other tasks

Some other things you might want to do on production.

#### Setting up passwordless SSH

Create `.ssh` directory in the user's home and make sure to set the permissions to 755.

```bash
mkdir ~/.ssh
chmod 755 ~/.ssh
```

Add an `authorized_keys` file and make sure to set permissions to 700.

```bash
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh/authorized_keys
```


#### Working with Superset

In order to run any superset native commands (for example `superset db upgrade`)
you must enter the superset environment and *manually run the postactivate script*.

```python
source ~/www/.virtualenvs/superset/bin/activate
source ~/www/.virtualenvs/superset/bin/postactivate
```
