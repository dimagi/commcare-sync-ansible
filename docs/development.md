Development
===========

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
