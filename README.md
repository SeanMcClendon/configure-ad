<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines. As we create 2 Virtual Machines, one will be utilized as a Domain Controller (DC). Inside said DC we will create showcase functionality and understand how IT departments allow users to sign into their accounts from any given computer (the other VM) that is connected to the Domain.<br />




<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Install Active Directory
- Join Client-1 to your domain (mydomain.com)
- Create accounts
- Reset Accounts/Password

<h2>Deployment and Configuration Steps</h2>

<h3>Setup Resources in Azure</h3>
1. Create Virtual Machine in https://portal.azure.com/

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/9a03e036-7fa8-4db5-949b-5ac9a188e574)

2. Create the Domain Controller VM (Windows Server 2022) named “DC-1”

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/8b7c3549-4953-4940-9cee-6ab660064db4)

3. For size select a vm with at least 2 cpu's. Pick your preferrence of Username and Password. (check Licensing box if it pops up)
   
![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/aabb4f2d-98d7-43dd-bc7f-25ee179e784e)

4. Click "Review and Create", after validation passes click "Create"
5. Create another VM, Choose same resource group (AD-lab), Name VM "Client-1" choose same time zone for region, and select "windows 10" for Image and same size as previous. choose same Username and Password. click allow for RDP port (3389)
6. click next for disks, then next for Network. Ensure client-1 is on the same vnet as the previous vm. Click Review and create, then create. 

   ![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/2744d808-e271-4709-a383-fdcca5ef9e06)

7. Set the Domain Controller's NIC Private IP address to static: Virtual Machines-> DC-1->Networking->CLick on Network Interface (NIC)->IP configurations (below settings)->click on the IP config1 then change to Static then hit save. 

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/2c8ee27c-633b-4c8d-86f4-4e75bfc73195)

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/c8103d8d-572e-4b7a-a123-48c2992cdab7)

 
 8. Ensure both VM's have the same V-net
  <h3>Ensure Connectivity between the client and Domain Controller</h3>
  
9. Login to Client-1 with Remote Desktop using Public IP Address and ping DC-1’s Private IP address with ping -t <ip address> (perpetual ping) notice that it timeed out, this is to show you that DC-1 firewall is blocking ICMPv4 traffic.

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/22aed54e-d98a-480c-affa-59d496dde23f)
 

10. Login to the Domain Controller go into the Firewall

  ![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/259e9c45-4b2b-4c32-82e1-ce2f2ab1de27)

11. Click "inbound rules, then sort by clicking on Protocol.

    ![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/34db337b-85d8-4a01-8969-3eb5d81031e9)

    ![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/7649fdeb-00bb-4f1e-b216-f9d1791a4b14)


12. Right click and Enable both Echo request ICMPv4 on the local windows Firewall.

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/b4e82f23-49f6-4075-9fc8-0b014643a1a5)


13. Check back at Client-1 to see the ping succeed

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/5d34aa98-4037-4377-8a22-a0e1dd067dc6)


<h3>Install Active Directory</h3>

14. Switch back to DC-1, at Server Manager page click "add rules and Features"

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/55fee380-7125-4bba-ba99-492905c001be)

15. click next/next/next the click "Active Directory Domain Services and features"
<p>


![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/2cd9d553-60dd-47c2-8dfd-5c467c3617a3)

16. click next/next/next/Install
17. After Installation succeeded, click close then look for the yellow triangle in top right and click Promote to Domain Controller

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/8f2ba4eb-a042-4d14-9ccb-83bf6e99f561)

18. Add New forest, type in "mydomain.com" (I chose generic, truly your choice)

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/3a8e0798-37de-49d8-abd7-cd9d1e60c43f)

19. click Next, create Password (Password1), wait for your domain name to populate, then click next all the way through and then install. DC_! will restart on its own.
20.  log back into DC-1 as user: mydomain.com\labuser(original username as before)
21.  In start menu search bar type Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES”

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/0123d4b7-e6e4-4d5e-b7a9-9a41d49da5b3)

22. Create a new employee named “Jane Doe” (same password) with the username of “jane_admin”, uncheck box for new password. 

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/9d956128-3b8b-4720-87b2-77b05b1e4f40)

23. Add jane_admin to the “Domain Admins” Security Group by right clicking on her name, clicking properties, select member of, type in Domain then check name. click Domain Admin. Apply then ok.


<h3>Join Client-1 to your domain (mydomain.com)</h3>

24. in Azure Portal, Set Client-1’s DNS settings to the DC’s Private IP address: DC-1->Networking->copy Private Ip address->clcik on client-1 NIC highlighted below-> click on DNS server on the right-> then click custom->paste DC-1 Private IP address then click save

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/077526f7-6c28-409d-8ede-51b9a51a44e2)

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/20ac4bd1-3927-462e-94d5-9530be213d6c)

25. Restart Client-1 Vitrual Machine, itll automatically restart your Remote Desktop Session. Log back in using client-1 public IP address as same user as before. 

26. Right click Start icon->System->Rename this PC advanced->Change-> clcik Domain-> type in domain.com-> then click ok

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/6684994a-b17c-4e04-adee-216b083e195a)

<h3>Setup Remote Desktop for non-administrative users on Client-1</h3>

27. Sign in under jane_admin domain account

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/bcade2f1-e8a2-466f-af59-7a207c0ba18e)

28. Follow prompts, Client-1 will restart, Login as jane_admin
29. Right click start icon-> Click Remote Desktop-> click "select users thatt can remotely acceess this pc"
30. Click Add-> type "domain users"-> click check names->ok->ok

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/b952fa3d-53b7-479a-a019-34c6561a6249)

<h3>Create a bunch of additional users and attempt to log into client-1 with one of the users</h3>

31. Open Powershell ISE as an Admin
32. Create a new File and paste the contents of the script into it (https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)
![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/8d115da4-54bc-427b-b391-e49890584711)

33. Run the script and observe the accounts being created (green triangle)

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/cba8772c-01a8-469c-9f15-0fd8ee957bf4)

34. Go to Active Directory Users and Computers, right click on Employees and refresh and observe new accounts/individuals being created.

 ![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/5a833be9-75db-4d14-bdbe-d91d5bce8b0c)

 35. Logout of Client-1, Remote Desktop back in, but as a newly created user of your choice that you can obtain from DC-1

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/4b5e647e-105f-4333-be9e-85daad417b78)

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/4a45a3ce-f99b-41bf-922d-6a0c98962c4f)

<h3>Showcase how to unlock accounts & reset a users password</h3>

36. Remote back into DC-1 Desktop.
37. Choose random newly made user from the active Directory list
38. Double click on name->go to Account->copy display name

    ![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/fe12f9f5-0503-41ad-ba5b-6ad2f51ea791)

39. log out of Client-1, Log back in with newly selected user with INCORRECT PASSWORD 10X. This is to lock account.
40. Remote back into DC-1
41. Double click on name->go to Account->check box to unlock account

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/ee1a2e0f-18e2-4cff-b5a7-f2ebf847425c)

42. To reset password, right click on Employees'/user name the click "reset password"

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/77c961fd-b704-40df-9e3b-5faa84078bb8)

![image](https://github.com/SeanMcClendon/configure-ad/assets/142221948/6317fab2-0d3c-4b18-8f86-33a01857293d)

 <h1>CONGRATULATIONS, YOU DID IT</h1>
</p>
<p>

</p>
<br />
