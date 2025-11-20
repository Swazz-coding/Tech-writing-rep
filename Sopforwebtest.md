# Standard Operating Procedure: Setup a Virtual Linux Server for Web Application Test

**Organization:** G3Company IT Operations  
**Department:** Systems Administration  
**Phone:** +1 (204) 555-1234  
**Document Owner:** Swathi Anil  
**Email:** swathi.anil@sanil.g3company.site  

---

## Revision Information

| Version | Date       | Author           | Changes Made                     |
|--------|------------|------------------|----------------------------------|
| 1.0    | 2025-11-20 | Swathi Anil      | Initial draft                    |

---

## Approval Table

| Role          | Name               | Email                      | Status     | Date       |
|---------------|--------------------|----------------------------|------------|------------|
| Author        | Swathi Anil        | swathi.anil@sanil.g3company.site | Drafted    | 2025-11-20 |
| Reviewer      | Bobbi Plante       | bobbi.plante@mitt.g3company.site | Pending    |            |
| Approver      | Marni Russell      | marni.russell@mitt.g3company.site| Pending    |            |

---

## Purpose

This Standard Operating Procedure (SOP) defines the step-by-step process for setting up a virtual Linux server on VMware vSphere 7.0, configured specifically for testing web applications. 

---

## Scope

This SOP applies to:
- System Administrators
- DevOps Engineers
- IT Support Staff
- Development Teams requiring test servers

It covers the creation of a CentOS Stream 9 virtual machine with Apache, PHP, MySQL (LAMP stack), Git, and basic firewall configuration for secure access. This procedure assumes access to VMware vSphere 7.0 environment and administrative privileges.

---

## Accountability Matrix

 Task                                | Responsible Party         | Emergency Contact       | Non-Emergency Contact     | Review/Approval       |
-------------------------------------|---------------------------|-------------------------|---------------------------|------------------------|
 Pre-creation Planning               | System Administrator      | On-call IT: 437-878-5117| IT Team Slack Channel     | IT Supervisor          |
 VM Creation & Configuration         | Virtualization Team       | On-call IT: 437-878-5117 | vmware-admin@g3company.site | IT Ops Manager           |
 OS Installation & LAMP Setup        | Systems Admin / DevOps    | On-call IT: 437-878-5117 | linux-team@g3company.site   | Senior Systems Engineer |
 Security & Firewall Configuration   | Security Officer (Optional)| Sec-Team-Pager          | security@g3company.site    | CISO Delegate            |
 Documentation & Handover            | Author / Knowledge Manager| N/A                     | docs@g3company.site          | Documentation Lead       |

---

## Definitions

 Term                | Definition                                                                 |
---------------------|----------------------------------------------------------------------------|
 VM                  | Virtual Machine – a software-based simulation of a physical computer.       |
 vSphere             | VMware’s virtualization platform for managing virtual machines.            |
 LAMP Stack          | Linux, Apache, MySQL, PHP – a common open-source web application stack.    |
 ISO                 | Disk image file used to install an operating system.                       |
 SSH                 | Secure Shell – encrypted protocol for remote command-line login.           |
 NAT                 | Network Address Translation – method for IP address translation in networks.|
 SELinux             | Security-Enhanced Linux – kernel security module providing access control. |

---

## Procedure Steps

### Step 1: Pre-Creation Planning and Assessment

#### 1.1 Define Requirements
- **Purpose**: Host internal web application prototypes and conduct functional testing.
- **Hostname**: `webtest01.sanil.g3company.site`
- **IP Address**: Assigned via DHCP from VLAN 105 (Test Network), or static if required (`192.168.105.x`)
- **OS Distro**: CentOS Stream 9 (x86_64)
- **Resource Allocation**:
  - vCPU: 2 cores
  - RAM: 4 GB
  - Storage: 40 GB thin-provisioned disk
  - Network: Single NIC connected to "Test_Network_VLAN105"

#### 1.2 Verify Resource Availability
- Log into vCenter Server and confirm sufficient CPU, memory, and datastore capacity on the target ESXi host.
- Ensure the CentOS Stream 9 ISO is available in the content library or datastore.

#### 1.3 Communicate with Stakeholders
- Notify development team of expected setup time.
- Confirm any special software or port requirements beyond standard LAMP stack.

---

### Step 2: Create and Configure the Virtual Machine

#### 2.1 Access vSphere Client
- Open VMware vSphere Client and log in using administrative credentials.
- Navigate to **Hosts and Clusters** > select target cluster.

#### 2.2 Initiate VM Creation
- Right-click the cluster > **New Virtual Machine** > **Create a new virtual machine**.
- Select compatibility: **ESXi 7.0 and later**.
- Choose guest OS family: **Linux**, version: **CentOS 8 (64-bit)** (compatible with Stream 9).

#### 2.3 Configure Hardware Settings
- **Name**: `webtest01`
- **Location**: `/Test-Servers/vApps`
- **Storage**: Select primary datastore with thin provisioning.
- **Disk Size**: 40 GB, mode: **Thin Provisioned**
- **CD/DVD Drive**: Connect to ISO image → browse and select `CentOS-Stream-9-x86_64-latest.iso`
- **Network Adapter**: Connected to **Test_Network_VLAN105**
- **CPU**: 2 vCPUs
- **Memory**: 4096 MB
- Enable **Guest OS Customization** (optional for future automation)

> ⚠️ **Note**: Disable unnecessary devices like floppy drive and printer for security hardening.

#### 2.4 Finish and Power On
- Click **Finish** to create the VM.
- Once created, right-click the VM > **Power On**.

---

### Step 3: Install CentOS Stream 9

#### 3.1 Boot from ISO
- After powering on, open the console.
- At boot menu, select **Install CentOS Stream 9**.

#### 3.2 Perform Installation
- **Language**: English (United States)
- **Installation Destination**: Select disk > click **Done**
- **Network & Hostname**:
  - Set hostname: `webtest01.internal.mitt.ca`
  - Turn ON Ethernet interface
  - Assign IP (DHCP recommended; use static only if required)
- **Root Password**: Set strong password (follow org policy). Do **not** enable root SSH login yet.
- **User Creation**: Create user `devuser` with sudo privileges.
- Begin installation.

#### 3.3 Complete Installation
- Wait for installation to finish.
- When prompted, remove CD-ROM and reboot.
- Log in locally or via console after reboot.

---

### Step 4: Post-Installation System Configuration

#### 4.1 Update System Packages
```bash
sudo dnf update -y
sudo dnf upgrade -y
```
#### 4.2: Set Timezone and NTP Sync
```bash
sudo timedatectl set-timezone America/Winnipeg
sudo systemctl enable chronyd --now
```
#### 4.3: Configure Firewall (firewalld)
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```
---
### Step 5: Install and Configure Web Application Test Environment

#### 5.1 Install LAMP Stack
#### Apache (HTTPD):
```bash
sudo dnf install httpd -y
sudo systemctl enable httpd --now
sudo systemctl status httpd
```
#### MySQL (MariaDB):
```bash
sudo dnf install mariadb-server mariadb -y
sudo systemctl enable mariadb --now
sudo mysql_secure_installation
```
#### PHP 8.1:
```bash
sudo dnf install php php-mysqlnd php-gd php-cli php-opcache -y
```

