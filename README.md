# Provision a self-hosted GitHub runner on an Openstack cloud

A set of Ansible scripts for managing a self-hosted GitHub action runner.
It deploys the action runner on the new host and installs Docker for running
the workflows in a container.

*Note: This is currently configured to attach the runner to a single repository.*
*Changes will be required to supported an organization level runner.*

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

Check `~/.config/openstack/clouds.yaml` and make a note of the name of
the key in the `clouds:` list, the default is usually `openstack`.
This is referred to as `CLOUD_ID` below.

### GitHub token

Configuration requires a GitHub access token with the `repo` scope for the
repository that will contain the runner.
[Create an access](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)
token and place the generated token in a file called `.env` in the same directory
as this readme formatted as `GITHUB_TOKEN=ghp-xxxxxxx`.

### Local machine

Provisioning requires `Ansible` and  `openstacksdk`.
It is recommended to install these tools inside a Python virtual environment
to avoid clashing with system package versions.
To create a new environment in the `./venv` directory and
install the necessary packages run:

```sh
>python -m venv ./venv
>source ./venv/bin/activate
>pip install -r ./requirements.txt
>ansible-galaxy install -r galaxy-requirements.yml
```

## Provision the runner

Provisioning the runner is a two step process:

- Create a new instance on the Openstack cloud (see [vars/gha-runner-vars.yml](./vars/gha-runner-vars.yml))
  for the instance config.
- Configuring the new instance with the necessary tools to connect to GitHub
  and support running the actions workflow.

This is all achieved through the
[gha-runner-provision.yml](./gha-runner-provision.yml) Ansible playbook.
Run the playbook from this directory :

```sh
source .env && ansible-playbook -u VM_USER --key-file ~/.ssh/KEY_FILE -i inventory.cfg -e cloud_id=CLOUD_ID -e key_name=KEY_NAME \
  -e github_org=GITHUB_ORG -e github_repo=GITHUB_REPO \
  -e github_token=$GITHUB_TOKEN ./gha-runner-provision.yml
```

where:

- `VM_USER` is the name of the user on the created VM that has sudo rights
- `KEY_FILE` is the name of the ssh public key file that contains the `KEY_NAME`
  added to the Openstack interface.

Once the playbook has completed you should see the runner connected at
<https://github.com/GITHUB_ORG/GITHUB_REPO/settings/actions/runners/>, where
`GITHUB_ORG` and `GUTHUB_REPO` should be replaced by the same values used in the
ansible command.

## Further Configuration

The `create_openstack_vm` role has defaults for flavor and image stored in the
[role defaults](./roles/create_openstack_vm/defaults/main.yml).
To override these pass additional `-e` variables on the `ansible-playbook`
commandline:

- `-e flavor_name=FLAVOR_NAME`
- `-e image_name=IMAGE_NAME`
