#### lyusingansible
- Test Environment Setup
prepare three machines:
then add user `test` in all three machines,  
```
useradd test 
passwd test
```

make new dict and set user group in all three machines
```
mkdir playbook
chown test:test playbook/
```

login as test user.  

then generate ssh.
```
ssh-keygen
```


copy ssh form every machine to another two
```
ssh-copy-id <machine2 ip>
ssh <machine2ip>
ssh-copy-id <machine2 name>
ssh <machine2-name>

ssh-copy-id <machine3 ip>
ssh <machine3ip>
ssh-copy-id <machine3 name>
ssh <machine3-name>
```
- Download and Installation 
install prep software in all three machines
```
yum install epel-release
yum update repolist
sudo yum update
```

install ansible
```
yum install ansible
```

- Ansible Configuration File
```
cd /etc/ansible
```
three files:
```
ansible.cfg
hosts
roles
```

for ansible.cfg,  
uncomment log_path=/var/log/ansible.log

- Ansible Python Dependencies
```
sudo yum list installed | grep python
```

- The HOSTS File

create our own hosts file

```
vim hosts
```
edit:
```
[local]
localhost
#localhost.localdomain
#127.0.0.1

[apache]
compu.labserver.com (or inner ip)

[appserver]
```

Remember,change`ansible.cfg` edit this line:
```
inventory= /etc/ansible/hosts
```
if hosts file path changes

- Overriding the Default HOSTS File
```
ansible all --list-hosts
```
```
ansible groupname -m ping
```

create a host file locally and override host file in `/etc/ansible/hosts`
```
ansible gpname -i hosts -m ping
```

- Overriding the Default System Ansible.Cfg File
```
sudo cp /etc/ansible/snaible.cfg .
chown test:test ansible.cfg
```
```
inventory= ./hosts
```
```
export ANSIBLE_CONFIG=/custom/path
set | grep ANSIBLE
```

- Overriding the Default Roles Path
 in `/etc/ansible/hosts`
```
roles_path =role1:role2
```

- Configuring Your 'Ansible' Account
enable nopasswd in all three machines:
```
visudo
```
add
```
test ALL=(ALL) NOPASSWD:ALL
```
and edit ansible.cfg,edit   
```
#ask_sudo_pass=True
```
```
ssh-copy-id localhost.localdomain
```

====

- Ansible Command Line
```
anible -s -m shell -a 'yum list installed | grep python'
```

- System Facts
get facts
```
ansible localhost -s -m setup |more
```

get other machine's facts and save on local machine
```
ansible localhost -s -m setup --tree /localpath
```

filter (do not use grep)
```
ansible localhost -m setup -a 'filter=ansible_*'
```

- System Facts: Common Values for Playbooks  

some example
```
ansible localhost -m setup -a 'filter=ansible_domain'
ansible localhost -m setup -a 'filter=ansible_fqdn'
ansible localhost -m setup -a 'filter=ansible_interfaces'
ansible localhost -m setup -a 'filter=ansible_kernel'
ansible localhost -m setup -a 'filter=ansible_memtotal_mb'
ansible localhost -m setup -a 'filter=ansible_virt*'
```

- Our First Playbook  
review:
```
ansible all -s -m yum -a 'pkg=lynx state=installed update_cache=true'
```
convert to new.yml
```
- hosts: all  #ifmultiple hosts, use : to seperate
  tasks:
  - name: install
    yum: pkg=lynx state=installed update_cache=true
```

- Variables: Inclusion Types

first: naive way
```
- hosts: all
  vars:
    web_root: /var/www/html
  tasks:
  - name: install
    yum: pkg=lynx state=installed update_cache=true
```

modified:
```
- hosts: all
  vars_files:
  - vars.yml
  tasks:
  - name: install
    yum: pkg=lynx state=installed update_cache=true
```
vars.yml:
```
web_root: /var/www/html
```

- Target Section


- Variable Section
(vars_files) file must start with `---`
vars.yml:
```
---
apache_version: 2.6
apache_version: mod_ssl
```

cbk.yml
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  vars:
    playbook_version:0.1
  vars_files:
    - a.yml
    - b.yml
  vars_prompt:
    - name: prop_message
      prompt: Prop Message
