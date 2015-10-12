
## Provisioning


### Server Provisioning

A set of actions to prepare a server for a given purpose.


## Deployment


### Deployment

All of the activities that make a (latest) software available for use.


## Automation

Provisioning + Deployment


### Why automation?

* code is documentation
* changes management
* all configuration files in one place
* repeatable
* resuable
* scalable


### Deployment The Old Way
 
Fabric

```
# fabfile.py

# ...

def deploy(settings='staging'):
  if settings not in SETTINGS:
    abort('Invalid settings.')
  with prefix('source ~/venv/{0}/bin/activate'.format(VENVS[settings])):
    with cd(DIRS[settings]):
      run('hg pull -u -b {0}'.format(BRANCHES[settings]))
      run('./manage.py collectstatic --noinput')
      run('./manage.py migrate --settings=settings.{0}'.format(settings))
      run('appctl restart {0}'.format(APPS[settings]))
```


### Server Provisioning With Fabric

![Automation](resources/automation.gif)


### Server Provisioning Tools

* Chef
* Puppet
* Ansible
* SaltStack
* ...



## Ansible


### Ansible

* provisioning
* deployment
* automation (orchestration)


### Ansible

* runs over SSH (default)
* Ansible-Pull (inverted the architecture) 


### Ansible

* ad-hoc commands
* inventory files
* playbooks


### Inventory

```!ini
[warsaw]
host1
host2

[carcow]
host2
host3

[poland:children]
warsaw
cracow
```


### CLI

```
$ ansible <pattern_goes_here> -m <module_name> -a <arguments>

$ ansible warsaw -i hosts.ini -m ping

$ ansible all -i hosts.ini -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```


### Playbooks

* expressed in YAML format
* composed of one or more ‘plays’
* defines machines and what remote user to run the tasks
* each play contains a list of tasks
* tasks are executed in order, one at a time, against all defined machines
* the goal of each task is to execute an Ansible module
* modules are ‘idempotent’
* each play can define handlers


### Example

```!yaml
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    apt: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```


### Running Playbook

`$ ansible-playbook playbook.yml -i hosts.ini`


## Demo


### Playbooks

* Variables
* Jinja2 templates
* Conditionals

 `when: ansible_os_family == "Debian"`

* Loops

 `with_items: [testuser1, testuser2]` 

* ...


### Modules

http://docs.ansible.com/ansible/modules_by_category.html


### Examples

https://github.com/ansible/ansible-examples



## Ansible Best Practices


### Directories structure

```
production
staging

group_vars/
   group1
   group2
host_vars/
   hostname1
   hostname2
   
library/
filter_plugins/

site.yml
webservers.yml
dbservers.yml
roles/
```


### Roles structure

```
roles/
    common/
        tasks/
            main.yml
        handlers/
            main.yml
        templates/
            ntp.conf.j2
        files/
            bar.txt
            foo.sh
        vars/
            main.yml
        defaults/
            main.yml
        meta/
            main.yml
    fooapp/
```


### Ansible Galaxy

* roles repository (open source)
* version management
* everyone can register a role


### Why Ansible Galaxy?

* real resuse across the projects
* iterative improvement
* quick bootstrap


### Our Ansible roles

https://github.com/sunscrapers/ansible-role-apt

More soon...


Example roles file:

```
sunscrapers.apt,0.0.1
ANXS.ntp,v1.1.0
ANXS.timezone,v1.0.2
ANXS.postgresql,v1.2.1
```

Roles instalation:

```
$ ansible-galaxy install -r roles.txt -p roles --force
```


### Playbook with roles 

```yaml
- hosts: app
  roles:
  - common
  - sunscrapers.apt
  - ANXS.timezone
  - ANXS.ntp
  - python
  - ANXS.postgresql
  - nginx
```


### Issues of sharing the code

* sensitive data
 * SSH keys
 * credentials
 * API keys
* sharing them securely
 * private repo
 * not keeping sensitive data in repo
 * **encrypted files**


### Ansible Vault

* encrypts var files
* secure sharing of sensitive data
* password needed to decrypt var files


### Ansible Vault

```
$ ansible-vault encrypt var_file.yml
$ ansible-vault edit var_file.yml
$ ansible-playbook main.yml -i hosts.ini --ask-vault-pass
```



## Q&A

To be continued...


## Thanks!

Follow us @sunscrapers
