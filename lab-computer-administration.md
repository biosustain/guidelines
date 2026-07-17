# Laboratory computer administration and user responsibilities

This document serves as a practical checklist of key considerations and best practices for laboratory computers.

## Acknowledgments

Author: Pasquale Domenico Colaianni (pasdom@dtu.dk)

Special thanks to Kai Blin (kblin@dtu.dk) and Simon Shaw (sisha@dtu.dk) for sharing their extensive knowledge in maintaining Linux systems.

Thanks also to Kasper Stener Petersen (kastpe@dtu.dk) for his contributions to standardizing IT processes and facilitating the usage of Linux in our laboratories.

## Table of Contents

- [Operating system](#operating-system)
- [Authentication](#authentication)
- [Passwords](#passwords)
- [Storage](#storage)
- [Firewall](#firewall)
- [Antivirus](#antivirus)
- [Misc](#misc)

## Operating system

- Stick to Debian and Ubuntu
  - Debian: install the "stable" release https://www.debian.org/releases/
  - Ubuntu: install the latest "LTS" release https://ubuntu.com/about/release-cycle
  - Some software might require a specific distro
  - Some systems come with a pre-installed operating system and sometimes dubious security setup
    - Discuss with vendor and local admin
- Use a deployment automation system (e.g. Ansible) for easy re-installation and system-wide change tracking
- If not using automation, manually document all system-wide changes
- Unattended-upgrades: only for security updates
  - Automatic restarts are supported, but can interrupt workflows
  - Kernel updates require restarts
  - Consider a maintenance window where an admin manually installs kernel updates and restarts the system

## Authentication

- sudo / root privileges only to admins
  - Some software might require root: discuss it with admins
- Do not share credentials
  - Document exceptions and reasons
- Prefer using unprivileged user accounts (no root, no sudo)
- Providing membership to groups `root`, `sudo` or `docker` is a security risk
  - Consider using _podman_ for rootless containers
- Automatic lock of the screen upon inactivity
- Central DTU AD/Kerberos auth systems allow for using the DTU credentials
    - https://documentation.ubuntu.com/server/how-to/sssd/with-active-directory/
  - Scattered documentation for DTU: not easy to set up and maintain
  - You might be unable to add AD users to the `sudo` group
  - Comes with integration for O-drive
  - It requires password-based login enabled in sshd

## Passwords

- Use secure password managers, and consider the _bus factor_
  - https://en.wikipedia.org/wiki/Bus_factor
- Prefer long passphrases over difficult-to-type passwords
  - Be aware that different keyboard layouts might cause password input to fail
- AD/Kerberos mitigates the need for password management

## Storage

- Separate partitions for `/`, `/home`, `/tmp`

## Firewall

- Open only required ports, from selected source IPs
  - Use DTU VPN for access from remote locations
- DTU Network topology https://net.ait.dtu.dk/ipam/
  - https://net.ait.dtu.dk/ipam/index.php?network=10.75.0.0/22
  - https://net.ait.dtu.dk/ipam/index.php?network=10.75.128.0/23
  - https://net.ait.dtu.dk/ipam/index.php?network=10.198.48.0/21
  - https://net.ait.dtu.dk/ipam/index.php?network=10.198.56.0/22
  - https://net.ait.dtu.dk/ipam/index.php?network=10.198.60.0/22

## Antivirus

- Might cause more issues than it solves
  - `clamav` is available for manual scans

## Misc

- Consider setting up user disk quotas
- Monitor metrics (Loki, Grafana)
- Discuss backup strategy with users
