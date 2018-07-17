Gopherbot
=========

A role for installing **Gopherbot** on a CentOS7/Amazon Linux 2 host, with most of the bits needed to update it's configuration from a git repository (the standard install).

Role Variables
--------------

These are the roles vars and defaults:
```yaml
gopherbot_version: "v1.1.5"
gopherbot_platform: "linux"
gopherbot_force_update: false
gopherbot_install_dir: "/opt/gopherbot"
gopherbot_brain_directory: /var/lib/gopherbot/brain
gopherbot_history_directory: /var/log/gopherbot/history
gopherbot_config_directory: "/usr/local/etc/gopherbot"
gopherbot_home: "{{ gopherbot_install_dir }}"
gopherbot_user: "gopherbot"
gopherbot_group: "{{ gopherbot_user }}"
gopherbot_extra_args:
gopherbot_service_name: "gopherbot"
gopherbot_local_port: 8880
gopherbot_alias: ; # A single-char alias for the robot, one of: *+^$?\[]{}&!;:-%#@~<>/
# Required
gopherbot_slack_token:
# For using a DynamoDB brain, AWS credentials
# gopherbot_dynamo_table:
# gopherbot_dynamo_key:
# gopherbot_dynamo_secret:
# gopherbot_dynamo_region: us-east-1
```

Note that `gopherbot_force_update` should normally be used on the `ansible-playbook` command line, e.g. `-e gopherbot_force_update=true` would update the installed archive.

Requirements
------------

To avoid complex configuration around the service account user, your playbook should include a role for creating the user and group that will be used by **Gopherbot**.

Advanced Example Playbook
-------------------------

This example playbook uses `singleplatform-eng.users` to create the user and group, and installs both a deploy key and (possibly encrypted) ssh private key. The public key for both of these should be set up as deployment keys for your **Gopherbot** configuration repository.

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
  become: yes
  tasks:
  - import_role:
      name: singleplatform-eng.users
  - name: Create ssh directory
    file:
      path: "{{ gopherbot_home }}/.ssh"
      owner: "{{ gopherbot_user }}"
      group: "{{ gopherbot_group }}"
      state: directory
      mode: 0700
  - name: Clone the config repository
    block:
    - copy:
        src: "files/{{ inventory_hostname }}/deploy/id_rsa"
        dest: "{{ gopherbot_home }}/.ssh/deploy_rsa"
        owner: "{{ gopherbot_user }}"
        group: "{{ gopherbot_group }}"
        mode: 0600
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
        accept_hostkey: yes
        dest: "{{ gopherbot_config_directory }}"
        key_file: "{{ gopherbot_home }}/.ssh/deploy_rsa"
      become_user: "{{ gopherbot_user }}"
  - name: Install the robot's SSH key
    copy:
        src: "files/{{ inventory_hostname }}/robot/id_rsa"
        dest: "{{ gopherbot_home }}/.ssh/id_rsa"
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
