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
To add a new Dimagi-managed production environment, you can follow the instructions below, changing
the root path from `inventories/` to `inventories/dimagi/`. Additionally you will need to push
changes to the [`commcare-inventories` repository on Github](https://github.com/dimagi/commcare-sync-inventories/). 

### Initialize Inventory Folder

First create a new inventory folder for your environment.
This is where your project-specific configuration will live. 

```bash
cp inventories/dev inventories/myproject
```
### Update Inventory Files

Edit the `hosts.yml` and `vars.yml` files with your project-specific changes.

### Ansible Vault

Production environments should use [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to manage secrets.
That page has lots of details about editing and using files with Vault.

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

HTTPS set up is currently not supported by this tool. To set up SSL, login to your machine and run:

```
sudo certbot --nginx
```

You'll need to repeat this process for each site (e.g. commcare-sync and superset).
You may also need to open up port 443 on AWS.

**Note that once you install HTTPS, running a full `ansible-playbook` will undo the changes!**

You can set `ssl_enabled=yes` and `superset_ssl_enabled=yes` to prevent this from happening
after enabling SSL support.

### Steady-State Deploy

For existing environments you should get the relevant `myproject-ansible-vault` and `myproject.pem`
files from a project team member and jump straight to deployment.

To deploy, run the following *from your local machine*.

```bash
ansible-playbook -i inventories/myproject commcare_sync.yml --limit myserver --vault-password-file ~/myproject-ansible-vault -vv --tags=deploy
```

You can also modify the fabric example in the [app repository](https://github.com/dimagi/commcare-sync) to deploy.
