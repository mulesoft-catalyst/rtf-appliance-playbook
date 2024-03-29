# Ansible Playbook for RTF Appliance Installation

`TL;DR`
- Playbook for automating RTF installation on VMs provisioned as per prerequisites
- Works for the RTF appliance model (NOT for BYO k8s model)
- Consider it the `manual` flavour of install automated via Ansible in an orchestrated fashion (after all it's a multi-node k8s cluster)

> Supported Linux distributions: RHEL {7,8}, Ubuntu LTS 18.04 (20.04 current and 22.04 next LTS to be added).

## Installing Ansible

Refer to [Ansible Docs](https://docs.ansible.com/ansible/latest/index.html), choose the best option to install ansible on the control node(s).

## Running the Playbook

### Create Inventory file

Ansible works against multiple managed nodes (hosts), in RTF appliance context, they are the controller and worker node VMs, in the form of a list or group of lists known as `inventory`.

> **IMPORTANT**: Please explicitly specify Linux user and group in the inventory file if remote target systems do NOT follow the [User Private Group](https://docs.fedoraproject.org/en-US/fedora/rawhide/system-administrators-guide/basic-system-configuration/Managing_Users_and_Groups/) Scheme (AKA `UPG` - user has a group of the same name, user is the sole member) which is widely adopted by most mainstream distributions. Otherwise `permission denied` errors are expected when performing file / directory related operations requiring escalated privileges.

**Recommendation**: Make remote target Linux systems `UPG` compliant if possible.
1. In `/etc/login.defs`, set `USERGROUPS_ENAB yes` (default value unless changed). It can be overridden by specifying `-N` or `--no-user-group` running `useradd` when creating a new user.
2. Manually create a private group for user, add user to group as its sole member. Example for user `noob` at scale: `ansible all -m shell -a "groupadd noob && usermod -aG noob noob" -b -i inventory_file`.


**`TL;DR`**: the 1st controller assumes the `installer` role (it is the only member of the `installer` group), all other controller and worker nodes are joiners (grouped in the `[nodes]` group).

The **`hosts`** file includes all RTF appliance cluster nodes, including controllers and workers.

Adjust the hosts file to reflect the target environment:

1. `internal_ip` -> The internal IP of a VM (it may have Public IPs associated with it, e.g. EC2 instances in a VPC), should be static
2. `docker_device` -> block device used by `Docker CE`
3. `etcd_device` -> block device used by `etcd` (ONLY for controller nodes)
4. `role` -> RTF Node Role (`installer`/`controller`/`worker`)

> NOTE: `installer` is the 1st controller node from which everything is bootstrapped.

### Set Playbook Variables

Ansible playbook is using global variable at `group_vars/all.yaml`.

All RTF Install related variables can be set in the `group_vars/all.yaml`.

> IMPORTANT: recently `init.sh` defaults default Linux NIC to `eth0` which may not work for systems adopting systemd (v197) Predictable NIC names, e.g. Fedora, `EL{7,8}`.

Make sure to set it correctly reflecting target systems in `group_vars/all.yaml`, e.g. `internal_interface: "ens192"` for VMware (as hypervisor), and VirtualBox emulating Intel Pro/1000 MT Desktop will be persisted as `enp0s3` by default. Of course, you can revert to traditional NIC names by passing `net.ifnames=0` as kernel command line parameter at boot via a bootloader (GRUB2).

### RTF Appliance Installer

> NOTE: It is a self-contained binary installer created using `gravity`.

Place the installer file at `installers` folder or symlink it, change `rtf_installer` global variable to installer file and put `rtf_download=no` at global variable file if you don't want the rtf installer node to download the installer over public Internet (should be avoided if outbound connectivity is slow, e.g. via a slow Proxy).

### Running the show (playbook)

```bash
ansible-playbook -i hosts rtf-install.yaml
```
> NOTE: increase verbose level by specifying `-v`, `-vvv` or even `-vvvv`, `RTFM` for more.


### Uninstall RTF Appliance

```bash
ansible-playbook -i hosts rtf-uninstall.yaml
```
> NOTE: if more flexibility is required during the uninstall process, use ad-hoc tasks instead.

```bash
# example
# uninstall gravity
ansible all -m shell -a "gravity system uninstall --confirm" -b -i hosts -l 'all'

# clean up RTF directory structure
# treat installer node differently to save time
ansible all -m shell -a "rm -rfv /opt/anypoint/runtimefabric" -b -i hosts -l 'all:!installer'
ansible all -m shell -a "sudo rm -rfv /opt/anypoint/runtimefabric/.{state,rtf,data}" -b -i hosts -l 'installer'

# umount files systems
ansible all -m shell -a "umount -l /var/lib/gravity/planet/etcd; umount -l /var/lib/gravity" -b -i hosts -l 'controllers'
ansible all -m shell -a "umount -l /var/lib/gravity" -b -i hosts --limit 'workers'
# remove fstab entries for etcd and docker block devices
ansible all -m shell -a "sed -i '/RTF/d' /etc/fstab" -b -i hosts -l 'all'

# reboot all nodes to get clean state
ansible all -m shell -a "systemctl reboot" -b -i hosts -l 'nodes'
```

### Troubleshooting

General troubleshooting steps:

1. review log, understand what has failed
2. identify nodes that has failed to bootstrap (installer) node or joining (all others) node(s)
3. group the failed nodes in inventory file (e.g. `[failed]`)
4. run cleanup against the failed nodes (ansible ad-hoc tasks), by leveraging the `--limit 'all:!failed'` option
5. run the playbook against the failed (remaining) nodes if installer node has completed registration; otherwise, rerun install playbook completely

Repeat if necessary (unlucky).