```

- Task Section
cbk.yml
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  vars:
    playbook_version:0.1
  vars_files:
    - a.yml
    - b.yml
  vars_prompt:
    - name: prop_message
      prompt: Prop Message
  tasks:
    - name: install
      action: yum name=lynx state=installed
    - name: Check
      action: yum name=telnet state=absent
```

- Handler Section
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  vars:
    playbook_version:0.1
  vars_files:
    - a.yml
    - b.yml
  vars_prompt:
    - name: prop_message
      prompt: Prop Message
  tasks:
    - name: install
      action: yum name=lynx state=installed
      notify: Restart
  handlers:
    - name: Restart    #same as notify
      action: service name=httpd state=restarted
```

- Outlining Your Playbook
- Create a Playbook from Our Outline

ckeck syntax without run
```
ansible-playbook test.yml --check
```

practice:
```
- hosts: web
  user: test
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: log start time
    raw: /usr/bin/date > /home/test/start.log    #file save on target machine
  - name: install httpd
    action: yum name=httpd state=installed
  - name: start
    service: name=httpd state=restarted
  - name: verify
    command: systemctl status httpd
    register: result
  - debug: var=result
  - name: log all packages
    raw: yum list installed > /home/test/install.log
  - name: log end time
    raw: /usr/bin/date > /home/test/end.log    #file save on target machine
```

- Optimizing Your Playbook
```
- hosts: web
  user: test
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: log start time
    command: /usr/bin/date
    register: start_sp
  - debug: var=start_sp
  - name: install httpd
    action: yum name=httpd state=installed
    notify: start
  - name: verify
    command: systemctl status httpd
    register: result
  - debug: var=result
  - name: log all packages
    command: yum list installed    
    register: all_install
  - debug: var=all_install
  - name: log end time
    command: /usr/bin/date
    register: end_sp
  - debug: var=end_sp
  - handlers:
    - name: start
      service: name=httpd state=restarted
```


- Taking Our Playbook for a Dry Run
Review:
```
ansible-playbook test.yml --check
```

- Asychronous Polling
(run in parallel)
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  vars:
    playbook_version:0.1
  vars_files:
    - a.yml
    - b.yml
  vars_prompt:
    - name: prop_message
      prompt: Prop Message
  tasks:
    - name: install
      action: yum name=lynx state=installed
      notify: Restart
      async: 300  # max time wait to complete this command.(mills)
      poll: 3 # how often check this command has been completed. Every 3 second check it.
  handlers:
    - name: Restart    #same as notify
      action: service name=httpd state=restarted
```

- Simple Variable Substitution
a.yml
```
---
apache_version: 2.6
apache_version: mod_ssl
pkg_lynx: lynx
```


```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  vars:
    playbook_version:0.1
  vars_files:
    - a.yml
    - b.yml
  vars_prompt:
    - name: prop_message
      prompt: Prop Message
  tasks:
    - name: install
      action: yum name=lynx state=installed
      notify: Restart
    - name: install lynx
      action: yum name={{pkg_lynx}} state=installed
    - name: install package
      action: yum name={{prop_message}} state=installed  # install package when prompt
  handlers:
    - name: Restart    #same as notify
      action: service name=httpd state=restarted
```

- Lookups
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  tasks:
    - debug: msg="start opening {{lookup('csvfile','line1head file=look.csv delimiter=, default=NOMATCH') }}"
    - debug: msg="{{ lookup('env','HOME'}}"
```

- RunOnce
in a group of machines, only run on first machine.
```
.....for brevity
- name: install
      action: yum name=lynx state=installed
      notify: Restart
      run_once: true   #not yes
```

- Local Actions

run playbook only in local:
```
ansible-playbook test.yml --connection=local  (event connection=ssh)
```

```
- hosts: 127.0.0.1
  connection: local
  ...omit for brevity
```

- Loops

this playbook will create 2 users on each machine
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  tasks:
    - name: list of users
      user: name= {{item}} state=present
      with_items:
        - user1
        - user2
```

