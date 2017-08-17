# bootstrap

Sets up a systemd service called `ansible-bootstrap` which will clone/pull and
run a specified Ansible playbook on boot.

## Usage

Generally best used in conjuction with a tool like Packer to build images with pre-loaded
configurations.

There are four key variables which control thsi:

* `bootstrap_playbook_repo` - SSH or HTTP(S) git URL for the playbook (optional)
* `bootstrap_playbook_vars` - Writes this dictionary to a playbook vars file (optional)
* `bootstrap_playbook_keyfile` - Path to a private key if the git repo is not publicly available.
* `bootstrap_playbook_content` - For simple playbooks. Not to be used with `bootstrap_playbook_repo`.
  * Example: ```
  - name: Dummy playbook
    hosts: all
    connection: local
    become: true
    tasks:
      - debug:
          msg: "Empty bootstrap playbook is empty..."
```
* `bootstrap_playbook_requirements` - Replaces a `requirements.yml` file when using `bootstrap_playbook_content`.

### Example Using Packer
#### packer.json
```json
{
  "description": "Some Image, provisioned with Ansible",
  "builders": [ ... ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "curl https://bootstrap.pypa.io/get-pip.py | sudo python -",
        "sudo yum install -y python-devel gcc gcc-c++ make libffi-devel",
        "sudo pip install wheel",
        "sudo pip install ansible~=2.3"
      ]
    },
    {
      "type": "ansible-local",
      "playbook_file": "playbook.yml",
      "galaxy_file": "requirements.yml"
    }
  ]
}
```

#### playbook.yml
```yaml
---
- name: Some Image
  hosts: all
  connection: local
  become: true
  vars:
    bootstrap_playbook_repo: 'https://github.com/inhumantsar/ansible-play-someimage.git'
    bootstrap_playbook_vars: {}
  roles:
    - inhumantsar.bootstrap
```

#### requirements.yml
```yaml
- src: inhumantsar.bootstrap
```
