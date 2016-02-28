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
- hosts: all
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


make new dir
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
- The 'Fetch' Module
```
- name: Copy remote hosts file to control server
  fetch:src=/etc/hosts dest=/home/test
  
```

- The 'User' Module
Add a user.
```
- name:
  user: name=newuser comment='comment' group='wheel'
  user: name=newuser state=absent remove=yes //remove a user
```
- The 'AT' Module
```

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
 
```
- The 'Apt' Module
```
- name: Equivalent to apt-get update
  apt: update_cache=yes
- name: Equivalent to apt-get upgrade
  apt: upgrade=dist
```
- The 'Copy' Module
```
action: copy src=/hostmachine/txt dest=clientmachine/txt owner=test group=txt mode=0777
```
-  The 'DNF' Module
```
name:
dnf: name="*" state=latest
//or
dnf: name="@Development tools" state=latest
```
- The 'Apache2_Module' Module
```
apache2_module: state=absent name=alias
```
- The 'SetFact' Module
```
- name:
  set_fact:
    singlefact: SOME
-debug: msg-{{playbook_version}}
-debug: msg-{{singlefact}}
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
