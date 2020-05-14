# CommCare Sync Ansible

Tools for setting up and deploying [commcare-sync](https://github.com/dimagi/commcare-sync).

This repository uses [Ansible](https://www.ansible.com/) to provide one-click set up of commcare-sync
on a server.

## What's Installed

| Name          | Description / Reason |
| ------------- | ----------- |
| Postgres 10   | Default database |
| Redis         | Message Broker for Celery |
| Python 3.8    | Running commcare-sync |
| Nginx         | Web Server |
| Supervisord   | Process manager |
| CommCare Sync | Django process for web application, and Celery process for background and scheduled tasks |


The above will all be configured, and after install commcare-sync should be properly set up.

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

[Foollow this guide](https://www.vagrantup.com/docs/installation/)

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
ansible-vault create --vault-password-file ~/myproject-ansible-vault ./inventories/myproject/group_vars/api/vault.yml
```

Add your secrets here. E.g.

```
# My Project Vault File

vault_default_db_password: SuperSecret
```

You run the following to edit the file later:

```bash
ansible-vault edit --vault-password-file ~/myproject-ansible-vault ./inventories/myproject/group_vars/api/vault.yml
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
ansible-playbook -i inventories/myproject commcare_sync.yml --limit myserver --vault-password-file ~/myproject-ansible-vault -vv
```

This should install everything required to run CommCare Sync!

#### Settting up HTTPS

HTTPS set up is currently not supported by this tool. To set up SSL, login to your machine and run:

```
sudo certbot --nginx
```

You may also need to open up port 443 on AWS.

**Note that once you install HTTPS, running a full `ansible-playbook` will undo the changes!**
You can modify the `nginx` include in `tasks/main.yml` to prevent this.


### Steady-State Deploy

For existing environments you should get the relevant `myproject-ansible-vault` and `myproject.pem`
files from a project team member and jump straight to deployment.

To deploy, run the following *from your local machine*.

```bash
ansible-playbook -i inventories/myproject commcare_sync.yml --limit myserver --vault-password-file ~/myproject-ansible-vault -vv --tags=deploy
```

You can also modify the fabric example in the [app repository](https://github.com/dimagi/commcare-sync) to deploy.
