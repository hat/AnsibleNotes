## Red Hat Certified Specialist in Ansible Automation (EX407) Preparation Course

### Appendix

* [YAML Brief](#yamlbrief)
* [Core Components of Ansible](#corecomponents)
* [Ansible Commands](#commands)
* [Inventory Management](#inventorymanagement)
* [Plays and Playbooks](#playbooks)
* [Templates](#templates)
* [Variables and Facts](#vars)
* [Roles](#roles)
* [Galaxy](#galaxy)
* [Parrallel](#parallel)
* [Vault](#vault)
* [Tower](#tower)
* [Documentation](#doc)



#### Resources

* [Ansible Diagram](https://www.lucidchart.com/documents/view/4b454bc1-80ea-4753-9457-7496a5bf661e)

#### <a name="yamlbrief">YAML Brief</a>

* YAML is markup languages used for formatting data
* Ansible uses YAML
* YAML composed of
  * key-value pairs
  * lists
  * dictionaries
* YAML files start with three hyphens on first line and close with three periods on last line
* List items are designated by single hyphen than space
* Each list item should have same indent
* Dictionaries designated with colon and space followed by indented key-value pairs
* Line Breaks in Ansible (| or >) with pip taking line break as part of input
* YAML Special Characters
  * []{}:>| - must use quotes to escape special characters
  * Yes and No - must be escaped in quotes if boolean not wanted
  * Floating Point Numbers taken as numeric unless quoted

#### <a name="corecomponents">Core Components of Ansible</a>

##### Inventories

* How Ansible locates and run against multiple systems
* List of hosts
* Default stored in /etc/ansible/hosts
* May be formatted as INI file or yaml - INI format default
* Can define groups of hosts, groups of groups, group level variables
* Variables used within inventory to control how ansible connects and interacts with hosts

```cli
# run the ping module on the group inside the inventory.ini file
ansible -i inventory.ini {group} -m ping
```

##### Modules

* Tools for particular tasks 
* Modules can take parameters
* Return json
* Can be run from command line or within playbook
* Default modules shipped with ansible and custom modules available
* Ansible supports custom modules

##### Variables

* Should be letters, nnumbers, underscores
* Should always start with letter
* Can be scoped by groups, host, or even in playbook
* Typically used for configuration values and various parameters
* Can store return value of commands
* Ansible variables can be dictionaries
* Number of predefined variables

##### Facts

* Facts provide information about given target host
* Facts are discovered by ansible automatically when it reaches out to host
* Fact gathering may be disabled --- does have performance hit
* Facts may be cached between playbook execution but not default

```cli
# Gathers facts from the host
ansible all -m setup
```

##### Plays and Playbooks

* Play maps group of hosts to well-defined roles
* Play may use one or more modules to achieve desired state on group of hosts
* Playbook is a series of plays
* Playbook may deploy web servers, install application, run SQL, etc...

##### Configuration Files

* Several possible places
  * ANSIBLE_CONFIG (environment variable)
  * ansiblecfg (in current directory)
  * ansible.cfg (in home directory)
  * /etc/ansible/ansible.cfg
* Configuration can be set in environment variables
* Some commonly used settings
  * ansible_managed
  * forks
  * inventory

#### <a name=commands>Ansible Commands</a>

##### Ad-Hoc Ansible Commands

* Ad-Hoc Ansible Command has the same capabilites as a playbook
* Effecticely one-liners of what to execute like a single bash command
* Three categories for use
  * Operational commands
    * checking log contents
    * Daemon control - stop/start specific daemon, etc...
    * Process management - validate process using CPU, kill process, etc...
  * Information commands
    * Check installed software
    * Check system properties
    * Gather system performance information - cpu, disk space, etc...
  * Research
    * Work with unfamiliar modules on test systems
    * Practice for playbook engineering
* Command: `ansible`
* Flags
  * `-i` inventory file to use
  * `-b` become - default is root
  * `-m` module to use
  * `-a` arguments to pass to module
  * `-f` fork for parralelism default is 5 hosts at a time
* Common Modules
  * Ping - validates server is up and reachable, no required parameters
  * Setup - gather ansible facts, no required parameters
  * Yum - uses yum package manager, 'name' and 'state' parameters required
  * Service - controls daemons, 'name' and 'state' parameters required
  * User - manipulate system users, 'name' parameter required
  * Copy - copy files, 'src' and 'dst' parameters required
  * File - works with files, 'path' parameter required
  * Git - interact with git repos, 'repo' and 'dest' parameters required

```cli
# example ad-hoc commands
ansible {host} -i {inventory} -b -m yum -a "name=elinks state=latest"
```

#### <a name="inventorymanagement">Inventory Management</a>

##### What is the Inventory

* Inventory is a list of hosts Ansible manages
* Inventory location possibilities
  * Default - /etc/ansible/hosts
  * Specified by CLI - ansible -i {inventory}
  * Set in ansible.cfg
* May contain hosts, patterns, groups, and variables
* May specify the inventory as a directory containing various inventory files (static and dynamic)
* Inventory file can be INI or YAML
* Inventories can be static or dynamic (via script)

```yaml
# Example YAML inventory
---
all:
    hosts:
        null.example.com
            ansible_port: 5556
            ansible_host: 192.168.0.10
    children:
        webservers:
            hosts:
                httpd1.example.com
                httpd2.example.com
            vars:
                http_port: 8080
        labservers:
            hosts:
                lab[01:99].example.com
```

```ini
# Example ini inventory
mail.example.com ansible_port=5556 ansible_host=192.168.0.10

[webservers]
httpd1.example.com
httpd2.example.com

[webservers:vars]
http_port=8080

[labservers]
lab[01:99].example.com
```

##### Static vs Dynamic

|   Static  |   Dyanamic    |
|   ------  |   ----------- |
|   INI or YAML format    |   Executable (bash script, python, etc...)    |
|   Maintained by hand  |     Script returns JSON containing inventory  |
|   Easy to manage  |   Useful for cloud resources and others sudden to change  |

##### Variables in Inventories

* Recommended to not store variables in inventory file
* Should store in YAML files located relative to inventory file
  * group_vars
  * host_vars

*Example of inventory variables below*

```cli
[user@computer inventory]$ cat inventory
mail.example.com

[httpd]
httpd1.example.com 
httpd2.example.com
```

```cli
[user@computer inventory]$ ls
group_vars
host_vars
inventory
```

```cli
[user@computer inventory]$ cat group_vars\httpd
http_port:8080
[user@computer inventory]$ cat host_vars/httpd1.example.com
opt_dir: /opt
```

```cli
[user@computer inventory]$ ansible httpd1.example.com -i inventory -a "ls -l {{opt_dir}}"
```

##### Dynamic Inventories

* Specifying executable as inventory file
* JSON output is expected to be returned to STDOUT
* Program/ script must respond to two possible paramters
  * --list
  * --host[hostname]
* Dynamic inventories can pull inventory from
  * Cloud provider
  * LDAP
  * Cobbler
  * other CMDB software etc...
* A lot of companies have a script available for dynamic inventories

#### <a name="playbooks">Ansible Plays and Playbooks</a>

##### Commonly Used Ansible Modules

* Working with files
  * copy
  * archive
  * unarchive
  * get_url
* user, group - modify user and group permissions
* ping - check host connectivity
* service - get info from host
* yum - used for package management
* lineinfile - drop lines into any given file
* htpasswd - modify htpasswd file
* shell, command - shell sets environment, command has no environment
* script - copies script provided, sets up shell, executes script
* debug - get from stdout and stderr and write out
* More modules at https://docs.ansible.com/ansible/latest/modules/modules_by_category.html

##### Create Playbooks

* Play maps group of hosts to well-defined roles
* Playbooks orchestrate more complex activities such as system or application deployment
* Playbooks written in YAML
* Can retry playbook on failed hosts only
* Limit allows playbook to run against specified hosts
* Watch out for spaces/ indentation, very important to Ansible

*Example of playbook below*

```yaml
---
- hosts: webservers
    remote_user: root

    tasks:
    - name: ensure apache is at the latest version
      yum: name=httpd state=latest
    - name: write the apache config file
      template: src=/srv/httpd.j2 dest=/etc/httpd.conf

- hosts: databases
    remote_user: root

    tasks:
    - name: ensure postgresql is at the latest version
      yum: name=postgresql state=latest
    - name: ensure that postgresql is started
      service: name=postgresql state=started
```

##### Variables to Retrieve Command Result

* `register` keyword saves output of play
* May be referenced within the play
* Use dot reference to extract info out of variable (JSON syntax)

*Sample below*

```yaml
---
- hosts: example.com
  tasks:
    - name: copy a file
      copy:
        src: testfile
        dest: /home/user/test
        mode: 400
      register: var
   - name: output debug info
      debug: msg="debug info is {{var}}"
```

##### Play Execution Conditionals

* Conditionals
  * when - like an if
  * with_items - like a loop with items
  * with_files - nearly identical with with_items

*below is example of with_items conditional*

```yaml
---
- hosts: local
  become: yes
  tasks:
    - name: create users
      user:
        name: "{{item}}"
      with_items:
        - sam
        - john
        - bob
```

* Handler will take action when called
* `notify` keywork calls handler
* Handler looks for `listen` keywork on handler to execute
* No matter how many times notify is called handler only called once

*example of playbook calling handler below*

```yaml
-name: template configuration file
  tasks:
    template:
      src: template.j2
      dest: /etc/foo.conf
    notify:
      -roll web
```

```yaml
handlers:
- name: restart apache
  tasks:
    service:
      name: apache
      state: restarted
    listen: "roll web"
```

##### Configure Error Handling

* Ignore acceptable errors to not stop execution
* Define failure conditions
* Define "changed"
  * `changed_when` keyword to set a change condition
* Blocks
  * Try/catch
  * `block` keyword for beginning of block
  * `rescue` keyword for what to do if block fails
  * `always` keyword for what to run regardless of block status

*Below is ignore error example - it will keep running if get_url fails on a specific host*

```yaml
---
- hosts: local
  tasks:
    - name: get files
      get_url:
        url: "http://{{item}}/index.html"
        dest: "/tmp/{{item}}"
      ignore_errors: yes
      with_items:
        - server1
        - server2
```

*Below is block groups example - also has rescue and always*

```yaml
---
- hosts: local
  tasks:
    - name: get file
      block:
        - get_url:
            url: "http://server1/index.html"
            dest: "/tmp/index_file"
      rescue:
        - debug: msg="The file doesn't exist!"
      always:
        - debug: msg="Play done!"
```

##### Tags
   
* Tags allow user to selectively run specified tasks in playbooks
* Can use `--tags` to run specific plays with tag or `--skip-tags` to run all plays without tag

*Below is an example of tags*

```yaml
---
- host: web
  become: yes
  tasks:
    - name: deploy app binary
      copy:
        src: /home/user/apps/hello
        dest: /var/www/html/hello
      tags:
        - webdeploy
- host: db
  become: yes
  tasks:
    - name: deploy db script
      copy:
        src: /home/user/apps/script.sql
        dest: /opt/deb/scripts/script.sql
      tags:
        - dbdeploy
```

*Below example of only running play with tag of dbdeploy*

```bash
ansible-playbook deploy.yml --tags dbdeploy
```

#### <a name="templates">Templates</a>

##### Template

* Skeletal file that can be dynamically completed using variables
* Mostly used for file management
* Usually uses template module within playbook
* Processed using Jinja2 language - eliminates comma errors and space errors

##### Template Module

* Template module used to deploy template modules
* Allowed to put ansible variables into Jinja2 template files
* Two required parameters
  * src - template to use on ansible host
  * dest - where file should be located on target host
* Optional parameters
  * validate - requires succesful validation command to run against file prior to deployment


*Below is example of using the template module*

```yaml
---
- hosts: webserver

    tasks:
    - name: ensure apache is at latest version
      yum: name=httpd state=latest
    - name: write the apache config file
      template: src=/srv/httpd.j2 dest=/etc/httpd.conf
```

#### <a name=vars>Variables and Facts</a>

##### Variables

* Names must start with a letter and can contain letters, numbers, underscores
* Places to define variables
  * vars, vars_files, vars_prompt
  * Command Line: ansible-playbook play.yml -e '{"myVar":"myValue","anotherVar":"anotherValue"}'
  * Roles, blocks, inventories
* YAML allows python style dictonaries and variables
* Two ways to access dictionaries
  * Bracket - employee['name']
  * Dot notation - employee.name

##### Magic Variables and Filters

* Special variables are known as magic variables
* Magic Variables
  * `hostvars`: look at facts about other hosts in inventory
    * `{{hostvars['node1']['ansible_distribution']}}`
  * `groups`: provides inventory information
    * `{{groups['webservers']}}`
* Jinja2 filters can manipulate text format
* Filters are applied by pipe `|`
* [Jinja2 filters](http://jinja.pocoo.org/docs/2.10/templates/#builtin.filters)
  * `{{groups['webservers']|join(' ')}}`

*Example of variables in playbook*

```yaml
---
- hosts: local
  vars:
    inv_file: /home/user/vars/inv.txt
  tasks:
  - name: create file
    file:
      path: "{{inv_file}}"
      line: "{{groups['labservers']|join(' ')}}"
```

##### Facts

* Information discovered by Ansible about target system
* Two ways to collect facts
  * Setup module
  * Gathered by default when playbook is executed
* Fact gathering in playbooks may be disabled using gather_facts attribute
* How to use
  * All facts may be gathered through variables
    * `ansible sample -m setup -a "filter=*ipv4"`
  * Possible to use regex, in ad-hoc mode, for pattern matching
  * Facts can be used with conditionals to have plays for different hosts
* Custom facts using facts.d
  * Create custom facts using facts.d directory `/etc/ansible/facts.d`
  * Put directory and facts on server you want the facts to be shown
  * Can be INI, JSON, or executables\
  * Create /etc/ansible directory
  * Package Ansible NOT NEEDED - just directory
  * Custom facts are in ansible_local of returned facts

#### <a name=roles>Roles</a>

##### Roles

* Roles provide a way of automatically loading certain var_files, tasks, and handlers based on a known file structure
* Not all directories required only what is to be used
* Ansible Galaxy command can create folder structure or manually creating directories
* Directory structure
  * tasks - where the plays are defined
  * vars - variable files go here
  * defaults - lowest precedence variable files go here, usually overwritten by vars or in playbooks
  * handlers - tasks that are called via the notify keyword
  * files - for ordinary files not templates (.txt, etc...)
  * template - for template files such as Jinja2 etc...
  * meta - defines metadata and configuration for a role such as dependencies, etc...

*Sample of directory structure*\

```bash
ls /etc/ansible/roles/sample_role
    tasks
        -> main.yml
        -> playbook1.yml
        -> playbook2.yml
    handlers
        -> main.yml
    files
        -> test.txt
    templates
        -> test.j2
    vars
        -> var1.yml
        -> var2.yml
    defaults
        -> defaultvar1.yml
    meta
```

##### In-Line Roles and Role Dependencies

* Include roles inside playbooks `roles:` or `include_role` module - include_role will precompile
* `dependencies` module sets dependencies on another role
* Must have `allow_duplicates: true` in metadata to have role played more than once (probably won't be used)

#### <a name=galaxy>Ansible Galaxy</a>

##### Download Roles from Ansible Galaxy

* Galaxy is a large public repository of Ansible roles
* `ansible-galaxy` binary that ships with Ansible
  * `ansible-galaxy init <role_name>`: creates new role in current directory with folder structure
  * `ansible-galaxy install <username.role>`: download role from galaxy.ansible.com
  * `ansible-galaxy list`: list current installed roles
  * remove, search, and login are also useful commands
* Can install a galaxy role in a playbook (ansible inception :P )

#### <a name=parallel>Parallelism in Ansible</a>

##### Parallelism

* Built into ansible
* Ability for control node to work on multiple hosts in parallel
* Default creates 5 forks to execute actions in parallel
* Ansible documentation recommends no more than 50 forks, 5-10 recommended for even small hosts
* Change default value in ansible.cfg
* `-f`: Flag to set fork command
* `serial`: Keyword confine number of simultaneous updated within playbook

*Example of serial keywork in playbook - will do one host, then 2, then 50% of remaining followed by other 50%*

```yaml
---
- hosts: labservers
  become: yes
  serial:
    - 1
    - 2
    - 50%
  max_fail_percentage: 30 # would consider playbook failure if more than 30% fail
  tasks:
    - name: add host entry
      lineinfile:
        path: /etc/hosts
        line: 'user1 ansible.labserver.com'
```

#### <a name=vault>Ansible Vault</a>

##### Ansible-Vault Command

* `ansible-vault encrypt` : Encrypts a file and requires password to unencrypt
* `ansible-vault rekey` : Chnage password of encrypted file
* `--vault-id` feature replaces `--ask-vault-password` and `--ask-vault-file` flags
* `no_log` will disable logging of some information
* `ansible-vault edit` : prompts password to allow editing
* `ansible-vault encrypt_string` : encrypt string to put in playbooks

##### Ansible-Vault in Playbooks

* Vault file is a text file containing only the password - will not be encrypted so secure the file
* `ansible-vault encrypt --vault-id prod@vault secure.txt` : encrypts secure.txt and assigns id of prod and uses `vault` file containing password 
* `ansible-playbook vault.yml --vault-id prod@vault` : runs playbook using 'vault' file containing password
* Unencrypted data shows on log if you don't set no_log in playbook settings

*Example of no_log using encrypted file*
```yaml
---
- hosts: localhost
  vars_files:
    - /home/user/vault/secure
  tasks:
  - name: Output mesage
    shell: echo {{message}} > /home/user/vault/deployed.txt
    no_log: true
```

#### <a name=tower>Ansible Tower</a>

##### Introduction to Ansible Tower

* Not installed by default on Ansible
* Tower provides web server interface for Ansible
* 10 hosts maximum without license
* User permission and audit trail are key benefits

##### Installing Ansible Tower

* Untar tarball and get license key from email
* README explains install steps
* Change the three password in the inventory file
* `/etc/tower/settings.py` : has some base configuration for Tower
* Once installs configures and sets up the requirements running on Port 443

##### Working with Ansible Tower

* Add Hosts, Create Playbooks, Set SCM, etc...
* Web GUI very powerful but easy to navigate
* Need to provide machine credential to run command from Tower - can use private key or client specific
* Playbook is through Project tab
* Template is task repeatable task

#### <a name=doc>Documentation</a>

##### CLI

* `ansible-doc {module}` : `-all` `-list` shows all module information

##### HTTP

* <a href="https://docs.ansible.com">docs.ansible.com</a>