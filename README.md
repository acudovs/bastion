# Ansible Playbook for managing the Bastion virtual machine

This Ansible playbook is designed to control the Bastion virtual machine. It uses a set of tasks to create
or destroy a Bastion virtual machine, and expose or isolate a Virtualization Host behind the Bastion.

The main idea is to disconnect a network device (Ethernet, Wi-Fi, etc.) from the Virtualization Host,
effectively isolating it from the network. This device is then passed through to a minimalist Bastion
virtual machine, which is used as a firewall, default gateway, and DNS server. In addition, VPN client
software can be installed on the Bastion host to allow access to corporate resources. This setup enhances
security by creating an isolated environment for network operations controlled by the Bastion host.

The Playbook assumes that libvirt is configured and running on the Virtualization Host, the current user
is a member of the libvirt group, and can execute sudo without password. The created virtual machine runs
Oracle Linux 9 with the minimum number of packages installed to support the GlobalProtect VPN client (not
included in this repository).

Note that not all devices can be passed to virtual machines, and others may have bugs if they are neither
supported nor tested in passthrough mode by the vendor. So good luck.

This screenshot shows the Bastion virtual machine running on a laptop with a Wi-Fi network device connected
to the virtual machine and the GlobalProtect UI running in GNOME Desktop Manager.

![](https://github.com/acudovs/bastion/blob/master/images/GlobalProtect.png?raw=true)

## Playbook Structure

The playbook is structured with the following main components:

### bastion.yml

The main playbook file that orchestrates the execution of different tasks.

#### create.yml

This role creates the Bastion virtual machine. It includes tasks such as downloading disk image, creating
cloud-init ISO image, provisioning the Bastion virtual machine, and starting it.

#### destroy.yml

This role destroys the Bastion virtual machine. It includes tasks such as shutting down and removing the
virtual machine along with its associated resources.

#### isolate.yml

This role isolates the Virtualization Host behind the Bastion virtual machine. It includes tasks such as
configuring network device passthrough and setting up network configuration.

#### expose.yml

This role reverses the isolation process. It includes tasks such as restoring network configurations and
disabling network device passthrough.

#### facts.yml

This role gathers simple facts about the virtual machine, including its existence and network information.

## Running the Playbook

To execute the playbook, use one of the following commands:

```bash
ansible-playbook bastion.yml -e play=create
ansible-playbook bastion.yml -e play=create --tags=console
ansible-playbook bastion.yml -e play=isolate
ansible-playbook bastion.yml -e play=expose
ansible-playbook bastion.yml -e play=destroy
```

## Customizing Configuration

Modify variables in the `roles/bastion/defaults/main.yml` file to customize the behavior of the playbook,
such as virtual machine specifications, disk images, network configuration, and more.

Modify the cloud-init templates in the `roles/bastion/templates` directory to customize the virtual machine
provisioning process.

## Known problems

Depending on the network device, when the device is detached from the virtual machine, the device stops
working on the host until next reboot.
