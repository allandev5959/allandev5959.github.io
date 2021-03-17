---
layout: post
title: How To Set Up an AD Attack Lab Part One
---
![Alt](https://bohansec.com/assets/AD-Attack-1/imgix-klWUhr-wPJ8-unsplash.jpg "imgix")

## Preparation

What you need for this lab:
* At least 16GB RAM, if you do not have, you might experience some performance issue
* VMware Workstation (Player should be fine but I used Pro 15.5)
* Windows Server 2019 (Standard should be fine but I used Datacenter)
* Windows 10 (2x) (Pro should be fine but I used Education)

Note, in order for the GPO(Group Policy Object) to work on disable the windows defender, I highly suggest you use windows 10 build 1809 or before because the GPO can’t disable windows defender on the newer version. I realized this issue while I was doing this lab and have to switch from build 1909 to build 1809. You can see more information about this [here](https://www.bleepingcomputer.com/news/microsoft/malware-can-no-longer-disable-microsoft-defender-via-the-registry/){:target="_blank"}.

I downloaded my ISO images from Azure Education, but if you do not have one, you can use the evaluation version from the following addresses:

[Windows Server 2019 Download](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019){:target="_blank"}

[Windows 10 Download](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise){:target="_blank"}

Note, in this lab, I will use NAT DHCP for my environment, but you can choose the network type based on your environment. 

### Install Windows 2019

1. In your VMWare, choose “File -> New Virtual Machine”.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/1.PNG "Insall Server 2019")

2. Choose from browsing where your ISO file downloaded.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/2.PNG "Insall Server 2019")

3. Keep the default setting for the next two pages.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/3.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/4.PNG "Insall Server 2019")

4. Uncheck “Power on this virtual machine after creation”, then hit the finish. Now, you can start the machine and the installation process will automatically start for you.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/5.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/6.PNG "Insall Server 2019")

5. Once the installation process finished, login, and change the server name to whatever you want. Here, I named it “DC-2019”. Now take a snapshot and restart the computer.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/7.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/8.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/9.PNG "Insall Server 2019")

6. Once login, go to “Manage”, then go to “Add Roles and Features”, accept the default settings till the “Server Roles” page. Now, click “Active Directory Domain Services” and hit “Add Feature”.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/10.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/11.PNG "Insall Server 2019")

7. Accept the default setting in the remaining pages and wait till the installation process to finish. Now, restart the server.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/12.PNG "Insall Server 2019")

8. Once the server restarted, we will go to the “Post Server Configuration”.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/13.PNG "Insall Server 2019")

9. Add a new forest, and choose a domain name you prefer. Here, I chose “KUDOS.local” as my domain name.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/14.PNG "Insall Server 2019")

10. Setup a password for your DSRM, accept all the default configuration in the rest of the pages, and click “install” on the last page.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/15.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/16.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/17.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/18.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/19.PNG "Insall Server 2019")

11. Once the server restarted again, you will see the server ask you to log in as the domain admin, which means our Active Directory successfully installed on this server.

    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/20.PNG "Insall Server 2019")
    ![Insall Server 2019](https://bohansec.com/assets/AD-Attack-1/21.PNG "Insall Server 2019")


### Install Windows 10

1. Choose where your windows 10 ISO located in the browser.

    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/22.PNG "Insall Windows 10")

2. Accept the default settings in the rest pages and install the image. Note, if you use the evaluation version, you do not need a product key.  

    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/23.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/24.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/25.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/26.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/27.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/28.PNG "Insall Windows 10")

3. Now, depends on whether you used a VMWare player or pro. If you used the Player, you have to repeat the steps again to install a second windows 10. If you have pro, you can use the clone function to clone an identical installation from the current one. Since I have the Pro, I used the clone method.

    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/29.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/30.PNG "Insall Windows 10")

4. Change the windows 10 system names and restart the machines.

    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/31.PNG "Insall Windows 10")
    ![Insall Windiws 10](https://bohansec.com/assets/AD-Attack-1/32.PNG "Insall Windows 10")

### Configuring the Active Directory on Server 2019

1. Login Server 2019, opens the “Active Directory Users and Computers”, creates a new OU object named “Groups” under the current domain.

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/33.PNG "Configure Server 2019")

2. Place all the Groups users to the created “Groups” object.

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/34.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/35.PNG "Configure Server 2019")

