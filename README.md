Gopherbot
=========

A role for installing **Gopherbot** on a CentOS7/Amazon Linux 2 host, with most of the bits needed to update it's configuration from a git repository (the standard install).

Role Variables
--------------

These are the roles vars and defaults:
```yaml
## Required Settings and suggested values
# gopherbot_config_directory: "/usr/local/etc/gopherbot"
# gopherbot_home: "/home/gopherbot"
# gopherbot_user: "gopherbot"
# gopherbot_group: "{{ gopherbot_user }}"
# gopherbot_slack_token:
# gopherbot_encrypt_brain: false # required if no brain key supplied

## Optional / Suggested settings
# gopherbot_brain_key: # string at least 32 chars long, or leave blank and supply interactively
## For using a DynamoDB brain, AWS credentials
# gopherbot_dynamo_table:
# gopherbot_dynamo_key:
# gopherbot_dynamo_secret:
# gopherbot_dynamo_region: us-east-1

## Defaults that can be overridden
gopherbot_version: "v1.1.5"
gopherbot_platform: "linux"
gopherbot_force_update: false
gopherbot_install_directory: "/opt/gopherbot"
gopherbot_brain_directory: "/var/lib/gopherbot/brain"
gopherbot_history_directory: "/var/log/gopherbot/history"
gopherbot_workspace_directory: "/var/lib/gopherbot/workspace"
gopherbot_encrypt_brain: true # see above
gopherbot_extra_args:
gopherbot_service_name: "gopherbot"
gopherbot_local_port: 8880
gopherbot_alias: ; # A single-char alias for the robot, one of: *+^$?\[]{}&!;:-%#@~<>/
```

Note that `gopherbot_force_update` should normally be used on the `ansible-playbook` command line, e.g. `-e gopherbot_force_update=true` would update the installed archive.

Requirements
------------

To avoid complex configuration around the service account user, your playbook should include a role or plays for creating the user and group that will be used by **Gopherbot**.

Advanced Example Playbook
-------------------------

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
  - name: Create ssh directory
    file:
      path: "{{ gopherbot_home }}/.ssh"
      owner: "{{ gopherbot_user }}"
      group: "{{ gopherbot_group }}"
      state: directory
      mode: 0700
  - name: Clone the config repository
    block:
    - yum:
        name: git
        state: present
    - file:
        path: "{{ gopherbot_config_directory }}"
        state: directory
        owner: "{{ gopherbot_user }}"
        group: "{{ gopherbot_group }}"
        mode: 0750
    - git:
        repo: "{{ gopherbot_config_repository }}"
        dest: "{{ gopherbot_config_directory }}"
      become_user: "{{ gopherbot_user }}"
  - name: Install the robot's SSH key
    copy:
        src: "files/{{ inventory_hostname }}/id_rsa"
        dest: "{{ gopherbot_home }}/.ssh/id_rsa"
        owner: "{{ gopherbot_user }}"
        group: "{{ gopherbot_group }}"
        mode: 0600
  - name: Install the robot's SSH config
    copy:
        src: "files/{{ inventory_hostname }}/ssh-config"
        dest: "{{ gopherbot_home }}/.ssh/config"
        owner: "{{ gopherbot_user }}"
        group: "{{ gopherbot_group }}"
        mode: 0600
  - import_role:
      name: lnxjedi.gopherbot
```

License
-------

MIT

Author Information
------------------

This role was written by David Parsley, the principle author of **Gopherbot**.
