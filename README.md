# Ansible Playbook for RTF Appliance Installation

`TL;DR`
- Playbook for automating RTF installation on VMs provisioned as per prerequisites
- Works for the RTF appliance model (NOT for BYO k8s model)
- Consider it the `manual` flavour of install automated via Ansible in an orchestrated fashion (after all it's a multi-node k8s cluster)


## Installing Ansible

Refer to [Ansible Docs](https://docs.ansible.com/ansible/latest/index.html), choose the best option to install ansible on the control node(s).

## Running the Playbook

### Inventory file

Ansible playbook is using inventory file called **hosts**

**hosts** file is for inventory all node of  deployment, need to add node variable below:

1. `internal_hostname` -> internal vpc/network hostname
2. `docker_device` -> `docker` block device
3. `etcd_device` -> `etcd` block device (ONLY required for controller nodes)
4. role -> RTF Node Role (installer/controller/worker)

> NOTE: `installer` is the 1st controller node from which everything is bootstrapped.

### Global Variable

Ansible playbook is using global variable at **group_vars/all.yaml**

All related RTF deployment setting can be set in that file like download installer etc

### Installer File

Put installer file at **installers** folder, change rtf_installer global variable to installer file and put rtf_download=no at global variable file if you don't want rtf installer node to download the installer 


## Running

`
ansible-playbook -i hosts rtf-install.yaml
`
