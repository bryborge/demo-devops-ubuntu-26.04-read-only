# Demo: Ubuntu 26.04 Read-Only

This project demonstrates how to bootstrap, provision, and manage a computer running Ubuntu 26.04 set up as "read-only".

## Overview

### Motivation

The motivation for this project was to explore how to install and provision a student-friendly Linux distribution on a
machine running in a university laboratory environment.

### Tradeoffs

**Cons:**

- RAM is a constraint if the machine is not rebooted regularly (overlayroot writes to memory instead of disk).
- Adds friction to users who may need a new library or dependency installed on the machine.
  - Note: This is sort of a pro AND a con.
- Updates and provisioning require orchestrated reboots and temporary disables of read-only mode, slightly complicating remote management.
- Local logs, crash dumps, and forensic data are lost upon reboot unless forwarded to a remote logging server.
- Users must understand that their local saves will be destroyed.
- Some applications may behave poorly or experience slow startups if they heavily rely on persisting local caches.

**Pros:**

- Declarative, mostly automated processes for maintaining system state over time with many potential users.
- A simple reboot restores the machine to a pristine, known-good state, wiping out malware, accidental changes, and leftover student files.
- Greatly minimizes the maintenance burden; lab administrators rarely need to re-image machines manually.
- Provides a safe sandbox environment where users can freely experiment (even with `sudo`) without permanently breaking the host system.
- Encourages better habits, pushing users toward repeatable/scripted workflows and version control instead of relying on local state.

## Getting Started

### Requirements

- Python (see `.python-version` file for specific Python version)
- 2x USB storage drives (for autoinstall and initial system bootstrapping)

### 1. Bootstrapping the System

#### Autoinstall

1.  Burn the iso image of Ubuntu Desktop 26.04 to a USB drive.
2.  Find an additional USB drive and format it with FAT32 partition. Name it `CIDATA`.
3.  Copy the contents of `cloud-init/` onto the newly formatted USB drive from step 2.
4.  Plug both of the USB drives into the target machine that will run Ubuntu 26.04.
5.  Power on the machine and select `Try/Install Ubuntu` from the bootloader menu.
6.  The presence of `CIDATA` and the files on it should trigger the installation automatically.

Once the installation is complete, the computer will power off.

#### Provisioning

1.  Install the Python dependencies for Ansible:

    ```sh
    pip install -r requirements.txt
    ```

2.  Ensure the target machine's IP address or hostname in `ansible/inventory.ini` is correct.

3.  Run the bootstrap playbooks to prepare the machine (e.g., rotate credentials and set up the overlayroot):

    ```sh
    ansible-playbook -i ansible/inventory.ini ansible/playbooks/00_bootstrap/00_rotate-creds.yml -kK
    ansible-playbook -i ansible/inventory.ini ansible/playbooks/00_bootstrap/01_overlayroot.yml -kK
    ```

    Follow the prompts as required.

### Installing Packages

Run the package installation playbook to install the required baseline software (git, direnv, pyenv, vim):

```sh
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-packages.yml -kK
```

### Updating the System

When routine updates are required, run the system update playbook. It handles temporarily disabling the read-only filesystem, upgrading packages, and re-enabling read-only mode automatically:

```sh
ansible-playbook -i ansible/inventory.ini ansible/playbooks/update-system.yml
```

### Testing and Linting

We use `yamllint` to ensure our Ansible playbooks and configuration files are properly formatted.

To run the linter across all YAML files in the repository:

```bash
yamllint .
```
