Install CommCare Sync for Production
====================================

This page outlines the process you need to follow to set up a CommCare
Sync instance on a production machine.

## Choose a location for the control node

While it is possible to run the Ansible playbooks from anywhere,
it is recommended to run them *on the server you are setting up*.
This will ensure consistency and also streamline the installation
process.

These instructions assume a setup where the Ansible control node and
playbooks are run on the server being set up.

## Set up the server

Dimagi installs CommCare Sync on Ubuntu 24.04 LTS.

### SSH access

If you don't already have an SSH key pair, you'll need to create one:

```shell
ssh-keygen -t ed25519 \
    -f ~/.ssh/id_ed25519_example \
    -C "your_email@example.com"
```

This uses the Ed25519 algorithm, which is secure, fast and up-to-date.
`-f` is optional and sets a filename. Give it a name that indicates
which context it should be used in (e.g. if you have a work identity
and a personal identity) or the date when it was created. Standard
practice is to set your email address as the key's comment with `-C`.

Copy your public key to the `~/.ssh/authorized_keys` file of your user
on the remote machine:

```shell
ssh-copy-id -i ~/.ssh/id_ed25519_example.pub ubuntu@remote.ip.addr
```

(AWS can do this for you if you
[specify a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
when you launch an EC2 instance.)

### Set the hostname

*On the remote server*

```shell
sudo hostnamectl set-hostname myproject.example.com`
```

### Install Ansible

From the
[Ansible Installation Guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu),
run the following on your control machine.

```shell
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

## Install CommCare Sync

### Set up user accounts

On the server, create an account for the Ansible user to use. This can
be done with the following command, answering the prompts.

```shell
sudo adduser ansible
```

Allow the user to use `sudo` without being prompted for a password:

```shell
sudo adduser ansible sudo
cat << EOL | sudo tee /etc/sudoers.d/99-ansible
# User rules for ansible
ansible ALL=(ALL) NOPASSWD:ALL
EOL
```

Ansible will use SSH to connect to the host as the Ansible user. The
Ansible user won't need to log in using a password. Lock the password.

```shell
sudo passwd -l ansible
```

Log in as `ansible` and create an empty SSH `authorized_keys` file:

```shell
ubuntu $ sudo -iu ansible
ansible $ mkdir ~/.ssh/
ansible $ touch ~/.ssh/authorized_keys
ansible $ logout
```

Generate an SSH key pair for your user on the remote server. Do not set
a passphrase. Append the public key to the Ansible user's
`authorized_keys` file:

```shell
ubuntu $ ssh-keygen -t ed25519 -C "$USER@$(hostname -f)"
ubuntu $ cat ~/.ssh/id_ed25519.pub | sudo tee -a ~ansible/.ssh/authorized_keys
```

### Clone the repository

```shell
git clone https://github.com/dimagi/commcare-sync-ansible.git
```

### Prepare your environment

#### Initialize Inventory Folder

Production environments live in the `inventories/` folder. First create
a new inventory folder for your environment (we'll use `myproject` as
an example). This is where your project-specific configuration will
live.

```shell
mkdir -p inventories/myproject
cp -r inventories/example/* inventories/myproject
```

#### Update Inventory Files

Edit the following files with your project-specific changes:
* `inventories/myproject/hosts.yml`
    * `vars.env`: your environment name
    * `hosts.local1.ansible_user`: username of your Ansible user
    * `hosts.local1.ansible_host`: hostname/public IP where your Ansible user lives
* `inventories/myproject/group_vars/commcare_sync/vars.yml`
    * `public_host`: your commcare-sync hostname
    * `superset_public_host`: your superset hostname
    * `superset_public_hosts`: a list of superset hostnames. This overrides superset_public_host
    * `superset_enabled`: specifies whether Superset should be installed
    * `commcare_default_server`: (Optional) Full URL of HQ environment for projects. Default: https://www.commcarehq.org

#### Ansible Vault

Production environments should use
[Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
to manage secrets. That page has lots of details about editing and using
files with Vault.

The example environment includes a vault file which you should remove:
```shell
rm inventories/myproject/group_vars/commcare_sync/vault.yml
```

#### Initial Vault Setup

The following one-time setup is used to generate keys and files for
Ansible Vault.

Generate a vault key:

```shell
openssl rand -base64 2048 > ~/myproject-ansible-vault
```

Create a vault vars file:

```shell
ansible-vault create \
    --vault-password-file ~/myproject-ansible-vault \
    ./inventories/myproject/group_vars/commcare_sync/vault.yml
```

Add your secrets here, e.g.
```yaml
---

vault_default_db_password: <secret1>
vault_default_superset_password: <secret2>
vault_mapbox_api_key: <secret3>
vault_django_secret_key: <secret4>
vault_django_fernet_keys:
  # Generate with `from cryptography.fernet import Fernet; Fernet.generate_key()`
  - <secret5>
vault_superset_secret_key: <secret6>
...
```

Use your password manager to generate good random keys. (Don't have a
password manager? (WHAT?!) Try [KeePassXC](https://keepassxc.org/).)

You can edit the file later using:

```shell
ansible-vault edit \
    --vault-password-file ~/myproject-ansible-vault \
    ./inventories/myproject/group_vars/commcare_sync/vault.yml
```

### Run the installation scripts

```shell
ansible-galaxy install -r requirements.yml
ansible-playbook -i inventories/myproject commcare_sync.yml \
    --vault-password-file ~/myproject-ansible-vault -vv
```

### Setting up HTTPS

Currently this tool does not support setting up HTTPS. To set up SSL,
log into your new CommCare Sync server and install certbot:

```shell
sudo apt install certbot python3-certbot-nginx
```

Then run

```shell
sudo certbot --nginx
```

and follow the prompts.

You'll need to repeat this process for each site (e.g. commcare-sync and
superset). You may also need to open up port 443 on AWS or your
firewall.

**After setting up HTTPS you should set `ssl_enabled=yes` and
`superset_ssl_enabled=yes` in your `vars.yml` file, otherwise running a
full `ansible-playbook` will undo the changes!**


Initialize Superset
-------------------

If installation included Superset, run the following commands to
initialize its data:

```shell
superset db upgrade
superset fab create-admin
superset load_examples  # Optional: This command loads a sample dashboard
superset init
```
