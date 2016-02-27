#### lyusingansible
- configure
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

- config
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
