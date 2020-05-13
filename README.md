# CommCare Sync Ansible
---

Tools for setting up and deploying [commcare-sync](https://github.com/dimagi/commcare-sync).

## Setup

### Install Ansible

From the [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu),
run the following on your local/control machine

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

## Develop

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
