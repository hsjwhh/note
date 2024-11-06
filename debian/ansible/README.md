# Ansible
ansible-playbook -i hosts l_apt_update.yml --ask-become-pass
```yaml
---
- name: Update APT on multiple servers
  hosts: local
  become: yes
  become_user: root
  tasks:
    - name: Update APT cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      register: update_result

    - name: Upgrade all packages if updates are available
      apt:
        upgrade: dist
      when: update_result.changed
```

ansible-playbook -i hosts u_apt_update.yml
```yaml
---
- name: Update APT on multiple servers
  hosts: Debian
  become: yes
  become_user: root
  tasks:
    - name: Update APT cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      register: update_result

    - name: Upgrade all packages if updates are available
      apt:
        upgrade: dist
      when: update_result.changed
```


```ini
[local]
debian12 ansible_host=192.168.1.6 ansible_user=user ansible_ssh_private_key_file=/home/user/ansible/.ssh/debian12_id_rsa
[Debian]
u1 ansible_host=202.5.17.100 ansible_user=root ansible_ssh_private_key_file=/home/user/ansible/.ssh/u1_id_rsa
u2 ansible_host=104.128.239.189 ansible_user=root ansible_ssh_private_key_file=/home/user/ansible/.ssh/u2_id_rsa
```