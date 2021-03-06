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
- The 'SELinux' Module  
permissive, enforcing, disabled
```
tasks:
  - name: change
    selinux: state=disabled
```
- The 'SEBoolean' Module
```
- name: ..
  seboolean: name=httpd_anon_write state=yes
```

- The 'Raw' Module
- The 'Ping' Module
```
- name: ..
  ping:
```
- The 'Package' Module  
 Generic OS package manager  
```
- name: ..
  package: name=telnet state=latest
```
- The 'Unarchive' Module
```
- name: ..
  unarchive: src=test.tar.gz dest=/home/unarchive
```
- The 'HTPasswd' Module  
add a user
```
- name:
  htpasswd: path=/etc/apache2/.htpasswd name=test password=password owner=test group=test mode=0644
```
- The 'GetURL' Module  
Download stuff
```
- name:
  get_url: url=http://... dest=/ owner=test group=test mode=0644
```
- The 'Group' Module
add a group
```
- name:
  group: name=newgroup state=present/absent
```
- The 'Mail' Module
```
- name:
  mail:
    host='localhost'
    port=25
    to="a@b.com
    subject="sub"
    body='{{ansible_hostname}}'
```
- The 'Filesystem' Module
```
- name:
  filesystem: fstype=ext3 dev=/dev/xvdf1 opts="-cc"
```
- The 'Mount' Module
```
- name:
  mount: name=/mnt/data src=/dev/xvdf1 fstype=ext3 opts=rw state=present
```
- The 'Notify' Module
- The 'AptRepo' Module
```
- name:
  apt_repository: repo='deb http://dl.google.com/inuxdeb/stable main' state=present
```
- The 'AptKey' Module
```
- name:
  apt_repository: url=https://dl-ssl.google.com/linux_signing_key.pub state=present
```
- The 'ACL' Module
```
- name:get acl
  acl: name=/etc/test.acl.txt
```
- The 'Git' Module
```
- name:checking out git repo
  git: repo=ssh://rengokantai@ly.git dest=/home/repo 
```
- Creating a Jinja2 Template File  
- The 'Template' Module  
see line 666.
- The 'MySQL_DB' Module
```
- name: install python mysql
  yum: pkg=MySQL-python state=latest
  name:create db
  mysql_db: name=testdb state=present login_user=root login_password=password
```

dump:
```
  mysql_db: name=testdb state=dump target=/var/dump.sql login_user=root login_password=password
```
- The 'MySQL_User' Module
```
- name: create mysql user
  yum: name=user1 password=password1 priv=dbname.tablename:ALL state=present login_user=root login_password=password
```
- The 'Kernel_Blacklist' Module  
block some modules
```
- name: block module
  kernel_blacklist: name=dummy state=present
```
check in client machine:  
check modprobe.d  
```
cat blacklist-ansible.conf
```
remove module from linux kernel:
```
sudo modprobe -r dumm
```
==
- Roles - The Directory Structure  
from playbook/  
```
mkdir roles
cd roles
```

then we create some subcommands:
```
mkdir common
mkdir webserver
....
```

in any of subfolders, create some vars, for example
```
cd common
mkdir files
mkdir tasks
mkdir templates
mkdir handlers
mkdir meta
mkdir defaults
mkdir vars
```
- Role Based Tasks
create `main.yml` in each folder

create a master playbook: (first)
```
---
- hosts: apache
  user: test
  sudo: yes
  connection: ssh
  roles:
    - webserver
```
- Task Order - Pre and Post Tasks
```
---
- hosts: apache
  user: test
  sudo: yes
  connection: ssh
  pre_tasks:
  - name: name
    raw: date > x.log
  roles:
    - webserver
  post_tasks:
  - name: name
    raw: date > x.log
```  
- Roles - Conditional Execution  
see line 513-518
- Roles - Variable Substitution
see line 239
- Roles - Handlers
see line 297
- Roles - Using Notification
- Roles - Configuring Alternate Roles Paths  
open ansible.cfg,uncomment and change
```
#role_path=/etc/ansible/roles
```
- Roles - Conditional Include Statements  
Must memorize this syntax  
```
---
- hosts: apache
  user: test
  sudo: yes
  connection: ssh
  pre_tasks:
  - name: name
    raw: date > x.log
  roles:
    - {role: redhat, when: ansible_os_family=="RedHat"}
    - {role: debian, when: ansible_os_family=="Debiant"}
  post_tasks:
  - name: name
    raw: date > x.log
```  
- Roles - Waiting For Events
see line 722
- Roles - Executing a Task Until
see line 529
- Roles - Using Tags
```
ansible-playbook test.yml --tags "tagname" --limit "hosts"    #--limit override hosts
```
- Roles - Breaking a Playbook Into a Role
- Roles - Passing Variables from Command Line
see line 645
- Roles - Using Jinja2 Templates
- Roles - DelegateTo
see line 686
- Roles - LocalAction
see line 676
==

