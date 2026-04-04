Development
===========

Development for this tool is set up to run on a Vagrant VM.
To develop and test locally, follow the instructions below.

## Install Ansible

From the [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu),
run the following on your control machine.

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

## Set up Vagrant

[Follow this guide](https://www.vagrantup.com/docs/installation/)

## Deploy to local VM

*From your local machine*

```bash
vagrant up --provision
```

This should download and deploy the latest CommCare Data Pipeline code to a local Vagrant VM.

If everything is successful you can load [http://192.168.11.10/](http://192.168.11.10/) in a browser
and get going!

## Running tests with Molecule

Individual roles can be tested in isolation using
[Molecule](https://ansible.readthedocs.io/projects/molecule/), which
provisions a Docker container and runs the role against it. Tests
currently exist for the `postgres`, `redis`, and `nginx` roles.

### Install Molecule and dependencies

```bash
uv venv
source .venv/bin/activate
uv pip install -r requirements-dev.txt
```

You will also need [Docker](https://docs.docker.com/get-docker/)
installed and running.

### Run tests

To test a single role:

```bash
cd roles/postgres
molecule test
```

This will create an Ubuntu 24.04 Docker container, run the role, execute
the verification checks, and tear down the container.

To test all roles that have Molecule scenarios:

```bash
for role in postgres redis nginx; do
  (cd roles/$role && molecule test)
done
```

Tests also run automatically on push and pull request via the GitHub
Actions workflow in `.github/workflows/molecule-test.yml`.
