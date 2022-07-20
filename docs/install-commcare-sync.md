Set up CommCare Sync
=====================
This page outlines the process you need to follow in order to set up a CommCare Sync instance on a machine.

## Install Ansible

From the [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu),
run the following on your local/control machine

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

## Install CommCare Sync

### Choose a location for the control

While it is possible to run the ansible playbooks from anywhere, 
it is recommended to run them *on the server you are setting up*.
This will ensure consistency and also streamline the install process.

These instructions assume a set up where the ansible control and playbooks are run on the server being set up.

### Set up user accounts

On the server, create an account for the ansible user to use.
This can be done with the following command, answering the prompts.

```bash
sudo adduser ansible
```

### Clone the repository

```bash
git clone https://github.com/dimagi/commcare-sync-ansible.git
```

### Prepare your environment

#### Initialize Inventory Folder
Production environments live in the `inventories/` folder.
First create a new inventory folder for your environment (we'll use `myproject` as an example).
This is where your project-specific configuration will live. 

```bash
cp -r inventories/example inventories/myproject
```
#### Update Inventory Files

Edit the following files with your project-specific changes:
* `inventories/myproject/hosts.yml`
    * `vars.env`: your environment name
    * `hosts.local1.ansible_user`: username of your ansible user
    * `hosts.local1.ansible_host`: hostname/public IP where your ansible user lives
* `inventories/myproject/group_vars/commcare_sync/vars.yml`
    * `public_host`: your commcare-sync hostname
    * `superset_public_host`: your superset hostname
    * `superset_enabled`: specifies whether Superset should be installed 

#### Ansible Vault

Production environments should use [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to manage secrets.
That page has lots of details about editing and using files with Vault.

The example environment includes a vault file which you should remove:
```bash
rm inventories/myproject/group_vars/commcare_sync/vault.yml
```

#### Initial Vault Setup

The following one-time setup is used to generate keys / files for Ansible Vault.

_Generate Vault Key_

```bash
openssl rand -base64 2048 > ~/myproject-ansible-vault
```

_Create Vault Vars File_
```bash
ansible-vault create --vault-password-file ~/myproject-ansible-vault ./inventories/myproject/group_vars/commcare_sync/vault.yml
```

Add your secrets here, e.g.

```
# My Project Vault File

vault_default_db_password: <secret1>
vault_django_secret_key: <secret2>
vault_mapbox_api_key: <secret3>
vault_django_secret_key: <secret4>
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


### Run the installation scripts
```bash
ansible-galaxy install -r requirements.yml
ansible-playbook -i inventories/myproject commcare_sync.yml --vault-password-file ~/myproject-ansible-vault -vv
```

### Settting up HTTPS

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

You can set `ssl_enabled=yes` and `superset_ssl_enabled=yes` to prevent this from happening
after enabling SSL support.