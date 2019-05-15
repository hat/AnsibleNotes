## Red Hat Certified Specialist in Ansible Automation (EX407) Preparation Course

#### Resources

* [Ansible Diagram](https://www.lucidchart.com/documents/view/4b454bc1-80ea-4753-9457-7496a5bf661e)

#### YAML Brief

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

#### Core Components of Ansible

##### Inventories

* How Ansible locates and run against multiple systems
* List of hosts
* Default stored in /etc/ansible/hosts
* May be formatted as INI file or yaml - INI format default
* Can define groups of hosts, groups of groups, group level variables
* Variables used within inventory to control how ansible connects and interacts with hosts

```
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

```
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