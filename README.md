#### lyusingansible
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
  cron: name="list files" minute="0" hour="1" job="ls =al > /home/test/result.log"
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
