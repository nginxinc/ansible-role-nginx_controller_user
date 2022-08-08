NGINX Controller User
==========================

# This repository has been archived. There will likely be no further development on the project and security vulnerabilities may be unaddressed.

Manage users with NGINX Controller.

Requirements
------------

[NGINX Controller](https://www.nginx.com/products/nginx-controller/)

Role Variables
--------------

### Required Variables

`controller.fqdn` - FQDN of the NGINX Controller instance

`controller.auth_token` - Authentication token for NGINX Controller

`nginx_controller_user.metadata.name` -  Email of the user

### Template Variables

This role has multiple template related variables. The descriptions and defaults for all these variables can be found in **[vars/main.yml](./vars/main.yml)**

Dependencies
------------

Example Playbook
----------------

To use this role you can create a playbook such as the following (let's name it `nginx_controller_user.yaml` for the purposes of this example).

```yaml
- hosts: localhost
  gather_facts: no

  vars:
    nginx_controller_user_email: "user@example.com" # Required by nginx_controller_generate_token role
    nginx_controller_user_password: "mySecurePassword" # Required by nginx_controller_generate_token role
    nginx_controller_fqdn: "controller.mydomain.com"
    nginx_controller_validate_certs: false

  tasks:
    - name: Retrieve the NGINX Controller auth token
      include_role:
        name: nginxinc.nginx_controller_generate_token

    - name: Configure the user
      include_role:
        name: nginxinc.nginx_controller_user
      vars:
        nginx_controller_user:
          metadata:
            name:  # the *email address* of the user account
            displayName:   # a friendly display name for the user (spaces and special characters allowed)
            description:  # a description of the user
            tags:   # an array of tags
            - tagOne
            - tagTwo
          desiredState:
            firstName:  # the user first name
            lastName:  # the user last name
            password: # the user password
            verifyPassword:  # the user current password.  Used when existing user updates their own account
            roles:  # reference(s) to security Roles the user is a member of
            - ref: 
            - ref: 
            groups:  # 3.7 reference(s) to security Groups the user is a member of
            - ref:   
```

Example of using a dictionary of users from a file:

controller_user_vars.yaml

```yaml
---
users:
  - 
    metadata:
      name: "ken@acmefinancial.net"
    desiredState:
      firstName: "Ken"
      lastName: "Bocchino"
      password: "<secure password>"
      roles:
      - ref: "/platform/roles/user"
  - 
    metadata:
      name: "robert@acmefinancial.net"
    desiredState:
      firstName: "Robert"
      lastName: "Teller"
      password: "<secure password>"
      roles: 
      - ref: "/platform/roles/admin"
  - 
    metadata:
      name: "president@acmefinancial.net"
    desiredState:
      firstName: "president"
      lastName: "frank"
      password: "<secure password>"
      roles:
      - ref: "/platform/roles/guest"
```

Example playbook:

```yaml
  tasks:
    # Define an array of user accounts
    - name: read the user array
      include_vars:
        file: "controller_user_vars.yaml"
        name: users

    - name: Configure array of users
      include_role:
        name: nginxinc.nginx_controller_user
      vars:
        nginx_controller_user: "{{ item }}"
      with_items: "{{ users.users }}"
```

You can then run `ansible-playbook nginx_controller_user.yaml` to execute the playbook.

Alternatively, you can also pass/override any variables at run time using the `--extra-vars` or `-e` flag like so `ansible-playbook nginx_controller_user.yaml -e "nginx_controller_user_email=user@company.com nginx_controller_user_password=notsecure nginx_controller_fqdn=controller.example.local nginx_controller_validate_certs=false"`

You can also pass/override any variables by passing a `yaml` file containing any number of variables like so `ansible-playbook nginx_controller_user.yaml -e "@nginx_controller_user_vars.yaml"`

License
-------

[Apache License, Version 2.0](./LICENSE)

Author Information
------------------

[Brian Ehlert](https://github.com/brianehlert)

&copy; [NGINX, Inc.](https://www.nginx.com/) 2020
