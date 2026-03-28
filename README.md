# debian-to-parrotos-conversion
 The following Ansible replicates the logic of ParrotSec Debian Conversion Script. 
 
## What For?
The main goal of this script is to replicate the installation of core and headless packages done by the ParrotSec script, to then also install the pentesting tools suite offered by them to have a fully converted Debian machine into a headless ParrotOS for pentesting engagements, all through Ansible.
Also includes post-installation ansible to configure shell for all users.


Base script by ParrotSec: https://gitlab.com/parrotsec/project/debian-conversion-script



### **Workflow**
1. **Deploy the Debian Host**:
   - Make sure to configure your SSH access accordingly.

2. **Run the Ansible Playbook**:
   - Execute the playbook to convert the VM:
     ```bash
     ansible-playbook -i inventory.ini debian_to_parrotos.yml
     ```

3. **Verify the Conversion**:
   - SSH into the VM and confirm the following:
     - The shell prompt reflects Parrot OS customization.
     - Penetration testing tools are installed (`nmap`, `metasploit`, etc.).



## Technical Details

### **Ansible Script Overview**
The playbook performs the following tasks:

#### Core Tasks
- **System Preparation**:
  ```yaml
  - name: Update and Upgrade the System
    apt:
      update_cache: yes
      upgrade: dist
  ```
- **Install Dependencies**:
  ```yaml
  - name: Install Required Dependencies
    apt:
      name:
        - curl
        - gnupg
        - wget
      state: present
  ```

#### Adding Repositories
- Copy Parrot OS repository files:
  ```yaml
  - name: Copy Parrot OS sources.list
    copy:
      src: files/sources.list
      dest: /etc/apt/sources.list
  ```
- Add the GPG key by downloading and installing the official keyring package:
  ```yaml
  - name: Download Parrot OS Archive Keyring Package
    get_url:
      url: "https://deb.parrot.sh/parrot/pool/main/p/parrot-archive-keyring/parrot-archive-keyring_2024.12_all.deb"
      dest: "/tmp/parrot-archive-keyring_2024.12_all.deb"

  - name: Install Parrot OS Archive Keyring
    apt:
      deb: "/tmp/parrot-archive-keyring_2024.12_all.deb"
      state: present
  ```

#### Conversion to Parrot OS
- Install core and headless packages:
  ```yaml
  - name: Install Parrot Core and Headless Packages
    apt:
      name:
        - parrot-core-lite
        - base-files
        - parrot-apps-basics
        - parrot-drivers
      state: present
  ```
- Install penetration testing tools (optional, disabled by default):
  ```yaml
  - name: Install Parrot Tools Full Suite
    apt:
      name: parrot-tools-full
      state: present
    when: install_pentesting_tools | bool
  ```
  Controlled by the `install_pentesting_tools` variable (default: `false`). Enable at runtime:
  ```bash
  ansible-playbook -i inventory.ini debian_to_parrotos.yml -e "install_pentesting_tools=true"
  ```
  Or set per-host in your inventory:
  ```ini
  [parrot_vms]
  myhost ansible_host=192.168.1.10 install_pentesting_tools=true
  ```

#### Finalization
- Reboot the system:
  ```yaml
  - name: Reboot the VM to Finalize Installation
    reboot:
      msg: "Rebooting to finalize Parrot OS installation."
      pre_reboot_delay: 5
      post_reboot_delay: 10
  ```


---


