# Provision a self-hosted GitHub runner on an Openstack cloud

A set of Ansible scripts for managing a self-hosted GitHub action runner.
It deploys the action runner on the new host and installs Docker for running
the workflows in a container.

## Set Up

You will need an installation of Python 3.9+ available on the ``PATH``.

### Openstack

On the Openstack cloud web interface:

- Add an SSH key pair registered under Project->Compute->Key Pairs of the
  Openstack web interface. Take note of the name.
- Create an application credential to access the project from the commandline
  without a password:
 


### Local machine

- A ``~/.config/openstack/clouds.yaml`` containing the authorization
  information for the Cloud API.


Provisioning requires ``Ansible`` and the ``openstacksdk``.
It is recommended to install these tools inside a virtual environment
to avoid clashing with system package versions.
To create a new environment in the ``./venv`` directory and
install the necessary packages run:

```sh
>python -m venv ./venv
>source ./venv/bin/activate
>pip install -r ./requirements.txt
```

## Provision the runner

Provisioning the runner is a two step process:

- Create a new instance on the Openstack cloud (see vars/gha-runner-vars.yml)
  for the instance config.
- Configuring the new instance with the necessary tools to connect to GitHub
  and support running the actions workflow.