- Conditionals
```
---
- hosts: web
  usr: test
  become :yes
  become_method: sudo   (sudo: yes user:test)
  remote_user: test  
  connection: ssh
  gather_facts: no
  .... for brevity
  tasks:
    - name: install on debian
      command: apt-get -y install apache2
      when: ansible_os_family="Debian"
    - name: install on redhat
      command: yum -y install httpd
      when: ansible_os_family="Redhat"
```
- Until
```
  .... for brevity
  tasks:
    - name: install httpd
      yum: pkg=httpd state=latest
    - name: verify
      shell: systemctl status httpd
      register: result
      until: result.stdout.find("active (running)")!= -1
      retries: 5
      delay: 5
    - debug: var=result
```
- Notify
- Vault
encrypt yaml file.
```
ansible-vault create file.yml
```
enter password.

edit file:
```
ansible-vault edit file.yml
```

change password:
```
ansible-vault rekey file.yml
```

uncrypt
```
ansible-vault decrypt file.yml
```

- Prompt - Interactive Playbook
```
...for brevity
  vars_prompt:
    - name: prop_message
      prompt: Prop Message
      default: httpd #If no input
      private: no  #user input will not be echoed
      ... for brevity
```

- Basic Include Statements
subtask.yml:
```
- name: ...
  task: ...
- name: ...
  task: ...
```

main.yml:
```
...for brevity
 tasks:
    - include: subtask.yml
    - name: install httpd
      yum: pkg=httpd state=latest
...for brevity
```

- Tags
run specific task, or skip specific task

test.yml
```
...for brevity
 tasks:
    - name: install httpd
      yum: pkg=httpd state=latest
      tags:
        - a
...for brevity
```
```
ansible-playbook test.yml --tags "tagname"
```

```
ansible-playbook test.yml --skip-tags "tagname"
```

