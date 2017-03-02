# AWS STS Role

This Ansible role allows a user to assume a given role, generating temporary security credentials that can be used to assume the role.

## Requirements

- Python 2.7
- PIP package manager (**easy_install pip**)
- Ansible 2.2 or greater (**pip install ansible**)
- AWS CLI (**pip install awscli**)

## Setup

The recommended approach to use this role is an Ansible Galaxy requirement to your Ansible playbook project.

Alternatively you can also configure this repository as a Git submodule to your Ansible playbook project. 

### Installing using Ansible Galaxy

To set this role up as an Ansible Galaxy requirement, first create a `requirements.yml` file in a subfolder called `roles` and add an entry for this role.  See the [Ansible Galaxy documentation](http://docs.ansible.com/ansible/galaxy.html#installing-multiple-roles-from-a-file) for more details.

```
# Example requirements.yml file
- src: https://github.com/docker-in-production/aws-sts.git
  scm: git
  version: 1.0.0
  name: aws-sts
```

Once you have created `roles/requirements.yml`, you can install the role using the `ansible-galaxy` command line tool.

```
$ ansible-galaxy install --role-file=roles/requirements.yml --roles-path=./roles/ --force
$ git commit -a -m "Added aws-sts 1.0.0 role"
```

To update the role version, simply update the `requirements.yml` file and re-install the role as demonstrated above.

### Installing using Git Submodule

You can also install this role by adding this repository as a Git submodule and then checking out the required version.

The submodule should be placed in the folder **roles/aws-sts**, and can then be referenced from your playbooks as a role called `aws-sts`.

You should also checkout the specific release required for your project as demonstrated below:

```
$ git submodule add https://github.com/docker-in-production/aws-sts.git
Submodule path 'roles/aws-sts': checked out '05f584e53b0084f1a2a6a24de6380233768a1cf0'
$ cd roles/aws-sts
roles/aws-sts$ git checkout 1.0.0
roles/aws-sts$ cd ../..
$ git commit -a -m "Added aws-sts 1.0.0 role"
```

You can update to later versions of this role by updating your submodules:

```
$ git submodule update --remote roles/aws-sts
$ cd roles/aws-sts
roles/aws-sts$ git checkout 2.0.0
roles/aws-sts$ cd ../..
$ git commit -a -m "Updated to aws-sts 2.0.0 role"
```
## Usage

### Inputs

The STS role relies on the following inputs:

- `sts_role_arn` (Mandatory) - this variable must be provided by the calling playbook, representing the ARN of the role to assume

- `sts_role_session_name` (Optional) - a name for the temporary session token that is generated

- `sts_disable` (Optional) - disables all actions of this role.  Useful for long running playbooks that would be affected by the duration (maximum 60 minutes) of using STS token.

- `sts_region` (Optional) - the target AWS regions.  Alternatively you can set the AWS region using the `AWS_DEFAULT_REGION` environment variable.  If this is not in your environment, the playbook defaults to `ap-southeast-2`.

- AWS credentials - you must configure the environment such that your credentials are available to assume the role.  For example, you can set an access key and secret key via environment variables, or configure a profile via environment variables, or rely on an EC2 instance profile if running in AWS.  A dictionary called `sts_creds` is output by this module, which sets up the appropriate configuration with AWS STS token settings.

### Outputs

If the assume role operation is successful, the temporary credentials issued by AWS are registered to a variable called `sts_session`:

- `sts_session.sts_creds.access_key`
- `sts_session.sts_creds.secret_key`
- `sts_session.sts_creds.session_token`

And a dictionary called `sts_creds` which you can configure as the environment for subsequent plays to operate under the privileges of the assumed role:

```
AWS_DEFAULT_REGION: "{{ sts_region }}"
AWS_ACCESS_KEY: "{{ sts_session.sts_creds.access_key }}"
AWS_ACCESS_KEY_ID: "{{ sts_session.sts_creds.access_key }}"
AWS_SECRET_KEY: "{{ sts_session.sts_creds.secret_key }}"
AWS_SECRET_ACCESS_KEY: "{{ sts_session.sts_creds.secret_key }}"
AWS_SECURITY_TOKEN: "{{ sts_session.sts_creds.session_token }}"
```

You should call this role from a dedicated play, and then define your subsequent playbook tasks in separate plays.  This allows the `sts_creds` or individual `sts_session` variables to be used to configure the environment of your remaining tasks.


## Examples

The following demonstrates how to call this role from a playbook:

```yaml
- name: Assume Role Play
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    sts_role_arn: "arn:aws:iam::123456789:role/admin"
    sts_role_session_name: testAssumeRole
    sts_region: us-west-2
  roles:
    - aws-sts
```

The following shows the recommended play configuration to use the temporary credentials issued:

```yaml
...
...
- name: My Playbook
  hosts: localhost
  connection: local
  environment: "{{ sts_creds }}"
  tasks:
   - ...
   - ...
...
...
```

## Testing

You can run the included [`test.yml`](./test.yml) playbook to test this role directly.

To test the role you need to provide the role that you want to assume as demonstrated below:

```
$ ansible-playbook test.yml -e sts_role_arn=arn:aws:iam::388576150228:role/diAdmin
PLAY [Assume Role] *************************************************************

TASK [set_fact] ****************************************************************
ok: [localhost]

TASK [checking if sts functions are sts_disabled] ******************************
skipping: [localhost]

TASK [setting empty sts_session result] ****************************************
skipping: [localhost]

TASK [setting sts_creds if legacy AWS credentials are present (e.g. for Ansible Tower)] ***
skipping: [localhost]

TASK [assume sts role] *********************************************************
ok: [localhost]

TASK [set sts facts] ***********************************************************
ok: [localhost]

TASK [set sts facts] ***********************************************************
ok: [localhost]

TASK [debug] *******************************************************************
skipping: [localhost]

TASK [debug] *******************************************************************
ok: [localhost] => {
    "msg": {
        "AWS_ACCESS_KEY": "xxxx",
        "AWS_ACCESS_KEY_ID": "xxxx",
        "AWS_DEFAULT_REGION": "ap-southeast-2",
        "AWS_SECRET_ACCESS_KEY": "xxxx",
        "AWS_SECRET_KEY": "xxxx",
        "AWS_SECURITY_TOKEN": "xxxx"
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=5    changed=0    unreachable=0    failed=0
```

## Release Notes

### Version 1.0.0

- First Release