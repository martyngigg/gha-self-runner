# Provision a self-hosted GitHub runner on an Openstack cloud

A set of Ansible scripts for managing a self-hosted GitHub action runner.
It deploys the action runner on the new host and installs Docker for running
the workflows in a container.

## Set Up

You will need an installation of Python 3.9+ available on the `PATH`.

### Openstack

On the Openstack cloud web interface:

- Add an SSH key pair registered under `Project->Compute->Key Pairs` of the
  Openstack web interface. The name is referred to as `KEY_NAME` below.
- Create an application credential to access the project from the commandline
  without a password:
  - Under `Identity->Application Credentials` click
    "Create Application Credential" and enter just a name and click the "Create
    Application Credential" button at the bottom.
  - On the following screen click "Download clouds.yaml" and move it to
    `~/.config/openstack/clouds.yaml`.

### Local machine

Check ``~/.config/openstack/clouds.yaml`` and make a note of the name of
the key in the ``clouds:`` list, the default is usually `openstack`.
This is referred to as `CLOUD_ID` below.

Provisioning requires `Ansible` and the `openstacksdk`.
It is recommended to install these tools inside a Python virtual environment
to avoid clashing with system package versions.
To create a new environment in the `./venv` directory and
install the necessary packages run:

```sh
>python -m venv ./venv
>source ./venv/bin/activate
>pip install -r ./requirements.txt
```

## Provision the runner

Provisioning the runner is a two step process:

- Create a new instance on the Openstack cloud (see [vars/gha-runner-vars.yml](./vars/gha-runner-vars.yml))
  for the instance config.
- Configuring the new instance with the necessary tools to connect to GitHub
  and support running the actions workflow.

This is all achieved through the
[gha-runner-provision.yml](./gha-runner-provision.yml) Ansible playbook.
Run the playbook with the :

```sh
ansible-playbook -i inventory.cfg -e cloud_id=CLOUD_ID -e key_name=KEY_NAME ./gha-runner-provision.yml
```
