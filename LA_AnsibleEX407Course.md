## Red Hat Certified Specialist in Ansible Automation (EX407) Preparation Course

### Appendix

* [YAML Brief](#yamlbrief)
* [Core Components of Ansible](#corecomponents)
* [Ansible Commands](#commands)
* [Inventory Management](#inventorymanagement)
* [Plays and Playbooks](#playbooks)
* [Templates](#templates)



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