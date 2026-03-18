<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop (RDP)
- Active Directory Domain Services
- PowerShell
- DNS

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- 
- 
- 
- 

<h2>Deployment and Configuration Steps</h2>


## Lab Goals

By the end of this lab, the following will be accomplished:

- A Windows Server VM deployed as a domain controller in Azure
- Active Directory Domain Services installed and configured
- Organizational units, users, and security groups created
- A Windows 10 client joined to the domain
- Domain authentication validated end-to-end

---

## Deployment Overview

1. Provision two Azure VMs (domain controller and client)
2. Install and configure Active Directory Domain Services
3. Promote the server to a domain controller
4. Configure DNS on the client machine
5. Join the client to the domain
6. Validate authentication and directory functionality

---

## Step-by-Step Configuration

### 1. Set up Azure Virtual Machines

Two virtual machines are required for this lab.

**Domain Controller — DC-01**

- OS: Windows Server 2022
- Size: Minimum 2 vCPUs
- Virtual Network: New VNet created during VM provisioning
- Private IP: Set to **Static**

> Setting a static private IP on the domain controller is important as a dynamic IP could cause the DNS and domain join to fail if the address changes after a restart.

**Client Machine — CLIENT-01**

- OS: Windows 10 (21H2)
- Virtual Network: Same VNet and subnet as DC-01

> Both machines must share the same virtual network for domain communication to function properly.

---

### 2. Install Active Directory Domain Services

1. Log into **DC-01** via Remote Desktop
2. Open **Server Manager**
3. Select **Add Roles and Features**
4. Install **Active Directory Domain Services** (and DNS Server if prompted)
5. Complete the installation wizard

---

### 3. Promote DC-01 to a Domain Controller

1. In Server Manager, click the **AD DS notification flag**
2. Select **Promote this server to a domain controller**
3. Choose **Add a new forest**
4. Set the root domain name to `corp.local`
5. Configure the Directory Services Restore Mode (DSRM) password
6. Complete the wizard and allow the server to reboot

After rebooting, DC-01 is now a functioning domain controller for `corp.local`.

---

### 4. Create Organizational Units, Users, and Groups

**Organizational Units**

Open **Active Directory Users and Computers** and create the following OUs:

- `IT`
- `Employees`
- `Group Computers`

> Custom OUs are used to model a realistic enterprise directory structure. Default AD containers are left intact as they are system-managed.

**Domain Users**

Create a small set of test users to demonstrate directory functionality:

| Username | Role |
|---|---|
| jdoe | Standard user |
| asmith | Standard user |
| itadmin | IT administrator |

**Security Groups**

Create the following security groups within the appropriate OUs and assign users accordingly:

| Group | Members |
|---|---|
| Domain-Users | jdoe, asmith |
| IT-Admins | itadmin |

> Using group-based permissions rather than individual user assignments reflects enterprise access control best practices and simplifies ongoing management.

---

### 5. Configure Client DNS and Join the Domain

**Set DNS on CLIENT-01**

1. Log into **CLIENT-01** via Remote Desktop
2. Open **Network Adapter Settings**
3. Set the **Preferred DNS Server** to the **static private IP of DC-01**

This ensures the client can resolve the domain controller and locate domain services.

**Join the Domain**

1. Open **System Properties** → **Change settings**
2. Select **Domain** and enter `corp.local`
3. Authenticate with a domain administrator account
4. Reboot when prompted

---

### 6. Validate Domain Functionality

After CLIENT-01 reboots:

- Log in using a domain user account (e.g., `corp\jdoe`)
- Confirm successful domain authentication
- Verify group membership is applied correctly
- Confirm CLIENT-01 appears in Active Directory under the **Group Computers** OU
- Test DNS name resolution and connectivity to DC-01

---

## Outcome

At the conclusion of this lab:

- Active Directory is fully deployed and operational within Azure
- Domain services replicate a traditional on-premises architecture
- Users authenticate against the domain from a joined client machine
- Core identity and directory management concepts are demonstrated hands-on

---

## Possible Extensions

This lab can be expanded in several directions:

- Group Policy Object (GPO) configuration
- File shares with NTFS and share permissions
- Domain-based software deployment
- Azure VPN or hybrid connectivity scenarios
- Security auditing and hardening

---

## Skills Demonstrated

This project reflects practical knowledge of:

- Identity and access management fundamentals
- Windows Server administration
- Cloud-hosted infrastructure with on-premises architecture
- Enterprise directory services design and management

These skills are directly applicable to help desk, IT support, systems operations, and junior system administrator roles.
