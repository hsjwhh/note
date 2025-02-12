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
# YAML 文件起始标识（标准Ansible Playbook开头）
- name: Update APT on multiple servers  # Play 的名称（描述性说明）
  hosts: Debian                      # 指定目标主机组（必须在inventory中定义）
  become: yes                        # 启用权限提升（相当于sudo）
  become_user: root                  # 指定提升为root用户（默认是root可省略）
  
  tasks:                            # 任务列表开始
    - name: Update APT cache         # 第一个任务名称（描述性说明）
      apt:                           # 使用apt模块
        update_cache: yes            # 执行等同于`apt-get update`的操作
        cache_valid_time: 3600       # 缓存有效期（秒）：若1小时内已更新过则跳过
      register: update_result        # 将任务执行结果保存到变量update_result中

    - name: Upgrade all packages if updates are available  # 第二个任务名称
      apt:                           # 使用apt模块
        upgrade: dist                 # 执行发行版升级（等同于`apt-get dist-upgrade`）
      when: update_result.changed     # 条件判断：仅当缓存更新成功时才执行升级操作
```


```ini
[local]
debian12 ansible_host=192.168.1.6 ansible_user=user ansible_ssh_private_key_file=/home/user/ansible/.ssh/debian12_id_rsa
[Debian]
u1 ansible_host=202.5.17.100 ansible_user=root ansible_ssh_private_key_file=/home/user/ansible/.ssh/u1_id_rsa
u2 ansible_host=104.128.239.189 ansible_user=root ansible_ssh_private_key_file=/home/user/ansible/.ssh/u2_id_rsa
```