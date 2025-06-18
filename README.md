# Guidelines

Author: Pasquale Domenico Colaianni, pasdom@biosustain.dtu.dk

This document outlines guidelines for employees and collaborators at CFB, with the goal of educating readers and minimizing the likelihood of issues arising with our computational platforms.

I will keep this document up-to-date with the needs of the center.

You are welcome to contact me for corrections and improvements. I understand that what has worked for me might not necessarily work for all: in that case, I encourage engaging in a discussion.

## Table of Contents

- [Principle of least privilege](#principle-of-least-privilege)
- [Azure Role Based Access Control (RBAC)](#azure-role-based-access-control-rbac)
  - [On privileged roles and resource groups](#on-privileged-roles-and-resource-groups)
- [Tags in Azure](#tags-in-azure)
- [Virtual machines (VMs) in Azure](#virtual-machines-vms-in-azure)
  - [Billing](#billing)
  - [Firewall](#firewall)
    - [Naming conventions for firewall rules](#naming-conventions-for-firewall-rules)
  - [Auto-shutdown](#auto-shutdown)
- [Secrets](#secrets)
- [sudo](#sudo)
- [Virtual machines with GPUs](#virtual-machines-with-gpus)
- [Security updates](#security-updates)
  - [Systems running on demand](#systems-running-on-demand)
- [Operating systems releases](#operating-systems-releases)
- [System administration](#system-administration)

## Principle of least privilege

> The principle of least privilege (PoLP) requires that in a particular abstraction layer of a computing environment, every module (such as a process, a user, or a program, depending on the subject) must be able to access only the information and resources that are necessary for its legitimate purpose.

Source: https://en.wikipedia.org/wiki/Principle_of_least_privilege

Many decisions in the following guidelines are based on this principle.

## Azure Role Based Access Control (RBAC)

> Azure role-based access control (Azure RBAC) is a system that provides fine-grained access management of Azure resources. Using Azure RBAC, you can segregate duties within your team and grant only the amount of access to users that they need to perform their jobs. Access management for cloud resources is a critical function for any organization that is using the cloud. Azure role-based access control (Azure RBAC) helps you manage who has access to Azure resources, what they can do with those resources, and what areas they have access to.

Source: https://learn.microsoft.com/en-us/azure/role-based-access-control/

Examples of built-in roles:

- Owner
- Role Based Access Control Administrator
- Contributor
- Network Contributor
- Reader
- Storage Blob Data Reader

A description of each role: https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles

Examples of custom roles (created by us):

- _Contributor without deletes_: same as _Contributor_ but without the ability to delete resources.
- _Virtual Machine Start-Restart-Stop_: it allows a user to start, restart and stop a virtual machine.

When assigning a role at the resource group scope, consider using _Time-bound_ under the _Assignment type_ tab.

> [!IMPORTANT]
> If you find that one or more destructive operations are possible via role _Contributor without deletes_, please inform an administrator.

### On privileged roles and resource groups

Privileged roles such as _Contributor_ and _Owner_ enable for all kinds of operations. Including the **modification** and the **deletion** of existing resources.

Assigning these roles poses a risk to the integrity of the platform.

Role _Contributor without deletes_ mitigates the risks, but does not protect against **modifications**.

Resource groups (RGs) should be project-wide rather than team-wide, following a sensible naming convention that clearly indicates which team owns a given project. Tags can provide additional clarity.

For example, for a given team _Foo_ and projects _Bar 1_ and _Bar 2_, one could create the following project-wide resource groups:

- rg-foo-bar-1
- rg-foo-bar-2

Role _Role Based Access Control Administrator_ provides with the ability to add role assignments.

The general recommendation is that a resource group should include only the people involved in most (if not all) of its resources.

> [!NOTE]
> Current team-wide RGs will not necessarily be fully migrated into project-wide RGs. I understand that some resources might pertain to a team and not to a project. In that scenario a team-wide RG makes sense, but it must exercise extra care in respecting the PoLP.

> [!WARNING]
> To Azure administrators:
>
> - Do not assign _Contributor_. Instead, use _Contributor without deletes_.
> - When assigning _Role Based Access Control Administrator_, do not select _Allow user to assign all roles_ under the _Conditions_ tab.

## Tags in Azure

Tag resources in Azure with the following keys:

- `created-by`: an example value would be `pasdom@dtu.dk`
- `created-for`: if the resource is created for someone else
- `purpose`: a description of why the resource exists

https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources-portal

## Virtual machines (VMs) in Azure

### Billing

VMs are billed as long as they are not in status _Stopped (deallocated)_.

To stop and deallocate a VM, navigate to its _Overview_ page and click _Stop_. Give it 30 seconds, click _Refresh_ and verify that the field _Status_ reports _Stopped (deallocated)_.

> [!WARNING]
> Commands `sudo shutdown -h now` or `sudo systemctl poweroff` do **not** deallocate the VM: these only _Stop_ it.

### Firewall

Creating a VM via the Azure portal results in the creation of more than one resource: a virtual machine, a disk, a network security group, etc.

The resource of type _Network security group_ (NSG) is the firewall. Navigate to its _Settings_, _Inbound security rules_. The same settings are reachable from the VM page: _Networking_, _Network settings_.

Ensure that SSH is not allowed to the whole Internet. Meaning: if you see a rule for SSH, and the _Source_ field is too permissive (i.e. `*`), this needs to change.

Click on the rule, then _Source_, and select _My IP Address_. Click _Save_. Once the request is successful, it might still take some seconds before the rule propagates to the network.

> [!IMPORTANT]
> Modifying firewall rules requires role _Network Contributor_ on the NSG resource.

> [!NOTE]
> Firewalls are not limited to virtual machines: they also apply to resources like storage accounts, container registries, databases, etc. These resources must be securely configured as well.

> [!NOTE]
> AIT has also published documentation addressing this topic: https://frontenddtudocumentation.z6.web.core.windows.net/wiki/restrict-network-ports.html

#### Naming conventions for firewall rules

Name rules according to their purpose. For example:

- _SSH-pasdom_ allows user _pasdom_ to connect via SSH
  - _Source_: `IP Addresses`
  - _Source IP addresses/CIDR ranges_: the user's IP address
  - _Service_: `SSH`

- _WEB-pasdom_ allows user _pasdom_ to connect to a web server
  - _Source_: `IP Addresses`
  - _Source IP addresses/CIDR ranges_: the user's IP address
  - _Service_: `Custom`
  - _Destination port ranges_: `80,443`
  - _Protocol_: `TCP`

- _WEB-DTU_ allows users at DTU's offices to connect to a web server
  - _Source_: `IP Addresses`
  - _Source IP addresses/CIDR ranges_: `192.38.0.0/16`
  - _Service_: `Custom`
  - _Destination port ranges_: `80,443`
  - _Protocol_: `TCP`

In general, expose only the ports that need to be exposed, and allow access only to the required IP addresses.

### Data storage

Virtual machines are not suitable for long-term data storage.

TODO: guidelines for data storage.

An email from our colleague August Lauridsen, IT Supporter, 2025-05-01:

> If you are using any of the Office 365 cloud apps to collaborate within your group, please make sure that any important data is also saved on the O drive (Department Drive-NNFCB).
>
> This is especially relevant for content stored in OneDrive, OneNote, or shared in Teams, as these can often be tied to an individual's personal DTU account. When someone leaves DTU, their account is closed, along with access to any data they created or shared. Shared OneDrive folders and files shared in Teams (if linked to a personal OneDrive account) will also be deleted when the DTU account is closed, even if others have access to them.
>
> This can affect, for example, tabs added in a Teams channel that appear to be part of the group's shared space but are actually linked to a personal account.
>
> If you're unsure whether your shared folders, notes, or Teams files are dependent on your personal account and might break when you leave, feel free to stop by our office or send us an e-mail at itsupport@biosustain.dtu.dk
>
> We're happy to take a look with you.

### Auto-shutdown

In the _Overview_ page (and within the submenu _Operations_) you will find _Auto-shutdown_. Enable it whenever possible.

If your computation takes longer than a day, remember to turn it off for the time needed. When the work is done, I encourage re-enabling Auto-shutdown to mitigate unnecessary expenses.

## Secrets

Examples of secrets include: passwords, API keys, tokens, etc.

Store secrets in a secure password manager.

Do not reuse the same secret across different accounts or services.

Do not store or hard-code secrets in codebases or any plaintext documents.

Environment variables files are a sensible exception to the rule: these might contain secrets.

> [!CAUTION]
> Environment variables files must be accessible only by the owner (e.g. `chmod 600 .env`) and must be ignored by both _Docker_ and _Git_. See paragraph [Ignoring files in Docker and Git](#ignoring-files-in-docker-and-git).

Examples of secure password managers: [Bitwarden](https://bitwarden.com/help/is-bitwarden-audited/), [1Password](https://support.1password.com/security-assessments/), [Keepass](https://keepass.info/ratings.html).

Guidelines for strong passwords:

- https://www.inside.dtu.dk/en/medarbejder/it-og-telefoni/informationssikkerhed/sikker-adfaerd/sikker-adgangskode
- https://en.wikipedia.org/wiki/Password_strength#Guidelines_for_strong_passwords

### Ignoring files in Docker and Git

Both Docker and Git provide a way to ignore (exclude) files. This eliminates the risk of accidentally copying secrets into Docker images or tracking them in git repositories.

Documentation:

- https://docs.docker.com/build/concepts/context/#dockerignore-files
- https://git-scm.com/docs/gitignore

A collection of templates for _.gitignore_: https://github.com/github/gitignore

## sudo

`sudo` allows a permitted user to execute a command as the superuser or another user.

A user can be provided elevated privileges for selected commands.

Imagine a user who needs to list Docker containers and access logs. The system administrator can allow the user by creating a file within directory _/etc/sudoers.d_:

```
$ sudo cat /etc/sudoers.d/pasdom
pasdom ALL=(ALL) NOPASSWD: /usr/bin/docker logs *
pasdom ALL=(ALL) NOPASSWD: /usr/bin/docker ps *
```

That allows user _pasdom_ to execute `sudo docker logs` and `sudo docker ps`, with arguments included.

> [!CAUTION]
> Adding users to the `docker` group effectively grants them full root access.

## Virtual machines with GPUs

> [!WARNING]
> The Nvidia repositories might contain drivers for Debian that are not digitally signed. In that case, _secure boot_ must be disabled. The following will cause a VM restart: visit the Azure portal, navigate to the VM, _Overview_, _Security_ and remove the tick from _Enable secure boot_. Apply.

```
sudo apt update
sudo apt upgrade
sudo apt install gcc
sudo apt purge cuda-keyring
sudo reboot
# for Ubuntu 24.04 (Noble Numbat):
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
# for Debian 12 (Bookworm):
wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt install cuda-toolkit-12-6
sudo apt install cuda-drivers-565
```

Now follow the post-installation actions: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions

I create a file at `/etc/profile.d/cuda.sh` and insert the environment variables there.

Reboot the system and it should now be ready to use.

References:

- Ubuntu: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#network-repo-installation-for-ubuntu
- Debian: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#network-repo-installation-for-debian

## Security updates

It is important to automate security updates.

Systems that run 24/7 might follow slightly different guidelines, compared to systems that run "on demand".

In Linux, a package named `unattended-upgrades` helps with checking and installing updates. But its default configuration might be not optimal.

I recommend checking and installing _security_ updates automatically, and enabling automatic reboots.

I create a file in _/etc/apt/apt.conf.d_.

```
$ cat /etc/apt/apt.conf.d/51unattended-upgrades
#clear Unattended-Upgrade::Allowed-Origins;
#clear Unattended-Upgrade::Origins-Pattern;
Unattended-Upgrade::Origins-Pattern {
        "origin=${distro_id},codename=${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
```

And:

```
$ cat /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Learn more: https://documentation.ubuntu.com/server/how-to/software/automatic-updates/

### Systems running on demand

Some systems are used "on demand": these are started, used for several hours or days, and then shut down until the next use.

In this scenario, having automatic updates (checks, installs, reboots) is undesirable. A manual approach might serve you better.

Disable automatic updates:

```
$ cat /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

And install them manually, before your computation:

```
sudo unattended-upgrades --verbose
sudo reboot
```

Rebooting is necessary only if security updates are found and installed.

## Operating systems releases

Do not use operating system releases that are out of support.

- https://ubuntu.com/about/release-cycle
- https://www.debian.org/releases

When the release you use goes out of support, it is time to upgrade.

- https://documentation.ubuntu.com/server/how-to/software/upgrade-your-release/index.html
- https://wiki.debian.org/DebianUpgrade

## Connecting via SSH

By convention, systems expose SSH on port 22.

If I am unable to connect to a system, I check that:

- the system is running
- the SSH port is open

On Linux, I test open ports with `nc`:

```
$ nc -v -w 3 -z www.google.com 80
www.google.com [142.250.179.164] 80 (http) open
$ nc -v -w 3 -z www.google.com 443
www.google.com [142.250.179.164] 443 (https) open
$ nc -v -w 3 -z www.google.com 22
www.google.com [142.250.179.164] 22 (ssh) : Connection timed out
```

Instead of `www.google.com`, insert the desired destination IP address.

## System administration

### Setting the timezone

```
sudo timedatectl set-timezone 'Europe/Copenhagen'
```

### Hardened SSH access

```
$ cat /etc/ssh/sshd_config.d/60-hardened-access.conf
AddressFamily inet
PasswordAuthentication no
PermitRootLogin no
X11Forwarding no
```

For public-facing systems, consider using solutions like _fail2ban_, which temporarily blocks IP addresses based on criteria such as the number of failed connection attempts in a given span of time.

### More Bash history, enable timestamps

```
$ vim /etc/profile.d/pasdom.sh
export HISTSIZE=10000
export HISTFILESIZE=1000000000
export HISTTIMEFORMAT='%F %T '
```
