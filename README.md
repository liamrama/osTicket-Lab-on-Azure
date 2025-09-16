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

### (Adding Inbound rule to Windows Firewall)
Within the VM, I also addded an Inbound rule in Windows Defender Firewall, allowing port 80 and 443 (http and https). 

### Connecting to the VM via RDP

I'm now able to connect via RDP. Running Remote Desktop Protocol (mstsc), I enter the VM's public IP (20.x.x.x), enter in my admin credentials, and successfully connect!

### Downloading and installing XAMPP
XAMPP is an open source web server platform that bundles Apache, MariaDB, PHP, and Perl. 
Why XAMPP? For Windows, it's a basic way for me to host a local web server with Apache, mySQL, and PHP. This is also possible on Ubuntu Server via a LAMP stack (Apache, MySQL, PHP). 

- On the VM, download [XAMPP](https://www.apachefriends.org/index.html) for Windows
- Allow all components and install to the default directory (C:\xampp)
- When installed, we're greeted with the control panel: 
<img width="669" height="439" alt="JRzjpTK" src="https://github.com/user-attachments/assets/6c560ae3-607a-45ce-a1c2-916b3651e013" />


### Configuring phpMyAdmin and Creating a MySQL Database for osTicket
From here, we start the Apache and MySQL services, and select `Admin` option for Apache. This takes us to the phpMyAdmin page. 
<img width="666" height="433" alt="uh0bORb" src="https://github.com/user-attachments/assets/59cadf62-188e-4de8-bff8-bac036e646ee" />
<img width="1917" height="1035" alt="lBV6gX8" src="https://github.com/user-attachments/assets/b5468db3-a4c8-4f05-a8b7-32b097f6c3d8" />

For this lab, I'm choosing to keep it locally hosted, so the host name does not need to be changed. I do want to set a password for the phpMyAdmin user, for enhanced security. 
In the admin page, under `User accounts` , I select `'root'@'localhost'` and navigate to `Change password` I set a new password.

<img width="1190" height="482" alt="JrmlN52" src="https://github.com/user-attachments/assets/addec4c4-0b92-4514-b773-742f722f08c4" />

Select `Go` at the bottom of the page to apply settings. 

Navigate to `C:\xampp\phpMyAdmin` and locate `config.inc.php` . By default, the host is set to the localhost domain- 127.0.0.1.
Edit the `password` value to be the same password set in the phpMyAdmin portal. 

We'll now create the Database for osTicket, where all the information will live in. 
In the phpMyAdmin portal, navigate to the `Databases` tab, and under `Create database`, I will create one named `osticket`. 
<img width="956" height="255" alt="BnTdoDq" src="https://github.com/user-attachments/assets/f8c4d821-f31c-4fcd-ae73-f375a56c3a49" />
Preemptively, we need to give our `root@localhost` account privileges for the newly created database. 
Select the account under User accounts, navigate to the Database tab, select the new database, and select Go. Select the `Check all` box for Database-specific privileges, then hit Go.  
<img width="750" height="261" alt="8N2NR1i" src="https://github.com/user-attachments/assets/c01373bd-69e1-4ccd-9e64-11f04ee1991a" />


### Installing osTicket
Download the latest version of [osTicket](https://osticket.com/download).
Extract the folder. 
To integrate it with XAMPP, navigate to `C:\xampp\htdocs` and create a new folder, named `osticket`. 
Copy the `scripts` and `upload` folders from the extracted `osTicket` folder into our newly created folder. 
<br>
Once the folders are copied, we're now able to open our osTicket page using our web browser! In the url field, we can enter `localhost/osticket/upload` and are greeted with this page: 
<img width="1914" height="1003" alt="bkWxF7P" src="https://github.com/user-attachments/assets/a825adb5-79bc-46ea-8ba5-59ebd4bf60c5" />

And the following page informs us:  
<img width="824" height="501" alt="zvCUR8t" src="https://github.com/user-attachments/assets/b18e071d-fd84-42c9-b7be-b9ce74e95d6e" />

Let's troubleshoot. We simply need to ensure that the config file is present. Navigate to `C:\xampp\htdocs\osticket\upload\include\ost-sampleconfig.php` and rename 
`ost-sampleconfig.php` as `ost-config.php`. I chose to create a copy of the original for best practices of having a backup. 
<br>
After renaming and refreshing the page, we now see: 

<img width="1910" height="1000" alt="dtBne84" src="https://github.com/user-attachments/assets/99e330ab-cbdf-425c-b391-e3bcb995a2f4" />

We can now Install and configure osTicket. I create a mock Admin account and set credentials: 

<img width="831" height="806" alt="a76HE6g" src="https://github.com/user-attachments/assets/53bfaa23-8a66-4f9d-939f-dad07d036512" />

(troubleshooting note: the company emails conflicted, so I renamed one to `test@test1.com`. 
<br>
We also have to connect Database information. This is why we created the `osticket` database in the phpMyAdmin portal. Because it's locally hosted, our hostname remains `localhost` and not a set IP. 

<img width="827" height="370" alt="TzbMR0Q" src="https://github.com/user-attachments/assets/f22c2cda-ff1c-4f75-8305-5f08fe23721e" />
<br>
We can now complete our osTicket installation! 
<img width="831" height="653" alt="tWaG5kf" src="https://github.com/user-attachments/assets/936a709b-edce-482f-8dc6-9fcc32034842" />
<br>
osTicket recommends that we change the write permissions for our `ost-config.php` file for least privilege and best practice, so I accomplished it with PowerShell. 
`icacls` is a command that lets us modify the DACLs (Discretionary Access Control List) for files. I point it to our `ost-config.php` directory and run it. 
 
<img width="983" height="211" alt="ff4iIIT" src="https://github.com/user-attachments/assets/106ba0d8-e44f-4813-a9ad-baa958e8cb46" />