- Ansible Command Line - Installing Packages
```
ansible -u test -s -m yum -a "pkg=httpd state=latest"
```
- Ansible Command Line - Services and Hosts
```
ansible -u test -s -m service -a "name=httpd state=restarted"
```
- Ansible Command Line - Commands and Shells
```
ansible -u test -s -m command -a "ls -al"
```
- Ansible Command Line - Managing Users
```
ansible -u test -s -m user -a "name=testuser uid=1001 shell=/bin/bash"
```
- Ansible Command Line - Create and Manage Cron Jobs
```
ansible -u test -s -m cron -a "name='crontest' minute='0' hour='12' job='ls= al'"
```

```
ansible -u test -s -m cron -a "name='crontest' state='absent'"
```
- Ansible Command Line - Running Arbitrary Commands
```
ansible -u test -s -a "ls -al"
```
- Ansible Command Line - Output Tree
```
ansible -u test -s -m yum -a "pkg=httpd state=latest" -t dir  # -t: create a dir 
```
- Creating a Web Server Deployment - Outline
- Creating a Web Server Deployment - Playbook First Pass
```
---
- hosts: apache
  user: test
  sudo: yes
  connection: ssh
  gather_facts: yes
  vars: 
    apache_pkg: httpd
    dist: redhat
    apache_version: 2.4
    apache_dir: /var/www/test
    apache_fqdn: y.me
  tasks:
    - name: install
      yum: pkg=httpd state=latest
    - name: create dict
      file: path=/var/www/test state=directory mode=644
    - name: copy apache config file
      copy: src=files/httpd.conf.template dest=/etc/httpd/conf/httpd.conf owner=root group=root mode=644
    - name: create vhost.d dict on the remote host
      file: path=/etc/httpd/vhost.d state=directory mode=755
    - name: copy and untar source file 
      unarchive src=files/site.tar.gz dest=/var/www/sample
    - name: copy default vhost
      copy: src=files/default.conf.template dest=/etc/httpd/vhost.d/default.conf owner=root group=test mode=644
    - name: start web server
      service: name=httpd state=started
    - name: test web server
      shell: curl http://y.me
      register: result
    - name: display
      debug: var=result
```
- Creating a Web Server Deployment - Playbook Optimization
- Creating a Web Server Deployment - Breaking Into Role(s)
- Creating an NFS Server Deployment - Outline
- Creating an NFS Server Deployment - Playbook First Pass
```
---
- hosts: apache
  user: test
  sudo: yes
  connection: ssh
  gather_facts: yes
  vars: 
    dist: RedHat
    nfsutils_pkg: nfs-utils
    nfslibs_pkg: nfs-utils-lib
    nfsserver_service: nfs-server
    nfslock_service: nfs-lock
    nfsmap_service: nfs-idmap
    rpcbind_service:rcpbind
    export_path: /var/share
  tasks:
    - name: Install NFS server
      yum:pkg=nfs-utils state=latest
      del
    - name: Install NFS server 
      yum:pkg=nfs-utils-lib state=latest
    - name: copy export file to remote server
      copy: src=files/exports.template dest=/etc/exports owner=root group=root mode=644
    - name: start RPC bind service
      service: name=rpcbind state=started
    - name:start NFS service
      service: name=nfs-server state=started
    - name: start file lock service
      service: name=nfs-locl state=started
    - name: start NFS map service
      service: name=nfs-idmap state=started
    - name: start NFS client and utilities
      service: name=nfs-utils state=latest
      delegate_to 127.0.0.1
```
    
      

- Creating an NFS Server Deployment - Playbook Optimization
- Creating an NFS Server Deployment - Breaking Into Role(s)
- Creating a Database Server Deployment - Outline
- Creating a Database Server Deployment - Playbook First Pass
```
---
- hosts: apache
  user: test
  sudo: yes
  connection: ssh
  gather_facts: yes
  vars: 
    dbserver_pkg:mariadb-server
    dblieent_pkg: mariadb
    dbinstalldir: /var/lib
    dbinstance: dbtest
    dbdist: RedHat
    dbversion: 5.5
  tasks:
    - name: install
      yum: pkg=mariadb-server state=latest
    - name: install client
      yum: pkg=mariadb state=latest
    - start service
      service: name=mariadb state=started
    - pause: prompt="please run mysql_secure_installation"
    - name: restart
      service: name=mariadb state=retarted
    - name: copy remote database
      copy: src=files/mysql.sql dest=/var/lib/mysql.sql owner=root group=root mode=755
    - name: import sql file
      shell: mysql -u root -p password dbname < /var/lib/mysql.sql
      register: result
    - debug: var=result
    - name: add cronjob
      cron: name="dbbackup" minute="0" hour="0" job="mysqldump -u root -ppassword --databases dbname >mysql.sql"
    - name: run quick sql
      shell: mysql -u root -p password -e 'SHOW DATABASES;'
      register: mysqlresult
    - debug: var=mysqlresult
```     
      

- Creating a Database Server Deployment - Playbook Optimization
- Creating a Database Server Deployment - Breaking Into Role(s)

==

- Ansible 2.0 - Installation  
deprecated.
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
- Ansible 2.0 - Modules: The 'Package' Module
uniform package name.
```
tasks:
  - name:
    package: name=telnet state=latest
```

- Ansible 2.0 - Roles: The 'Find' Module
```
find: paths='/var/log" age="1d" recurse="yes" size="100k" patterns="*.log"  # greater than 100k
```
- Ansible 2.0 - Roles: The 'Package' Module
