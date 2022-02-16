# Getting Started with Ansible

## Key Concepts
![Ansible Key Concepts](Ansible.png?raw=true "Ansible.png")

## Prerequisites
Before you want to try this on your local, here are requirements
* A public key in .ssh/id_rsa.pub in your home directory
* Directly download [Virtualbox](https://www.virtualbox.org/) and install, or use homebrew
    ```
    brew install --cask virtualbox
    ```
* Vagrant
    ```
    brew install --cask vagrant
    ```
* Ansible
    ```
    brew install ansible
    ```

## Provision VMs
```
cd getting-started-with-ansible
vagrant up
```

Let's ssh into two guest VMs
```
ssh ci@172.16.16.11
```
```
ssh ci@172.16.16.12
```

## Ad-hoc Commands

Ping two guest VMs
```
ansible all -i inventory.yaml -m ping -u ci
```

Check current directory after login
```
ansible all -i inventory.yaml -m shell -a "pwd" -u ci
```

## Playbook
See all available  [Ansible modules](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html) 

Execute playbook to install Docker on guest VMs
```
ansible-playbook -i inventory.yaml playbook.yml
```
