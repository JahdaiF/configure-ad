<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Video Demonstration</h2>

- <b>Photos and video Coming Soon</b> 

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop (RDP)
- Active Directory Domain Services
- PowerShell
- DNS

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)



<h2>Deployment and Configuration Steps</h2>


<b> Lab Goals </b>

By the end of this lab, the following will be accomplished:

- A Windows Server VM deployed as a domain controller in Azure
- Active Directory Domain Services installed and configured
- Organizational units, users, and security groups created
- A Windows 10 client joined to the domain
- Domain authentication validated end-to-end

---

<b> Deployment Overview </b>

1. Provision two Azure VMs (domain controller and client)
2. Install and configure Active Directory Domain Services
3. Promote the server to a domain controller
4. Configure DNS on the client machine
5. Join the client to the domain
6. Validate authentication and directory functionality

---

<b> Step-by-Step Configuration </b>

<b> 1. Set up Azure Virtual Machines </b>

Two virtual machines are required for this lab.

<b>Domain Controller — DC-01</b>

- OS: Windows Server 2022
- Size: Minimum 2 vCPUs
- Virtual Network: New VNet created during VM provisioning
- Private IP: Set to <b>Static</b>

> Setting a static private IP on the domain controller is important as a dynamic IP could cause the DNS and domain join to fail if the address changes after a restart.

<b>Client Machine — CLIENT-01</b>

- OS: Windows 10 (21H2)
- Virtual Network: Same VNet and subnet as DC-01

> Both machines must share the same virtual network for domain communication to function properly.

---

<b> 2. Install Active Directory Domain Services</b>

1. Log into <b>DC-01</b> via Remote Desktop
2. Open <b>Server Manager</b>
3. Select <b>Add Roles and Features</b>
4. Install <b>Active Directory Domain Services</b> (and DNS Server if prompted)
5. Complete the installation wizard

---

<b>3. Promote DC-01 to a Domain Controller</b>

1. In Server Manager, click the <b>AD DS notification flag</b>
2. Select <b>Promote this server to a domain controller</b>
3. Choose <b>Add a new forest</b>
4. Set the root domain name to <b>mydomain.com</b>
5. Configure the Directory Services Restore Mode (DSRM) password (Password1)
6. Complete the wizard and allow the server to reboot

After rebooting, DC-01 is now a functioning domain controller for <b>mydomain.com</b>.

---

<b> 4. Create Organizational Units, Users, and Groups</b>

<b>Organizational Units</b>

Open **Active Directory Users and Computers** and create the following OUs:

- <b>_Admins</b>
- <b>_Employees</b>
- <b>_Clients</b>

> Custom OUs are used to model a realistic enterprise directory structure. Default AD containers are left intact as they are system-managed.

<b>Domain Users</b>

Create a small set of test users to demonstrate directory functionality:

| Username | Role |
|---|---|
| jane_admin | IT Administrator |
| asmith | Employee |
| Bob | Employee |

<b>Bulk User Creation via PowerShell</b><br>
In addition to the manually created accounts above, a PowerShell script was used to bulk create <b>10,000 employee accounts</b> within the <b>_Employees</b> OU. This simulates the process used in real-world IT environments where user accounts are created programmatically at scale. In a real environment, the script would pull from an HR data source rather than generating random names.<br>
<br>

The script automated:

- Random username generation using alternating consonants and vowels
- Secure password assignment (uniform password for lab purposes)
- Automatic placement into the **_Employees** OU
- Enabling all accounts upon creation

After the script completed, the accounts were verified in **Active Directory Users and Computers (ADUC)** under the **_Employees** OU. I then used one of the manually created domain accounts to successfully log into **CLIENT-01**, confirming end-to-end domain authentication was working correctly.

> In a production environment, account creation would be tied to an HR onboarding workflow with real employee data rather than randomly generated names.

*Script provided as part of course curriculum.* <b>View the full PowerShell script here:</b> <a href="https://github.com/JahdaiF/configure-ad/blob/main/Generate-Names-Create-Users.ps1">Generate-Names-Create-Users.ps1</a>

---
**Security Groups**

Create the following security groups within the appropriate OUs and assign users accordingly:

| Group | Members |
|---|---|
| Domain-Users | Bob, asmith |
| IT-Admins | jane_admin |

> Using group-based permissions rather than individual user assignments reflects enterprise access control best practices and simplifies ongoing management.

---

<b> 5. Configure Client DNS and Join the Domain </b>

<b>Set DNS on CLIENT-01</b>

1. Log into <b>CLIENT-01</b> via Remote Desktop
2. Open <b>Network Adapter Settings</b>
3. Set the <b>Preferred DNS Server</b> to the <b>static private IP of DC-01</b>

> <b>Troubleshooting Note:</b> This step is critical. Without pointing the client's DNS to the Domain Controller, the workstation will be unable to locate <b>mydomain.com</b> and the domain join will fail.

<b>Join the Domain</b>

1. Open <b>System Properties</b> → <b>Change settings</b>
2. Select <b>Domain</b> and enter <b>mydomain.com</b>
3. Authenticate with a domain administrator account (e.g., <b>jane_admin</b>)
4. Reboot when prompted

---

<b> 6. Organize and Validate Domain Functionality </b>

<b>Organize the Directory (On DC-01)</b>

Once the client has joined the domain, it must be organized within Active Directory:

1. Log into <b>DC-01</b> and open <b>Active Directory Users and Computers (ADUC)</b>
2. Navigate to the default <b>Computers</b> container to find <b>CLIENT-01</b>
3. Right-click <b>CLIENT-01</b> and select <b>Move</b>
4. Choose the <b>_Clients</b> OU created in Step 4

> Moving the computer object to a custom OU is a best practice that allows for specific Group Policy Objects (GPOs) to be applied to workstations later.

<b>Final Validation</b>

After <b>CLIENT-01</b> reboots, perform the following checks:

- Log in using a domain user account (e.g., <b>mydomain.com\asmith</b>)
- Confirm successful domain authentication and desktop initialization
- Verify group membership is applied correctly
- Confirm <b>CLIENT-01</b> appears in Active Directory under the <b>_Clients</b> OU
- Test DNS name resolution and connectivity by pinging <b>DC-01</b>

---

<h2>Outcome</h2>

At the conclusion of this lab:

- Active Directory is fully deployed and operational within Azure
- Domain services replicate a traditional on-premises architecture
- Users authenticate against the domain from a joined client machine
- Core identity and directory management concepts are demonstrated hands-on

---
<h2>Troubleshooting</h2>

One key challenge during this lab was ensuring CLIENT-01 could locate the domain controller. Initially the domain join failed because the client was using the default Azure DNS rather than DC-01.

<b>Solution:</b> I navigated to the Azure Network Interface settings for CLIENT-01 and manually set the DNS server to the static private IP of DC-01. After restarting the VM the domain join completed successfully.

<h2>Possible Extensions</h2> 

This lab can be expanded in several directions:

- Group Policy Object (GPO) configuration
- File shares with NTFS and share permissions
- Domain-based software deployment
- Azure VPN or hybrid connectivity scenarios
- Security auditing and hardening

---

<h2>Skills Demonstrated</h2>

This project reflects practical knowledge of:

- Identity and access management fundamentals
- Windows Server administration
- Cloud-hosted infrastructure with on-premises architecture
- Enterprise directory services design and management

These skills are directly applicable to help desk, IT support, systems operations, and junior system administrator roles.
