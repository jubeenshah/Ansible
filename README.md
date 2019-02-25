# Ansible

## Table of Content
1. [Loops, Variables, and Return Values](#1-loops-variables-and-return-values)
2. 

### 1. Loops, Variables, and Return Values
* [Variable Overview](#variable-overview)
* [Variable Precedence](#variable-precedence)
* [Variable Scope](#variable-scope)
* [Variable Management](#variable-management)
* Registered Variables
* Inclusion
* Loops
* Ansible Return Values

#### Variable Overview
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
Three levels

| Scope | Definition |
|-------|-----------|
| Global | * Set by configuration, environment variables, command line |
| Play | * Set by playbook, play<br> *  Defined by `vars`, `include`, `include_vars`|
| Host | * Set at host level <br> * Example: `ansible_user` defines user to connect with on managed host |

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
