self-update
=========

A simple package to setup target server to fire a deploy request to ansible command server.

The full situation:
- We do not build any package server (yum/debian package server) to maintain the target servers software upgrading. 
- We only use ansbile commands to upgrade target server and setup environment.
- We must make sure target server with the latest package and currect environment when using AWS Auto Scaling.

The Solution:
- Let target server fire a deploy request to the deploy server at boot.
- Using Ansible Dynamic Inventory to handle dynamic IP.

Requirements
------------

At default setup, the target server is running on Ubuntu 14.04 with packages: jq, curl. Using curl & jq to get target server public ip: 

```bash
$ curl -s ipinfo.io | jq '.ip'
```

At deploy server which running ansible commands, it shoud have an account which target server cloud do it to do ssh login.

```bash
$ hostname
DEPLOY_SERVER
$ id
uid=1001(boot-update) gid=1001(boot-update) groups=1001(boot-update)
$ tree $HOME/.ssh
/home/self-update/.ssh
├── ansible-deploy.pem
└── authorized_key
$ tree $HOME/ansible-deploy
ansible-deploy/
├── bin
│   └── echo.sh
├── roles
│   └── self-update
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   ├── boot-update.id_rsa
│       │   ├── dynamic_host_executable_script.sh
│       │   └── echo.sh -> dynamic_host_executable_script.sh
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   └── selfupdate.sh.j2
│       └── vars
│           └── main.yml
└── target-server.yml
```

At target server:

```bash
$ hostname
TARGET_SERVER
$ tree $HOME/.ssh
/home/ubuntu/.ssh
├── authorized_keys
├── known_hosts
└── self-update.pem
$ ssh -i $HOME/.ssh/self-update.pem self-update@DEPLOY_SERVER "date"
```

Role Variables
--------------

# Default self-update script location
selfupdate_script_location: "/usr/local/bin/selfupdate-service.sh"

# Append self-update script to boot script
selfupdate_script_boot_file: "/etc/rc.local"

# When true: append self-update script to boot script
selfupdate_script_install_boot: true

# format: \"SERVER_IP1\",\"SERVER_IP2\"
# It always cloud be a command: 
#   $ curl -s ipinfo.io | jq '@text "\\\"\(.ip)\\\""' -r 
#   \"SERVER_IP\"
#   $ export selfupdate_self_public_ip=\\\"`curl -s ipinfo.io | jq '@text "\(.ip)"' -r`\\\"
#   $ echo $selfupdate_self_public_ip
#   \"SERVER_IP\"
selfupdate_public_ip_or_get_public_ip_command: "\"SERVER_IP\""

# Deploy server location
selfupdate_ansible_command_server_location: "localhost"
# Deploy server user & login info
selfupdate_ansible_command_server_login_user: "boot-update"
selfupdate_ansible_command_server_loing_private_key: "{{selfupdate_key_install_path}}"
# Deploy workspace & deploy info
selfupdate_ansible_command_server_workspace: "\\$HOME/ansible-deploy"
selfupdate_ansible_command_server_dynamic_host_executable_script_path: "bin/echo.sh"
selfupdate_ansible_command_server_playbook_yml: "test.yml"
selfupdate_ansible_command_server_host_in_playbook_yml: "localhost"
selfupdate_ansible_command_server_private_key_for_playbook_usage: "\\$HOME/.ssh/ansible-deploy.pem"

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

- Original : Yuan-Yi Chang
