# SSH Config - Ansible Role

This role enables easy use of the inventory without tedious adaption of the SSH config.

# Need to know
- Automatically generates a ssh config from inventory for an ansible project so all servers are accessed through the jumpserver
- Expects exactly one jumpserver per project in inventory group `jumpserver` and will fail if it does not exist
- Generates one ssh config file per project
- Requires `ansible_host`, `ansible_user` and `ip` to be defined for all managed hosts
- All hosts in inventory group `no_auto_ssh` are ignored
- This role is not capable to use subgroups in `no_auto_ssh` (they are treated like normal groups)
- Also supports inventories created during a molecule test, see [Molecule](#Molecule)

# Getting Started

Install the role:

```sh
ansible-galaxy install ssh_config
```

Setup `inventory.yaml` correctly:

```yaml
---
all:
  children:
    jumpserver:
      hosts:
        jumphost:
          ansible_host: jumphost 
          ansible_user: debian
          ip: 10.10.10.1

    linux_server:
      hosts:
        server1:
          ansible_host: server1
          ansible_user: ubuntu
          ip: 10.10.10.2
          
    no_auto_ssh:
      hosts:
        not_managed_server1:
        # no varialbe requirements
        
```

Run playbook:

```yaml
---
- hosts: localhost
  gather_facts: true

  tasks:
    - name: Setup ssh
      vars:
        ssh_key_path: "/path/to/key"
        # Set if it differs
        ssh_config_name: "config-test"
      ansible.builtin.import_role:
        name: bwinfosec.ssh_config
```

# Molecule

Include Role at the end of `create.yml`

**Important:** `manual_mode: true` and `jumpserver` have to be set

```yaml
---
- hosts: localhost
  gather_facts: true

  tasks: 
  - name: some creation tasks
    ...

  - name: Setup ssh
    vars:
      ssh_key_path: "/path/to/key"
      ssh_config_name: config # Set if it differs
      manual_mode: true # Has to be set for jumpserver+molecule
      project_name: molecule # optional
      jumpserver: # replace credentials with the appropriate jumpserver
        hostname: ssh_hostname
        ip: 192.168.0.1
        user: ssh_user
    ansible.builtin.include_role:
      name: bwinfosec.ssh_config
```

# Troubleshooting

`ansible_user_dir` is set by default but if you have troubles change it manually to your users home directory.
Also check if `ssh_config_folder` is correct for your setup.
You find the default values in _default/main.yml_

Another option is to update your local `ansible.cfg` configuration to use newly created SSH config:

```
[ssh_connection]
ssh_args = -F ./.ssh/ssh-config -o ControlMaster=auto -o ControlPersist=30m
control_path = ./.ssh/ansible-%%r@%%h:%%p
```