note [There is a tag called 'always' that always run.](http://docs.ansible.com/ansible/playbooks_tags.html)

- Basic Error Handling
```
...for brevity
 tasks:
    - name: install httpd
      yum: pkg=httpd state=wrongcommand
      ignore_errors: yes #default no
...for brevity
```

- Includes - Breaking Your Playbook Into Discrete Plays

- Starting At Task or Stepping Through All Tasks
```
ansible-playbook test.yml --start-at-task='taskname'
```

step each task and confirm.
```
ansible-playbook test.yml --step
```


- Passing Variables Into Playbooks at the Command Line
```
...for brevity
 tasks:
    - name: install {{var}}
      yum: pkg=httpd state={{status}}
      ignore_errors: yes #default no
...for brevity
```

run command:
```
ansible-playbook test.yml --extra-vars "var=httpd status=latest"
```

- Using Jinja2 Templates
test.j2
```
{{var1}}{{var2}}
```

```
---
- hosts: test
  connection: ssh
  user: test
  sudo: yes
  gather_facts: yes
  vars:
    var1: a
    var2: b
  tasks:
    - name:Install
      template: src=test.j2 dest=/home/test/test.conf owner=test group=test mode=750
```
file will be convert to ```a b``` , from server machine test.j2 to client machine test.conf

- LocalAction
run command on local machine
```
...for brevity
 tasks:
    - name: before install check connection
      local_action: command ping -c 5 clientmachine
...for brevity
```

- DelegateTo
```
...for brevity
 tasks:
    - name: install 
      yum: ping -c 1 secondmachine > /home/x.txt  #this happens on one client machine,and ping another client machine
      delegate_to: 127.0.0.1  # redirect to our server machine
...for brevity
```
==

- The 'Setup' Module
- The 'File' Module
```
ansible test -m file -a 'path=/etc'
```
```
ansible test -s -m file -a 'path=/tmp/etc state=directory mode=0700 owner=root'
```

```
ansible test -s -m command -a 'rm -rf /tmp/etc removes=/tmp/etc'
```
- The 'Pause' Module
stop for confirmation. enter to continue or ctrl c to exit
```
...for brevity
 tasks:
    - name: install 
      yum: pkg=httpd state=latest
    - name: pausing
      pause:
        prompt: "message...."
...for brevity
```
- The 'WaitFor' Module
```
...for brevity
 tasks:
    - name: install 
      yum: pkg=httpd state=latest
    - name: wait 8080 port to open
      wait_for:
        port: 8080
        state: started
...for brevity
```
in client machine, run
```
sudo systemctl start tomcat
```
then playbook will continue to run.

- The 'Yum' Module  
Note: name is alias of pkg
```
...for brevity
 tasks:
    - name: install 
      yum: name=httpd state=latest
...for brevity
```
- The 'Apt' Module
```
...for brevity
 tasks:
    - name: equi to apt-get update
      apt: update_cache=yes
    - name: equi to apt-get upgrade
      apt: upgrade=dist
...for brevity
```
- The 'Service' Module
```
...for brevity
 tasks:
    - name: install 
      yum: name=httpd state=latest
    - name: install service
      service: name=httpd state=enabled
...for brevity
```
- The 'Copy' Module
from local to remote. 
```
...for brevity
 tasks:
    - name: copy
      copy: src=/path/1.txt dest=/path2/1,txt owner=test group=test mode=0644 backup=yes # incase of file already exists
...for brevity
```
- The 'Command' Module
```
- name:
  command: /home/test/testing/test.sh
  args:
    chdir: /home/test/testing
```
- The 'Cron' Module
```
- name
  cron: name="list files" minute="0" hour="1" job="ls -al > /home/test/result.log"
```

remove this cronjob:
```
- name
  cron: name="list files" state=absent
```
- The 'Debug' Module
output useful json format message
```
    - name: install 
      yum: name=httpd state=latest
      debug: msg="message"
```
- The 'Fetch' Module
from remote to local
```
- name: Copy remote hosts file to control server
  fetch:src=/etc/hosts dest=/home/test
```

using var from gather_facts variables:
```
- name: Copy remote hosts file to control server
  fetch:src=/etc/hosts dest=/home/{{ansible_hostname}} flat=yes
```

- The 'User' Module
Add a user.
```
- name:
  user: name=newuser comment='comment' group='wheel'
  user: name=newuser state=absent remove=yes //remove a user
```
- The 'AT' Module
run command in future.
```
- name:...
  at: command='ls -al /var > /home/log" count=1 units="minutes"  # execute in 1 min
```
another param:  unique=true
```
at: command='ls -al /var > /home/log" state=absent #remove command
```



-  The 'DNF' Module
a package manager.
```
- name:
  dnf: name="*" state=latest
  dnf: name="@Development tools" state=latest
- name: install the latest version of Apache from the testing repo
  dnf: name=httpd enablerepo=testing state=present
- name: upgrade all packages
  dnf: name=* state=latest
```
- The 'Apache2_Module' Module
install or disable modules of apache2
```
apache2_module: state=absent name=alias
```
- The 'SetFact' Module
```
- name:
  set_fact:
    singlefact: SOME
-debug: msg-{{playbook_version}}  # interval var
-debug: msg-{{singlefact}}   #setfact
```
- The 'Stat' Module
```
- name:
  stat: path=/xx
    register: p
- debug: msg='msg'
    when: p.stat.isdir is defined and p.stat.isdir
```
- The 'Script' Module
```
 - name
   script: /path/to/xx.sh >> up.log
```
- The 'Shell' Module
```
 - name
   shell: /usr/bin/uptime >>uptime.log
   args:
     chdir: logs/
     creates: uptime.log     #create uptime.log in logs/, create uptime.log only if not exists
```

- Ansible 2.0 - Roles: User Privilege Escalation Changes
```
 - hosts:
   remote_user:
   become:
   become_method:
   connection:
   pre_tasks:
   - name:
     raw:
   roles:
   - { role: role1, when : "ansible_os_family == 'RedHat'" }
   - { role: role2, when : "ansible_os_family == 'Debian'" }
   post_tasks:
   - name:
     raw:
```
- Ansible 2.0 - Roles: The 'Find' Module
```
find: paths='/var/log" age="1d" recurse="yes" size="100k" patterns="*.log"
```
- User Privilege Escalation Changes
 (old syntax: user:test sudo:yes)
```
 - hosts: apache
   remote_user: test
   become: yes
   become_method: sudo
   connection: ssh
   gather_facts: no
   tasks:
     name: 
     yum: name=telnet state=absent
```
 
- Modules: The 'Find' Module

```
 tasks:
 - name:
   find: paths="/etc" patterns="*.txt,*.log" recurse=yes (no by default)
   register: result
 - debug: var=result
 
