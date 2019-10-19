# Ansible Tutorial & Playbook Installing Docker

## Installing ansible

Install according your distro

```
# Fedora / RHEL 8 / CentOS 8
dnf install ansible

# Debian / Ubuntu
apt install ansible
```

Check installed ansible version:

```
ansible --version
```

> Ansible successfully installed

## Prerequisites

Create host file:

```
cat >> hosts << EOF

[group1]
host1 ansible_host=<host1 ip address> ansible_user=<host1 username>
host2 ansible_host=<host2 ip address> ansible_user=<host2 username>
EOF
```

run command (print date on each host) on all host:

```
ansible group1 -i hosts -m command -a "date"
```
> Make sure all host are accessible using passwordless ssh 

## Ansible Playbook

Generate ansible playbook yaml file for installing `docker-ce` and `docker-compose` on ubuntu:

```
cat >> ubuntu-lts-docker.yaml << EOF
---
- name: Install docker-ce on Xenial and Bionic
  hosts: all
  become: true
  tasks:
  - name: Install aptitude using apt
    apt: name=aptitude state=latest update_cache=yes force_apt_get=yes

  - name: Install prerequisites
    apt:
      name: ['apt-transport-https', 'ca-certificates', 'curl', 'gnupg2', 'software-properties-common', 'python3-pip', 'python3-setuptools', 'virtualenv']
      update_cache: yes

  - name: Add Docker GPG key
    apt_key: url=https://download.docker.com/linux/ubuntu/gpg

  - name: Add Docker APT repository
    apt_repository:
      repo: deb [arch=amd64] https://download.docker.com/linux/{{ansible_distribution|lower}} {{ansible_distribution_release}} stable

  - name: Install Docker
    apt:
      name: docker-ce
      update_cache: yes
      
  - name: Upgrade pip
    pip:
      name: pip
      extra_args: --upgrade

  - name: Install docker-compose
    pip:
      name: docker-compose
                            
EOF
```

Run ansible playbook:

```
ansible-playbook ubuntu-lts-docker.yaml -i hosts
```
