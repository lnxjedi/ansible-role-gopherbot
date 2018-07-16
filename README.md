Gopherbot
=========

A role for installing Gopherbot on a CentOS7/Amazon Linux 2 host

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Role files
----------
Since `Gopherbot` makes heavy use of `git` and `ssh`, you may want to supply up to two ssh keypairs:
* `files/<inventory_hostname>/robot/id_rsa*` - production ssh key, optionally encrypted
* `files/<inventory_hostname>/deploy/id_rsa` - deployment key for initial cloning of configuration repository

Dependencies
------------

To avoid complex configuration around the service account user, your playbook should include a role for creating the user.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

MIT

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
This is a VERY rough role, just imported from old 'deploy' role. Needs work.
