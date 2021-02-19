Deployment
==========

This page outlines best practices around using this repository to deploy a production server.
Details on what files need to be created can be found on the [production inventory](/production/) page.

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

