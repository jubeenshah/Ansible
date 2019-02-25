# Ansible

## Table of Content
1. [Loops, Variables, and Return Values](#loops,-variables,-and-return-values)
2. 

### 1. Loops, Variables, and Return Values
* Variables Overview
* Variable Precedence
* Variable Scope
* Variables Management
* Registered Variables
* Inclusion
* Loops
* Ansible Return Values

#### Variables Overview
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
