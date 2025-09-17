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

osTicket recommends that we change the write permissions for our `ost-config.php` file for least privilege and best practice, so I accomplished it with PowerShell. 
`icacls` is a command that lets us modify the DACLs (Discretionary Access Control List) for files. I point it to our `ost-config.php` directory and run it. 
(Another simple alternative is right-clicking on the file, selecting Properties, and setting it to Read-Only.) 
 
<img width="983" height="211" alt="ff4iIIT" src="https://github.com/user-attachments/assets/106ba0d8-e44f-4813-a9ad-baa958e8cb46" />

FOr additional best practices, I also delete our `/setup` directory, recommended by osTicket. 

### Exploring osTicket- Managing/Creating Users, Ticket Management
We've now installed osTicket! The account that we created was an Admin account, so we can log in via the Staff Control Panel url, `http://localhost/osticket/upload/scp`.
<img width="1912" height="1002" alt="e9946034-cddf-4f92-86b8-cefa6bf7c22f" src="https://github.com/user-attachments/assets/ce805bf5-c2b5-4b65-b62e-8ef21fec5c53" />

<img width="1897" height="999" alt="KMGfcPW" src="https://github.com/user-attachments/assets/def414ad-5798-4829-a242-6e2423c67399" />

As Admin in our Admin Panel view, our account has access to the whole scope of our osTicket settings, and since our account is also an agent, we have an Agent Panel view.
We can execute tasks like working available tickets, managing users, password lockouts, setting SLAs, etc. 

For example, I created support Agents with varying levels of permissions- from full Admin access to limited viewing of only assigned tickets. I was also able to create
Departments and set them to specific Departments: 

<img width="1053" height="453" alt="SZm6QXe" src="https://github.com/user-attachments/assets/db5324b5-b1d6-4592-9d76-55af367b9a12" />


For example, using our Admin account, I created mock users, and submitted a ticket through their user account. 
<img width="960" height="506" alt="FiMisR6" src="https://github.com/user-attachments/assets/3a4d6096-b0cf-4c91-a45a-72eb296c1f58" />

(I found that password resets can be automated through osTicket's provided templates! Allowing users to reset passwords themselves for osTicket) 

I was also able to access our osTicket system through other clients (ex., my Host PC) and submit tickets there, logging in as a created user. 
I created sample tickets, assigning them to different Support Agents and setting appropriate priorities, and replying to users. 

 <img width="990" height="737" alt="EgjM9ob" src="https://github.com/user-attachments/assets/ce2109e9-68ed-4393-be91-ff96fd978cc1" />

An example ticket that was opened, troubleshot, verified the fix, and subsequently closed: 

<img width="771" height="830" alt="ZWxR4Ku" src="https://github.com/user-attachments/assets/da4e4255-077d-4f2e-bf4b-5ed5b3f84307" />

### Testing Firewall rules
To test our basic firewall inbound rules, I checked if we were able to access osTicket, running in our Azure VM,  from the outside- via HTTP that we opened on port 80. 

<img width="1573" height="1038" alt="RRCUynR" src="https://github.com/user-attachments/assets/4078e2b3-854b-4252-8ac6-24739eb4769d" />

And we were. All tickets were created from an outside IP address, connecting to our osTicket page via the Azure VM's public IP.
I also verified the RDP Access rule by successfully RDP'ing to the VM from my IP address, ensuring traffic is restricted to it. 

## Lessons Learned

In conclusion, I was able to deploy a self-hosted osTicket environment using XAMPP (Apache/MySQL), running on a Windows Server VM via Microsoft Azure. 
This project deepened my understanding of a real-world ticketing system setup, web application hosting, and deploying (and securing) a helpdesk system in the cloud. It gave me a jumping off point for learning
Azure's cloud environment- I learned how to provision a cloud-based VM, manage a resource group and configure its firewall rules via Network Security Groups (NSGs), 
letting me implement least privilege by restricting open ports to only the essentials (HTTP(s), RDP), and configuring traffic (allowed IP addresses for RDP). 
I also learned- at a high level overview- configuring web servers and databases, and troubleshoot dependency issues, permissions, and networking network hardening. 
By far, the most valuable addition to my knowledgebase was exploring the osTicket environment. I managed Agent accounts, created Users and gained hands-on experience 
in ticket creation and management, User communication, assigning tickets and setting priorities, SLAs, with real-world mock ticket examples. Although there are many different 
ticketing systems, I understand that these aspects are universal to ALL helpdesk ticketing systems. Being able to troubleshoot and play with the environment granted me 
invaluable insight that I can translate into other systems like Jira or ServiceNow. 


