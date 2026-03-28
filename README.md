# debian-to-parrotos-conversion
 The following Ansible replicates the logic of ParrotSec Debian Conversion Script. 
 
## What For?
The main goal of this script is to replicate the installation of core and headless packages done by the ParrotSec script, to then also install the pentesting tools suite offered by them to have a fully converted Debian machine into a headless ParrotOS for pentesting engagements, all through Ansible.
Also includes post-installation ansible to configure shell for all users.


Base script by ParrotSec: https://gitlab.com/parrotsec/project/debian-conversion-script



### **Workflow**
1. **Deploy the Debian VM**:
   - Use a cloud-init YAML file to deploy the VM and configure it to connect back to the infrastructure via VPN.

2. **Run the Ansible Playbook**:
   - Execute the playbook to convert the VM:
     ```bash
     ansible-playbook -i inventory.ini debian_to_parrot.yml
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
- Add the GPG key:
  ```yaml
  - name: Add Parrot OS GPG Key
    shell: |
      wget -qO- https://deb.parrotsec.org/parrot/misc/parrotsec.gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/parrot-archive-keyring.gpg
    args:
      creates: /etc/apt/trusted.gpg.d/parrot-archive-keyring.gpg
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
- Install penetration testing tools:
  ```yaml
  - name: Install Parrot Tools Full Suite
    apt:
      name: parrot-tools-full
      state: present
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


