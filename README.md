# Ansible Playbook for RTF Deployment

## Ansible Installation

[Ansible](https://docs.ansible.com/ansible/latest/index.html)

Ansible only need be install at control host, like on bastion or your computer that connect to all targeted nodes.

Install on Mac OS:

`
brew install ansible
`


## RTF Deployment Playbook

### Invetory file

Ansible playbook is using inventory file called **hosts**

**hosts** file is for inventory all node of PCE deployment, need to add node variable below:

1. internal_hostname -> internal vpc/network hostname
2. docker_device -> docker hdd device
3. etcd_device -> etcd hdd device (Only for controllner node)
4. role -> RTF Node Role (installer/controller/worker)

### Global Variable

Ansible playbook is using global variable at **group_vars/all.yaml**

All related RTF deployment setting can be set in that file like download installer etc

### Installer File

Put installer file at **installers** folder, change rtf_installer global variable to installer file and put rtf_download=no at global variable file if you don't want rtf installer node to download the installer 


## Running

`
ansible-playbook -i hosts rtf-install.yaml
`