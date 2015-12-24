#### lyusingansible
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
