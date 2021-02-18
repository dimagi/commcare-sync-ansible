# CommCare Sync Ansible

Tools for setting up and deploying [commcare-sync](https://github.com/dimagi/commcare-sync).

This repository uses [Ansible](https://www.ansible.com/) to provide one-click set up of commcare-sync
on a server.

## What's Installed

The following are installed by default.

| Name          | Description / Reason |
| ------------- | ----------- |
| Postgres 10   | Default database |
| Redis         | Message Broker for Celery |
| Python 3.8    | Running commcare-sync |
| Nginx         | Web Server |
| Supervisord   | Process manager |
| CommCare Sync | Django process for web application, and Celery process for background and scheduled tasks |
| Superset      | BI Tool |

The above will all be configured, and after install, commcare-sync should be properly set up.

Note that some of these can be enabled/disabled based on variables. For example, setting `superset_enabled: no`
in your environment will prevent Superset from being installed.

## Getting Started

### Install Ansible

From the [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu),
run the following on your local/control machine

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

## Development

Development for this tool is set up to run on a Vagrant VM.
To develop and test, follow the instructions below.

### Set up Vagrant

[Follow this guide](https://www.vagrantup.com/docs/installation/)

### Deploy to local VM

*From your local machine*

```bash
vagrant up --provision
```

This should download and deploy the latest commcare-sync code to a local Vagrant VM.

If everything is successful you can load [http://192.168.11.10/](http://192.168.11.10/) in a browser
and get going!

## Production Environments

Production environments live in the `inventories/` folder.

To add a new production environment you can follow the steps below, replacing `myproject` with your project name.

Dimagi-managed production environments are maintained in the private `inventories/dimagi/` submodule.
These inventories can be accessed by running `git submodule update --init` after cloning the repository.
This requires access to the [`commcare-inventories` repository on Github](https://github.com/dimagi/commcare-sync-inventories/).

To work on a Dimagi-managed production environment, all instructions below are the same,
but the root path everywhere must be changed from `inventories/` to `inventories/dimagi/`.
Additionally, changes will need to be pushed to the separate
[`commcare-inventories` private repository](https://github.com/dimagi/commcare-sync-inventories/),
and then committed to the main repository by updating the submodule reference.

### Choose a location for the control

While it is possible to run the ansible playbooks from anywhere, 
it is recommended to run them *on the server you are setting up*.
This will ensure consistency and also streamline the install process.

These instructions assume a set up where the ansible control and playbooks are run on the server being set up.

### Set up user accounts

First login to the server and create an account for the ansible user to use.
This can be done with the following command, answering the prompts.

```bash
sudo adduser ansible
```

### Clone the repository

```bash
git clone https://github.com/dimagi/commcare-sync-ansible.git
```

### Initialize Inventory Folder

First create a new inventory folder for your environment.
This is where your project-specific configuration will live. 

```bash
cp -r inventories/example inventories/myproject
```
### Update Inventory Files

Edit the `hosts.yml` and `vars.yml` files with your project-specific changes.

### Ansible Vault

Production environments should use [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to manage secrets.
That page has lots of details about editing and using files with Vault.

The example environment includes a vault file which you can edit using:

```python
ansible-vault edit ./inventories/example/group_vars/commcare_sync/vault.yml
```

And entering the password `secret`.
You should remove this file and create a new one for your environment following the instructions below.

### Initial Vault Setup

The following one-time setup is used to generate keys / files for Ansible Vault.

#### Generate Vault Key

```bash
openssl rand -base64 2048 > ~/myproject-ansible-vault
```

#### Create Vault Vars File
```bash
ansible-vault create --vault-password-file ~/myproject-ansible-vault ./inventories/myproject/group_vars/commcare_sync/vault.yml
```

Add your secrets here. E.g.

```
# My Project Vault File
vault_default_db_password: SuperSecret
vault_django_secret_key: als0_SEcr3t
```

(You can generate a good random key from a command line:)
```
$ python3 -c 'import string
import secrets
chars = string.ascii_letters + string.digits
key = "".join(secrets.choice(chars) for x in range(64))
print(key)'
```

You run the following to edit the file later:

```bash
ansible-vault edit --vault-password-file ~/myproject-ansible-vault ./inventories/myproject/group_vars/commcare_sync/vault.yml
```

#### SSH access

Assuming you are running on AWS, *Copy the AWS private key to `~/myproject.pem` on your local machine.*

You may also need to change permissions on it.

```bash
chmod 400 ~/myproject.pem
```

Test it's working:

```bash
ssh -i ~/myproject.pem ubuntu@my.server.ip
```

#### Set hostname

*On the remote server*

`sudo hostnamectl set-hostname myproject-server`

#### Run Installation

```bash
ansible-playbook -i inventories/myproject commcare_sync.yml --vault-password-file ~/myproject-ansible-vault -vv
```

This should install everything required to run CommCare Sync!

#### Settting up HTTPS

HTTPS set up is currently not supported by this tool. To set up SSL, login to your machine and install certbot:

```
sudo apt install certbot python3-certbot-nginx
```

Then run:

```
sudo certbot --nginx
```

and follow the prompts.

You'll need to repeat this process for each site (e.g. commcare-sync and superset).
You may also need to open up port 443 on AWS or your firewall.

**After setting up HTTPS you should set `ssl_enabled=yes` and `superset_ssl_enabled=yes` in your `vars.yml` file,
otherwise running a full `ansible-playbook` will undo the changes!**


### Steady-State Deploy

For existing environments you should get the relevant `myproject-ansible-vault` and `myproject.pem`
files from a project team member and jump straight to deployment.

To deploy, run the following *from your local machine*.

```bash
ansible-playbook -i inventories/myproject commcare_sync.yml --limit myserver --vault-password-file ~/myproject-ansible-vault -vv --tags=deploy
```

You can also modify the fabric example in the [app repository](https://github.com/dimagi/commcare-sync) to deploy.


### Using the commcare-inventories submodule in production

When referencing the submodule in production environments, you should use [deploy keys](https://docs.github.com/en/developers/overview/managing-deploy-keys#deploy-keys).

Follow the instructions there, making sure to run `ssh-keygen` on the server you want to have access.

After adding a deploy key to the `commcare-inventories` repository you should be able to update the submodule as normal.


### Passwordless SSH

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


### Migrating a server

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


# Upgrading Superset

Tried [these instructions](https://superset.apache.org/docs/installation/upgrading-superset) but getting errors.

Problem was having to run postactivate on the venv. Manually running it and then running these commands fixed it:

```python
superset db upgrade
superset init
```
