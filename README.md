Gopherbot
=========

A role for installing **Gopherbot** on a CentOS7/Amazon Linux 2 host, with most of the bits needed to update it's configuration from a git repository (the standard install).

Role Variables
--------------

These are the required roles vars and overridable defaults:
```yaml
## Required Settings and suggested values
# gopherbot_config_directory: "/usr/local/etc/gopherbot"
# gopherbot_home: "/home/gopherbot"
# gopherbot_user: "gopherbot"
# gopherbot_group: "{{ gopherbot_user }}"
# gopherbot_encrypt_brain: false # required if no brain key supplied

## Defaults that can be overridden
gopherbot_version: "v2.0.0-snapshot"
gopherbot_platform: "linux"
gopherbot_force_update: false
gopherbot_install_directory: "/opt/gopherbot"
gopherbot_brain_directory: "/var/lib/gopherbot/brain"
gopherbot_history_directory: "/var/log/gopherbot/history"
gopherbot_workspace_directory: "/var/lib/gopherbot/workspace"
gopherbot_encrypt_brain: true # see above
gopherbot_extra_args: # args to the binary supplied in the systemd unit file
gopherbot_service_name: "gopherbot"
```

This should be protected in a vault-encrypted file:
```yaml
# Array of environment strings for protected values; see conf/gopherbot.yaml for
# references to 'env "GOPHER_<something>"'.
vault_gopherbot_environment:
- "GOPHER_SLACK_TOKEN=xoxb-123thisisntmykey"
- "GOPHER_BRAIN_KEY=thisneedstobeatleastthirtytwocharslong"
```

Note that `gopherbot_force_update` should normally be used on the `ansible-playbook` command line, e.g. `-e gopherbot_force_update=true` would update the installed archive.

Requirements
------------

To avoid complex configuration around the service account user, your playbook should include a role or plays for creating the user and group that will be used by **Gopherbot**.

Example Playbook
----------------

This example playbook is from the LinuxJedi.org Ansible project for deploying Gopherbot. You can see the entire repository at: https://github.com/parsley42/deploy-gopherbot


```yaml
---
- hosts: all
  connection: local
  gather_facts: false
  tasks:
  - fail: msg="This playbook requires '-e target=<targethost>'"
    when: target is undefined

- hosts: "{{ target }}"
  name: Install Gopherbot
  remote_user: build
  become: yes
  tasks:
  - name: Create robot private group
    group:
      name: "{{ gopherbot_group }}"
      system: yes
  - name: Create user for robot
    user:
      name: "{{ gopherbot_user }}"
      system: yes
  - import_role:
      name: lnxjedi.gopherbot
```

To download the role, create this `requirements.yaml` and run `ansible-galaxy install -r requirements.yaml`:

```yaml
---
- src: https://github.com/lnxjedi/ansible-role-gopherbot
  version: master
  name: lnxjedi.gopherbot
```

License
-------

MIT

Author Information
------------------

This role was written by David Parsley, the principle author of **Gopherbot**.
