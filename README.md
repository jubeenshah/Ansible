# Ansible

## Table of Content
1. [Loops, Variables, and Return Values](#1-loops-variables-and-return-values)
2. [Roles](#2-roles)

### 1. Loops, Variables, and Return Values
* [Variable Overview](#variable-overview)
* [Variable Precedence](#variable-precedence)
* [Variable Scope](#variable-scope)
* [Variable Management](#variable-mangement)
* [Registered Variables](#registered-variables)
* [Inclusion](#inclusion)
* [Loops](#loops)
* [Ansible Return Values](#ansible-return-values)

#### Variable Overview
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|

* `Variable:` String or number that gets assigned value
* Lets you reuse information across:
	* Playbooks
	* Inventory files
	* Tasks and roles
	* Jinja2 template files
* Use variables to define:
	* Users
	* Packages
	* Services
	* Files
	* Archives
* `Variable Names` must begind with letter, and can include letters, digits, and underscores
* You can use `Arrays`to define multiple variables for different users like below


```yaml
---
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
...
```
* Accssing variables from array can be done as follows:
	* `users.bjones.first_name`
* Options for variables in playbooks:
	* Define in vars block at beginning of playbook
	
```yaml
- hosts: all
vars:
  user: joe
  home: /home/joe
```
	* Pass as arguments by including file in playbook
	
```yaml
include: extra_args.yml
name: joe
group: wheel
```

#### Variable Precedence
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|
>	 If Ansible finds variables with same name, uses chain of precedence
>	 Ansible 2: Variables evaluated in 16 categories of precedence order



1. Role Default Variables:
		* Set in roles `vars` directory
		
```yaml
[configuration]
users:
 - joe
 - jane
 - bob
 ```
2. Inventory Variables:
	
```yaml
[host_group]
demo.exmample.com ansible_user: joe
``` 
3. Inventory `group_vars` variables: 
	    
```yaml
[hostgroup:children]
host_group1
host_group2

[host_group:vars]
user: joe
```
4. Inventory `host_vars` variables:
	
```yaml
[hostgroup:vars]
user: joe
```
5. `group_vars` variables defined in `group_vars` directory: 	
	
```yaml
--
user: joe
```
		
6. `host_vars` variables defined in `host_vars` directory:
	
```yaml
--
user: joe
```

7. Host facts, i.e facts discoverable by Ansible
8. Registered variables, registered with `register` keyword

```yaml
---
- hosts: all
tasks:
- name: Checking if Sources are Available
  shell: echo "This is a test"
  register: output
```
9. Variables defined via `set_fact`:

```yaml
- set_fact:
  user: joe
```
	
10. Variables defined with `-a` or `--args`:

```shell
ansible-playbook main.yml -a "user=joe"
```
	
11. `vars_prompt` variables:
	
```yaml

vars:
from: "user"
  vars_prompt:
  - name: "user"
    prompt: "User to create"
```
	
12. Variables included using `vars_files`:

```yaml
vars_files:
- /vars/environment.yml
```

13. `role` and `include`variables:

```yaml
---
- hosts: all
  roles:
- { role: user, name: 'joe' }
  tasks:
- name: Includes the environment file and sets the variables
  include: tasks/environment.yml
  vars:
    package: httpd
    state: started
```
	
14. Block variables
	* For tasks defined in `block` statement

```yaml
tasks:
- block:
  - yum: name={{ item }} state=installed
    with_items:
      - httpd
      - memcached
```

15. Task variables
	* Only for task itself:
	
```yaml
- user: name=joe
```

16. `extra` variables
	* Precedence over all other variables

```shell
ansible-playbook users.yml -e "user=joe"
```

#### Variable Scope
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|
Three levels

| Scope | Definition |
|-------|-----------|
| Global | <li> Set by configuration, environment variables, command line </li> |
| Play | <li> Set by playbook, play</li> <li>  Defined by `vars`, `include`, `include_vars`</li>|
| Host | <li> Set at host level </li><li> Example: `ansible_user` defines user to connect with on managed host </li>|

* Scopes let you determine best variable placement
* To define variable only for playbook, use vars block
	* Variable evaluated when playbook is played:
```yaml
vars:
  user: joe
  ```
* To make variable available for host independent of play, define as group variable in inventory file:

```yaml
[servers]
demo.example.com

[servers:vars]
user: joe
```

* To enable variable to override playbook, declare as extra:

```shell
[user@demo ~]$ ansible-playbook users.yml -e 'user=joe'
```

#### Variable Mangement
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|
* Declare playbook variables in various locations:
	* In inventory file as host or group variables
	* In vars statement as playbook variables
	* In register statement
	* Passed as arguments using `-a`
	* Passed as extra arguments using `-e`

Example 1: Value Varying by Datacenter

```yaml
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenter1:vars]
package=httpd

[datacenter2:vars]
package=apache
```

Example 2: Value Varying by Host

```yaml
[datacenter1]
demo1.example.com package=httpd
demo2.example.com package=apache

[datacenter2]
demo3.example.com package=mariadb-server
demo4.example.com package=mysql-server

[datacenters:children]
datacenter1
datacenter2
```

Example 3: Default Value that Host Overrides

```yaml
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenters:children]
datacenter1
datacenter2

[datacenters:vars]
package=httpd
```

```shell
[user@demo ~]$ ansible-playbook demo2.exampe.com main.yml -e "package=apache"
```

#### Registered Variables
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|

* To capture command’s output, use `register` statement

```yaml
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
    - name: Install the package
      yum:
        name: httpd
        state: installed
      register: install_result
        - debug: var=install_result
```
```shell
[uder@demo ~]$ ansible-playbook playbook.yml
PLAY [Installs a package and prints the result] ****************************

TASK [setup] ***************************************************************
ok: [demo.example.com]

TASK [Install the package] *************************************************
ok: [demo.example.com]

TASK [debug] ***************************************************************
ok: [demo.example.com] => {
    "install_result": {
        "changed": false,
        "msg": "",
        "rc": 0,
        "results": [
            "httpd-2.4.6-40.el7.x86_64 providing httpd is already installed"
        ]
    }
}

PLAY RECAP *****************************************************************
demo.example.com : ok=3 changed=0 unreachable=0 failed=0
```

#### Inclusion
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|
* Tasks included with `include` executed same as if defined in playbook
* Variables included with `include_vars` parsed same as if declared in playbook
* Using multiple external files for tasks and variables:
	* Lets you build main playbook in modular way
	* Facilitates reuse of elements across playbooks
	* To import variables, use `include_vars`:

```yaml
---
- hosts: all
  tasks:
  - name: Includes the tasks file and defines the variables
    include_vars: variables.yml

  - name: Debugs the variables imported
    debug:
      msg: "{{ packages['web_package'] }} and {{ packages.db_package }} have been imported"
```
```shell
[user@prompt ~]$ ansible-playbook playbook.yml
PLAY ***********************************************************************

TASK [setup] ***************************************************************
ok: [demo.example.com]

TASK [Includes the tasks file and defines the variables] *******************
ok: [demo.example.com]

TASK [Debugs the variables imported] ***************************************
ok: [demo.example.com] => {
    "msg": "httpd and mariadb-server have been imported"
}

PLAY RECAP *****************************************************************
demo.example.com : ok=3 changed=0 unreachable=0 failed=0
```
* Variables defined as Python dictionary
* Same methods for accessing values as facts or array-based variables
	* Dot notation: `packages.db_package`, `packages.web_package`
	* Bracket notation: `packages['db_package']`, `packages['web_package']`

#### Loops
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|
* Loops require use of arrays
	* You define array and task that iterates over array
	* Loop iterates over values defined in array
* To pass loop as argument, use item keyword
	* Enables Ansible to parse array
* Ansible supports number of loop types:
	* Simple loops
		* List of items Ansible reads and iterates over
		* To define simple loop, provide list of items to `with_items`

```yaml
- yum:
    name: "{{ item }}"
    state: latest
  loop:
    - postfix
    - dovecot
```

	* Lists of hashes
		* Arrays passed as arguments can be list of hashes
		* Example: Pass multidimensional array (array with key/pair values) to `user` module

```yaml
- user:
    name: {{ item.name }}
    state: present
    groups: {{ item.groups }}
  loop:
    - { name: 'jane', groups: 'wheel' }
    - { name: 'joe', groups: 'root' }
```

#### Ansible Return Values
|[Variable Overview](#variable-overview)|
[Variable Precedence](#variable-precedence)|
[Variable Scope](#variable-scope)|
[Variable Management](#variable-mangement)|
[Registered Variables](#registered-variables)|
[Inclusion](#inclusion)|
[Loops](#loops)|
[Ansible Return Values](#ansible-return-values)|

| Value | Description |
|------|------------|
| `changed` | <li>Boolean indicating if task had to make changes</li> |
| `failed` | <li>Boolean indicating if task failed </li> |
| `msg` | <li>String with generic message relayed to user </li>|
| `results` | <li>Loop was present for task </li> <li>It contains list of normal module `result` per item</li> |
| `stderr` | <li>Contains error output of command-line utilities </li> <li> Utilities executed by some modules to run commands (`raw`, `shell`, `command`)</li> |
| `ansible_facts` | <li>Contains dictionary appended to facts assigned to host </li> <li> Facts are directly accessible </li>|
| `warnings` | <li>Contains list of strings presented to user </li> |
| `deprecations` | <li>Key containing list of dictionaries presented to user </li>|


### 2. Roles
* [Roles Overview](#roles-overview)
* [Roles in Playbooks](#roles-in-playbooks)
* [Role Creation](#role-creation)
* [Ansible Galaxy](#ansible-galaxy)


#### Roles Overview
|[Roles Overview](#roles-overview)
|[Roles in Playbooks](#roles-in-playbooks)
|[Role Creation](#role-creation)
|[Ansible Galaxy](#ansible-galaxy)|

_Role Uses:_

* Enable Ansible to load components from external files:
	* Tasks
	* Handlers
	* Variables
* Associate and reference:
	* Static files
	* Templates

```shell
[user@host roles]$ tree user.example
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
└── main.yml
```

|Subdirectory|Function|
|-------------|---------|
|`defaults`|<li>`main.yml` contains default values for role variables</li><li>Values can be overwritten when role is used</li><li>Lowest priority</li>|
|`files`|<li>Contains static files referenced by role tasks</li>|
|`handlers`|<li>`main.yml` contains role handler definitions</li>|
|`meta`|<li>`main.yml` defines role information</li><li>Includes author, license, platforms, optional dependencies</li>|
|`tasks`|<li>`main.yml` contains role task definitions</li>|
|`templates`|<li>Contains `Jinja2` templates referenced by role tasks</li>|
|`tests`|<li>Can contain inventory and `test.yml` playbook</li><li>Used to test role</li>|
|`vars`|<li>	`main.yml` defines role variable values</li><li>High Priority</li>|

#### Roles in Playbooks
|[Roles Overview](#roles-overview)
|[Roles in Playbooks](#roles-in-playbooks)
|[Role Creation](#role-creation)
|[Ansible Galaxy](#ansible-galaxy)|
* Simple to use roles in playbooks
* Example:

```yaml
---
- hosts: remote.example.com
  roles:
    - role1
    - role2
 ```
 
Order of Execution
* Default: Role tasks execute before tasks of playbooks in which they appear
* To override default, use pre_tasks and post_tasks
	* pre_tasks: Tasks performed before any roles applied
	* post_tasks: Tasks performed after all roles completed

```yaml
---
- hosts: remote.example.com
  pre_tasks:
    - shell: echo 'hello'
  roles:
    - role1
    - role2
  tasks:
    - shell: echo 'still busy'
  post_tasks:
    - shell: echo 'goodbye'
```

#### Role Creation
|[Roles Overview](#roles-overview)
|[Roles in Playbooks](#roles-in-playbooks)
|[Role Creation](#role-creation)
|[Ansible Galaxy](#ansible-galaxy)|

* Ansible looks for roles in:
	* `roles` subdirectory
	* Directories referenced by `roles_path`
		* Located in Ansible configuration file
		* Contains list of directories to search
	* Each role has directory with specially named subdirectories
* Use in Playbook
	* To access role, reference it in roles: playbook section
	* Example: Playbook referencing motd role
		* No variables specified
		* Role applied with default variable values:

```shell
[user@host ~]$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true

  roles:
    - motd
```

#### Ansible Galaxy
|[Roles Overview](#roles-overview)
|[Roles in Playbooks](#roles-in-playbooks)
|[Role Creation](#role-creation)
|[Ansible Galaxy](#ansible-galaxy)|

* [https://galaxy.ansible.com](https://galaxy.ansible.com)
* Library of Ansible roles written by Ansible administrators and users
* Archive contains thousands of Ansible roles
* Database helps users identify helpful roles for accomplishing task
* Includes links to documentation and videos for users and developers
