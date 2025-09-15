# osTicket Lab on Azure

## Objective
In this lab, I deploy and configure osTicket on a Windows Server VM using XAMPP (Apache + PHP), running on Microsoft Azure Cloud. I also configured access controls via an NSG
(Network Security Groups), allowing only http(s)(port 80/433) and Remote Desktop Protocol (port 3389) from only my Host computer's IP address. 
<br> 
The goal was to explore MS Azure's Cloud platform, deploying VMs, and gain hands-on experience with a ticketing system- familiarizing myself with opening and assigning tickets, 
provisioning SLAs, setting priority, and commenting/closing tickets. Through the process of setting up this self-hosted platform, I ended up learning and practicing basic firewall 
hardening and enforcing least-privilege access! 
<br>

The basic diagram for my lab: 

![7BOMui8](https://github.com/user-attachments/assets/ce5c2b93-faf2-48e3-a0d1-4b921d906f13)

## The Process

### Deploying a Windows Server 2022 VM on Azure

Within Azure: 

- Created a New Resource Group, titled "osTicket Lab". This is to package everything together. 

<img width="753" height="270" alt="xGOimEG" src="https://github.com/user-attachments/assets/24e74f71-3e75-415a-ad61-daf99a68c596" />


Within the Resource Group, I create a Windows Server 2022 VM named "osTicket-lab1."

  - I provision it with 2 vcpus + 8gb RAM, and set Administrator account credentials. I leave the default disk size (127gb), more than enough.
    
<img width="880" height="819" alt="pOxRo6Y" src="https://github.com/user-attachments/assets/35a64f13-bd7d-4593-81d7-99661017fa3e" />

  -  A new Virtual network (vnet) is created for this VM. In the setup page, I allow port 443 (https)- these rules will be further configured later.

  -  Finishing the Deployment:
 <img width="410" height="351" alt="maw1OLw" src="https://github.com/user-attachments/assets/3e3e2c50-e102-4fff-97af-3ec75eee8215" />


 ### Configuring NSG Firewall rules

 Under Networking > Network Settings, I set the following Inbound Rules:
 - Allow port 3389 with my IP as the source, to restrict RDP access only from MY IP address, to ensure least privilege.
 - Allow port 80 for HTTP, allowing me to connect to the osTicket server from my host (or any) computer via the VM's public IP.
 - Allow 443 for HTTPs if SSL is enabled. Not entirely needed, but good practice.
 - DenyAllInBound rule. Azure automatically creates this rule to deny all other inbound traffic except for specified allowed ports.
   
<img width="1384" height="330" alt="xpkUKyl" src="https://github.com/user-attachments/assets/601a6443-1664-410c-bfe1-571f62d377b4" />

### Connecting to the VM via RDP

I'm now able to connect via RDP. Running Remote Desktop Protocol (mstsc), I enter the VM's public IP (20.x.x.x), enter in my admin credentials, and successfully connect!

### Downloading and installing XAMPP
XAMPP is an open source web server platform that bundles Apache, MariaDB, PHP, and Perl. 
Why XAMPP? For Windows, it's a basic way for me to host a local web server with Apache, mySQL, and PHP. This is also possible on Ubuntu Server via a LAMP stack (Apache, MySQL, PHP). 

- On the VM, download [XAMPP](https://www.apachefriends.org/index.html) for Windows
- Allow all components and install to the default directory (C:\xampp)
- When installed, we're greeted with the control panel: 
<img width="669" height="439" alt="JRzjpTK" src="https://github.com/user-attachments/assets/6c560ae3-607a-45ce-a1c2-916b3651e013" />

### XAMPP configuration
We will configure XAMPP's properties to set our domain as the Azure VM's public IP. Down the line, this allows us to access osTicket via our public IP when it is deployed via XAMPP.
- Navigate to `C:\xampp\properties` and edit `apache_domainname=` to be the public IP. By default, it's the recursive address.
- Next, we'll configure myPHP to resolve at the same address. Navigate to `C:\xampp\phpMyAdmin` and locate `config.inc.php` . Edit the value to be the public IP address instead.
<img width="504" height="79" alt="6bgB5hA" src="https://github.com/user-attachments/assets/4111f4de-cb06-4a61-8ef6-54d7f645888f" />
- In `config.inc.php` , I will also edit the authentication credentials and set a password. 