3. Now, I created the following users as domain users. They are created for the purpose of our later attacking and follows a poor practice. DO NOT DO this in the production environment!

    * first user:bwallis
    * second user:calcock
    * second admin:csherman
    * SQL service running as the administrative privilege:SQLService

    Create the user bwallis:

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/36.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/37.PNG "Configure Server 2019")

    Create the second admin csherman by coping from the Administrator:

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/38.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/39.PNG "Configure Server 2019")

    Create the user calcock by coping the existing user bwallis:

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/40.PNG "Configure Server 2019")

    Create the SQL service by coping from Administrator user:

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/41.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/42.PNG "Configure Server 2019")

    Now, we can see we have the following users being created: 

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/43.PNG "Configure Server 2019")

    I also left the password in the description field of the SQL service user for the later attack, this is a very BAD practice and should be avoided 100% in a production environment:

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/44.PNG "Configure Server 2019")

4. Now, head to your “Server Manager->File and Storage Service”, choose “new shares” under tasks because we want to create a file share on the Active Directory.  

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/45.PNG "Configure Server 2019")

5. Choose the default “SMB Share - Quick”.

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/46.PNG "Configure Server 2019")

6. Give a name to your share, I named it “Documents”. Then, accept the default settings for the rest pages and create the share. 

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/47.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/48.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/49.PNG "Configure Server 2019")

7. Now, open a “CMD” prompt as “Administrator”. We want to set the “Service Principal Name” for our newly created “SQL Service”. Adjust the commands based on your server and domain name.

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/51.PNG "Configure Server 2019")

8. In this lab, we will examine the various AD attack against the misconfiguration and the AD default setting. So we will disable the windows defender from Domain GPO for now. Note, the GPO only could work on the windows period to 1809. Newer 1903 would not let the GPO disable the windows defender. Go to “Server Manager -> Tools”, open “Group Policy Management”. Then select the domain, go to “Create a GPO in this domain” option. Another way to do it is directly open the “Group Policy Management” directly as Administrator from the Start menu. We will name it “Disable Windows Defender”. 

     ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/52.PNG "Configure Server 2019")
     ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/53.PNG "Configure Server 2019")
     ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/54.PNG "Configure Server 2019")

9. Right-click the newly created policy. Go to “Edit”. Under “Policies -> Administrative Templates -> Windows Components -> Windows Defender antivirus”, we will enable “Turn off windows defender antivirus” GPO. Select “Enable” Then click “Apply” and “OK”.

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/55.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/56.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/57.PNG "Configure Server 2019")
    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/58.PNG "Configure Server 2019")

10. Now, we can see the GPO is enabled. This will ensure that when any clients join this domain, the windows defender antivirus will be disabled.

    ![Configure Server 2019](https://bohansec.com/assets/AD-Attack-1/59.PNG "Configure Server 2019")

### Configuring the Windows 10, and join them to the domain

1. Login the windows 10 we previously created, then create a folder named “Share” on the C drive, enable the share on it. 

    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/60.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/61.PNG "Configure Windows 10")

2. Add Server 2019’s IPv4 address to the Windows 10 DNS setting. 

    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/62.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/63.PNG "Configure Windows 10")

3. Go to “Access work or school”, click “Connect”. Select “Join this client to a local Active Directory Domain”. Type in the domain name, and the domain admin user name and password. Now, windows 10 should enroll in the domain. 

    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/64.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/65.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/66.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/67.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/68.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/69.PNG "Configure Windows 10")

4. Log in to one of the windows 10 machines. Go to “Computer Management”. set one of the domain users to the local admin group. Also, set it as the local administrator by creating a new user.

    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/70.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/71.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/72.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/73.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/77.PNG "Configure Windows 10")

5. Repeat the same step on the other windows 10 machine, except this time, the domain user belongs to this machine does not have access to another windows 10 machine. Note, you will see "bwallis" domain user appeared in the second machine as well because we want this user to have the local admin to the second windows 10 machine.

    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/74.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/79.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/75.PNG "Configure Windows 10")

6. Last, enable the network to discover and verify the two machines have joined the domain on the Server 2019. Note, you also need to start the “Function Discover Resource Publication” in “Services” in order for the machines to be discovered by others. See [here](https://www.ghacks.net/2018/04/17/fix-pcs-no-longer-recognized-in-network-after-windows-10-version-1803-upgrade/){:target="_blank"} for more info.

    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/76.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/78.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/80.PNG "Configure Windows 10")
    ![Configure Windows 10](https://bohansec.com/assets/AD-Attack-1/81.PNG "Configure Windows 10")

In this post, we learned how to set up the AD environment for our AD attack lab. In the following posts, I will discuss various ways to compromise the AD. Thank you for reading :)

Commands for creating SPN:
    {% highlight js %}
    setspn -a DC-2019/SQLService.KUDOS.local:62111 KUDOS\SQLService
    setspn -T KUDOS.local -Q */*
    {% endhighlight %}

