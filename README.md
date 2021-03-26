# Ansible Playbook for RTF Appliance Installation

`TL;DR`
- Playbook for automating RTF installation on VMs provisioned as per prerequisites
- Works for the RTF appliance model (NOT for BYO k8s model)
- Consider it the `manual` flavour of install automated via Ansible in an orchestrated fashion (after all it's a multi-node k8s cluster)

## Installing Ansible

Refer to [Ansible Docs](https://docs.ansible.com/ansible/latest/index.html), choose the best option to install ansible on the control node(s).

## Running the RTF Appliance Playbook

### Build the Inventory file

Ansible works against multiple managed nodes (hosts), in RTF appliance context, the controller + worker node VMs, in the form of a list or group of lists known as `inventory`.

`TL;DR`: the 1st controller assumes the `installer` role (it is the only member of the `installer` group), all other controller and worker nodes are joiners (grouped in the `[nodes]` group).

The **hosts** file includes all RTF appliance cluster nodes, including controllers and workers.

Adjust the hosts file to reflect the target environment:

1. `internal_ip` -> The internal IP of a VM (it may have Public IPs associated with it, e.g. EC2 instances in a VPC), should be static
2. `docker_device` -> block device used by `Docker CE`
3. `etcd_device` -> block device used by `etcd` (ONLY for controller nodes)
4. `role` -> RTF Node Role (`installer`/`controller`/`worker`)

> NOTE: `installer` is the 1st controller node from which everything is bootstrapped.

### Global Variable

Ansible playbook is using global variable at `group_vars/all.yaml`.

All RTF Install related variables can be set in the `group_vars/all.yaml`.

### Installer File

Place the installer file at `installers` folder, change `rtf_installer` global variable to installer file and put `rtf_download=no` at global variable file if you don't want the rtf installer node to download the installer over public Internet (should be avoided if outbound connectivity is slow, e.g. via a slow Proxy).

### Running the show (playbook)

```bash
ansible-playbook -i hosts rtf-install.yaml
```
> NOTE: increase verbose level by specifying `-v`, `-vvv` or even `-vvvv`, `RTFM` for more.

### Troubleshooting

To be added

Procedure
1. review log, understand what has failed
2. identify nodes that has failed to bootstrap (installer node) or join (all others)
3. group the failed nodes in inventory file (e.g. `[failed]`)
4. clean up against the failed nodes (ansible ad-hoc tasks), by leveraging the `--limit 'all:!failed'` option.
5. run the playbook against the failed (remaining) nodes

Repeat if necessary (unlucky).
