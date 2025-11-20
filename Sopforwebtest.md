# Standard Operating Procedure: Setup a Virtual Linux Server for Web Application Test

**g3company**  
**IT Operations Department**  
**Phone:** +1 (437) 878-4117  

---

## Revision Information

| Version | Date       | Author           | Changes Made                     |
|--------|------------|------------------|----------------------------------|
| 1.0    | 2025-11-20 | Swathi Anil      | Initial draft                    |

---

## Approval Table

| Version | Date       | Name               | Email                      | Role       |
|--------|------------|--------------------|----------------------------|------------|
| 1.0    | 2025-11-20 | Bobbi Plante       | bobbi.plante@mitt.ca       | Reviewer   |
| 1.0    | 2025-11-20 | Marni Russell      | marni.russell@mitt.ca      | Approver   |

---

## Purpose

This Standard Operating Procedure (SOP) defines the standardized process for setting up a virtual Linux server on VMware vSphere 7.0, specifically configured for testing web applications. The objective is to ensure consistent, secure, and reproducible deployment of isolated test environments using CentOS Stream 9 with a LAMP stack (Linux, Apache, MySQL, PHP). This SOP supports rapid provisioning of development and QA servers while adhering to internal security and operational best practices.

Expected outcomes include:
- A fully functional, secure Linux-based web test server.
- Verified network connectivity and service operation.
- Complete documentation for audit, troubleshooting, and future maintenance.
- Compliance with g3company’s IT infrastructure standards.

---

## Scope/Objectives

This SOP applies to:
- System Administrators
- DevOps Engineers
- IT Support Staff
- Development and QA Teams requiring non-production web application testing environments

This procedure covers:
1. Pre-creation planning and resource assessment.
2. Creation and configuration of a virtual machine in VMware vSphere 7.0.
3. Installation and post-installation setup of CentOS Stream 9.
4. Deployment of essential software packages for web application testing (LAMP stack).
5. Security hardening including firewall rules, SELinux, and removal of unnecessary services.
6. Functional verification of web and database services.
7. Documentation and knowledge transfer for ongoing management.

This SOP does **not** apply to production systems, cloud-hosted instances (AWS, Azure), or environments outside the g3company on-premises vSphere infrastructure.

---

## Accountability Matrix

| Task/Steps                              | Responsible Person/Team     | Emergency Contact         | Non-Emergency Contact       | Review/Approval       |
|-----------------------------------------|-----------------------------|---------------------------|-----------------------------|------------------------|
| Pre-creation Planning and Assessment    | System Administrator        | On-call IT: 437-878-4117   | it-team@g3company.site      | IT Manager              |
| Configuration of Virtual Machine        | Virtualization Team         | On-call IT: 437-878-4117   | vmware-admin@g3company.site | IT Ops Manager          |
| Installation of Guest OS & LAMP Stack   | System Administrator        | On-call IT: 437-878-4117   | linux-team@g3company.site   | Senior Systems Engineer |
| Post-creation Verification and Testing  | Quality Assurance Team      | QA-OnCall-Pager            | qa-team@g3company.site      | QA Manager              |
| Documentation of Virtual Machine        | Documentation Lead / Author | N/A                        | docs@g3company.site         | Knowledge Manager       |

---

## Definitions

| Term                | Definition                                                                 |
|---------------------|----------------------------------------------------------------------------|
| VM                  | Virtual Machine – a software-based emulation of a physical computer.       |
| vSphere             | VMware’s virtualization platform for managing virtual machines.            |
| LAMP Stack          | Linux, Apache, MySQL, PHP – a common open-source web application stack.    |
| ISO                 | Disk image file used to install an operating system.                       |
| SSH                 | Secure Shell – encrypted protocol for remote command-line login.           |
| NAT                 | Network Address Translation – method for IP address translation in networks.|
| SELinux             | Security-Enhanced Linux – kernel security module providing access control. |
| CMDB                | Configuration Management Database – system for tracking IT assets.         |
| HTTP/HTTPS          | Hypertext Transfer Protocol(Secure) – protocols used for serving web content. |
| DHCP                | Dynamic Host Configuration Protocol – assigns IP addresses automatically.  |

---

## Procedure Steps

### Step 1: Pre-creation Planning and Assessment

#### Step 1.1: Identify Purpose and Requirements
- Determine the purpose: Hosting internal web application prototypes and conducting functional testing.
- Define specifications:
  - **Hostname**: `webtest01.internal.g3company.site`
  - **IP Assignment**: DHCP from VLAN 105 (Test Network); static if required
  - **OS Distro**: CentOS Stream 9 (x86_64)
  - **vCPU**: 2 cores
  - **RAM**: 4 GB
  - **Storage**: 40 GB thin-provisioned disk
  - **Network**: Connected to "Test_Network_VLAN105"

> ⚠️ These values are minimum recommendations; adjust based on application requirements.

#### Step 1.2: Plan Resource Allocation
- Log into vCenter Server and verify available CPU, memory, and datastore capacity.
- Confirm that the target ESXi host has sufficient resources.
- Ensure the `CentOS-Stream-9-x86_64-latest.iso` is available in the content library or datastore.

#### Step 1.3: Communicate with Stakeholders
- Notify development or QA team of expected setup timeline.
- Confirm any special requirements such as specific ports, firewall rules, or third-party tools.

---

### Step 2: Configuration of Virtual Machine

#### Step 2.1: Access VMware vSphere Client
- Open VMware vSphere Client and log in with administrative credentials.
- Navigate to **Hosts and Clusters** > select the appropriate cluster.

#### Step 2.2: Create New Virtual Machine
- Right-click the cluster > **New Virtual Machine** > **Create a new virtual machine**.
- Select compatibility: **ESXi 7.0 and later**.
- Choose guest OS family: **Linux**, version: **CentOS 8 (64-bit)** (compatible with Stream 9).

#### Step 2.3: Configure Virtual Machine Settings
- **Name**: `webtest01`
- **Location**: `/Test-Servers/vApps`
- **Storage**: Select primary datastore with thin provisioning.
- **Disk Size**: 40 GB, mode: **Thin Provisioned**
- **CD/DVD Drive**: Connect to ISO → browse and select `CentOS-Stream-9-x86_64-latest.iso`
- **Network Adapter**: Connected to **Test_Network_VLAN105**
- **CPU**: 2 vCPUs
- **Memory**: 4096 MB
- Enable **Guest OS Customization** (optional for automation)

> ⚠️ Disable unnecessary devices like floppy drive, printer, and USB controllers for security hardening.

#### Step 2.4: Finish and Power On
- Click **Finish** to create the VM.
- Once created, right-click the VM > **Power On**.

---

### Step 3: Installation of Guest OS

#### Step 3.1: Attach Installation ISO
- After powering on, open the console.
- At boot menu, select **Install CentOS Stream 9**.

#### Step 3.2: Install Guest OS
- **Language**: English (United States)
- **Installation Destination**: Select disk > click **Done**
- **Network & Hostname**:
  - Set hostname: `webtest01.internal.g3company.site`
  - Turn ON Ethernet interface
  - Assign IP via DHCP or set static if needed
- **Root Password**: Set strong password per organizational policy.
- **User Creation**: Create standard user `devuser` with sudo privileges.
- Begin installation and wait for completion.

#### Step 3.3: Configure Networking
- After reboot, confirm network connectivity:
  ```bash
  ip addr show
  ping -c 4 google.com
  ```
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
### Step 6: Post-creation Verification and Testing
#### 6.1: Confirm Services Are Running
```
systemctl is-active httpd mariadb
```
