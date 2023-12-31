![HeaderPix](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/8acbe3f3-7642-4a49-af93-6753a901816c)

<h2>Introduction</h2>
Active Directory, developed by Microsoft, is a powerful directory service that centralizes user management, enhances security, and simplifies administrative tasks within a network environment. It offers an excellent opportunity to gain hands-on experience and establish a strong foundation in this essential technology by setting up a home lab.

<h2>Objective</h2>
This project aims to guide individuals, from beginners to those with advanced knowledge, through the process of creating a basic home lab with Active Directory. By creating a fully functional Active Directory environment, users can learn, experiment, and simulate real-world scenarios.

<h2>Lab Setup</h2>
Using VMWare Workstation 15 Player, set up the following virtual machines:

- 1 x Windows Server 2019 (Domain controller)
- 1 x Windows 10 Enterprise — User-machine 1
- 1 x Windows 10 Enterprise — User-machine 2
- 1 x Kali Linux — Attacker

Each virtual machine should have approximately 2GB of RAM, totaling 8GB RAM for all machines.

Download the required ISO from the official Microsoft Evaluation Center website: https://www.microsoft.com/en-us/evalcenter/

![1](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/01106fbc-fc19-4f99-99f9-2204dcd44abf)
![2](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/de8dac1e-1bc6-42f4-8303-33bc75e87236)
![3](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/42223cf5-0e42-4d2e-9529-1628d9bc78a3)
![4](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/4cd25f94-8be5-48e1-8904-840745bd8008)
![5](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/bc7e1a49-3c1d-43a7-b750-fa9c2661e46c)
![6](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/5c44cd10-82ff-4974-95d0-4afeb69d923c)
![7](https://github.com/fritz1reikoz/Active-Directory-Home-Lab/assets/55414490/0a28af26-deae-4a69-bcf3-d2176ba12561)


<h2>Virtual Machine Setup</h2>
Remove floppy drives and set the network to NAT. Perform the following steps for each virtual machine:

<h4>Domain Controller (OS Installation)</h4>

- Create a new virtual machine.
- Select the ISO file for the installer disc image.
- Choose “Windows Server 2019” as the version to install.
- Power on the virtual machine after creation.
- In VM Settings, remove the floppy disk and set the network to NAT.
- Start the virtual machine and press any key to enter setup mode.
- Select “Windows Server 2019 Standard Eval (Desktop Experience)”.
- Choose “Custom install” and select the unallocated space for installation.
- Follow the on-screen instructions to complete the installation.

<h4>Setting up Domain Controller</h4>

- In the virtual machine settings, go to “View Your PC Name” and rename the PC to something like myDomainController. Restart the virtual machine.
- Set a password for the PC, such as P@$$w0rd.
- In VMWare Player, go to “Manage” and install VMWare Tools.
- Open Server Manager and navigate to “Dashboard” > “Manage” > “Add Roles and Features”.
- Choose “Role-based or feature-based installation” and select the Active Directory Domain Services role. Continue with the installation.
- After the installation, click the flag icon in Server Manager and select “Promote this server to a domain controller”.
- Choose “Add a new forest” and enter a root domain name like ADHACKING.local. Proceed with the installation, setting a password for DSRM (Directory Services Restore Mode) and accepting the default options.

<h4>User Machine (OS Installation)</h4>

Perform the following steps for each user machine:

- Create a new virtual machine.
- Select the ISO file for the installer disc image.
- Choose “Windows 10 Enterprise” as the version to install.
- Power on the virtual machine after creation.
- In VM Settings, remove the floppy disk and set the network to NAT.
- Start the virtual machine and press any key to enter setup mode.
- Select “Custom install” and select the unallocated space for installation.
- Choose “Domain join instead” and enter an account name (e.g., saul goodman) and password (Password1).
- Proceed with the installation, selecting “No” for “Do more across devices…” and declining “Get help from digital assistant”. Choose your preferred privacy settings.

<h4>Setting up User Machine</h4>

- In VMWare Player, go to “Manage” and install VMWare Tools.
- In the virtual machine settings, go to “View Your PC Name” and rename the PC (e.g., <i>helpdesk</i>). Restart the virtual machine.
- Repeat these steps for the second user machine, using the account name <i>walter white</i>, password <i>Password1</i>, and PC name <i>Walter-PC</i>.

<h2>Active Directory (AD) Setup, Groups, and Policies</h2>

Log in to the Windows Server

(Domain Controller) and perform the following steps:

- Open Server Manager and go to “Tools” > “Active Directory Users and Computers”.
- Right-click on the root domain (e.g., ADHACKING.local) and select "New" > "Organizational Unit" > "Groups".
- Move all users (except Administrator and Guest) from the “Users” directory to the “Groups” directory.
- Right-click under the “Users” directory and create the following users:
- Uncheck “User must change password at next logon” and check “Password never expires”.
- Copy the “administrator” domain admin account with the login gus and password Password2020@!.
- Copy the “administrator” domain admin account with the logon SQLService, password MYpassword123#, and description "Password is MYpassword123#".
- Create a new user domain account with the logon saul and password Password123.
- Create a new user domain account with the logon walter and password Password123.

<b>Note:</b> It is generally not recommended to assign domain-admin rights to service accounts like SQL Service. Although this is a lab example, in real-life situations, granting domain-admin rights to such services should be avoided. Similarly, storing passwords in the description field is not a secure practice, i only do this because it's a homelab for testing.

<h2>Configuring File Server (Opening SMB)</h2>

Enable SMB file sharing and open ports 445 and 139 for later lab activities.

- Open Server Manager and go to “Dashboard” > “File and Storage Services” > “Shares”.
- Click “New Share” > “SMB Share — Quick” > “Next”.
- Set the share name as “hackme” and follow the prompts to create the share.

<h2>Creating Service Principal Name (SPN)</h2>

Set up for a Kerberoasting attack.

- Open the Command Prompt as an administrator.
- Use the following command to set the SPN: setspn -a myDomainController/SQLService.ADHACKING.local:60111 ADHACKING\\\\SQLService.
- Check if the SPN is set using the command: setspn -T ADHACKING.local -Q */*.

<h2>Group Policy Configuration</h2>

Disable Windows Defender for lab purposes (note: topics like AV bypass and evasion are not covered in this lab).

- Open Group Policy Management as an administrator.
- Expand the Forest > Domains > ADHACKING.local.
- Right-click and create a new Group Policy Object (GPO) named “Disable Windows Defender” in this domain.
- Edit the newly created GPO.
- Navigate to “Computer Configuration” > “Policies” > “Administrative Templates” > “Windows Components” > “Windows Defender Antivirus”.
- Double-click on “Turn off Windows Defender Antivirus” and enable the policy.

<h2>Connecting all Machines</h2>

Perform the following steps on both machines:

- Create a folder and set up a network share.
- On the first machine, give the first domain user local administrator rights.
- On the second machine, give both domain users local administrator rights.
- Create a network share folder.
- Create a folder named “share” in the C: drive.
- Right-click on the folder, go to “Properties” > “Sharing” > “Share” > “Share” > “Yes, turn on network discovery…”.
- Configure network settings.
- Open network & internet settings, change adapter options, and go to IPv4 > Preferred DNS Server.
- Enter the IP address of the Domain Controller.
- Join the domain.
- Go to “Access work or school” > “Connect” > “Join this device to local Active Directory domain”.
- Enter the domain name as ADHACKING.local.
- Join the domain as the Administrator with the password P@$$w0rd.
- When prompted to add an account, skip and restart the machine.
- Upon restart, select the other user and log in as the domain user (e.g., saul with the password Password123).
  
- <h4>Give local administrator rights:</h4>
1. Go to “Computer Management” > “Local Users and Groups” > “Groups”.
2. Double-click on “Administrators”.
3. Add the user (e.g., saul), check the name, apply, and click OK.

- <h4>Turn on Network Discovery:</h4>
1. Go to “Computer” > “Network”.
2. Click OK when prompted and click “Turn on network discovery…” in the top menu bar.

- <h4>Verify that both computers have joined the domain:</h4>
1. On the Domain Controller, open “Active Directory Users and Computers” > ADHACKING.local > "Computers".
2. Check if both computers are added.

- <h4>Set up for mitm6 attack lab example:</h4>

1. Login to the Domain Controller.
2. Open Server Manager > Dashboard > “Add roles and features” > Next.
3. Select “Active Directory Certificate Services” > Add features > Next.
4. Choose to restart the destination server automatically and click Install.
5. Click the flag notification icon and configure Active Directory services on the destination server.
6. Provide the credentials as ADHACKING\\\\Administrator.
7. Proceed with the configuration, selecting the Certification Authority and setting the validity to 99 years. Click Next and configure the settings.


<h2>Conclusion</h2>
Constructing a home lab with Active Directory offers a special chance to learn more, advance your understanding, and hone your abilities in this vital technology. I will have a platform to increase my knowledge, hone my abilities, and unleash the potential of this tremendous technology by setting up an Active Directory lab environment. For novices, this project provides step-by-step instructions; for experts, it provides insightful information. It is imperative that IT workers in network administration, security, and system management jobs comprehend Active Directory and its features. Invest in your career advancement and open doors to intriguing prospects in the IT industry by learning Active Directory abilities.
