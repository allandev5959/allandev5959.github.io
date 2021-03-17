---
layout: post
title: AD Attack Lab Part Three (An Introduction of BloodHound and PowerView)
---

![Alt](https://bohansec.com/assets/AD-Attack-3/leni-thalin-wtqo3pzvX5o-unsplash.jpg "Leni Thalin")

In part three of the AD attack lab series, we will learn how to use BloodHound and PowerView to enumerate the domain once you gain a foothold on the network. If you haven’t gotten the lab environment setup yet, go to [Part One](https://bohansec.com/2020/10/10/How-To-Set-Up-AD-Attack-Lab-Part-1/){:target="_blank"} and [Part Two](https://bohansec.com/2020/10/18/AD-Attack-Lab-Part-2/){:target="_blank"} to get the AD lab setup. 

### PowerView

Head over to one of your Windows 10 clients. Download the PowerView at [here](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerView){:target="_blank"}. Once the “powerview.ps1” is downloaded, open a terminal, execute “powershell -ep bypass” to enter the PowerShell bypassing execution policy mode, which allows executing any script we want. 

Note, I used the old version of the PowerView because some flags not work properly in the newer version of the PowerView.

![PowerView](https://bohansec.com/assets/AD-Attack-3/1.PNG "PowerView")

{% highlight js %}
powershell -ep bypass
{% endhighlight %}

Load the PowerView module into the current PowerShell session: 

{% highlight js %}
. .\PowerView.ps1
{% endhighlight %}

Now, let’s examine some basic flags and see how we can leverage them to enumerate the entire domain.

{% highlight js %}
Get-NetDomain
{% endhighlight %}

The “Get-NetDomain” will show the basic information about the domain which includes Forest name, Domain Controller, and the Domain Owner. 

![PowerView](https://bohansec.com/assets/AD-Attack-3/2.PNG "PowerView")

{% highlight js %}
Get-NetDomainController
{% endhighlight %}

The “Get-NetDomainController” shows the properties of the Domain Controller, which includes its IP address, OS version, Host Name, and other properties. 

![PowerView](https://bohansec.com/assets/AD-Attack-3/3.PNG "PowerView")

{% highlight js %}
Get-NetDomainController
{% endhighlight %}

The “Get-DomainPolicy” lists all the domain policies, from here, we can see some interesting property includes “SystemAccess”. You can see the minimal password length is only 7 which made the password cracking relevant easy.

![PowerView](https://bohansec.com/assets/AD-Attack-3/4.PNG "PowerView")

{% highlight js %}
(Get-DomainPolicy)."SystemAccess"
{% endhighlight %}

We can also narrow down and look for each property individually.

![PowerView](https://bohansec.com/assets/AD-Attack-3/5.PNG "PowerView")

{% highlight js %}
Get-NetUser 
{% endhighlight %}

The “Get-NetUser” command will list all the users on the domain with their properties. One interesting user we can find here is the “SQL Service” with its description field. The password is in the description field with plaintext. 

![PowerView](https://bohansec.com/assets/AD-Attack-3/6.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/7.PNG "PowerView")

{% highlight js %}
Get-NetUser | select cn
Get-NetUser | select samaccountname
Get-NetUser | select description
{% endhighlight %}

We can use “select” to grab the specific property that we are interested in. The “select” is similar to “grep” in Linux.

![PowerView](https://bohansec.com/assets/AD-Attack-3/8.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/9.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/10.PNG "PowerView")

{% highlight js %}
Get-UserProperty
Get-UserProperty -Properties pwdlastset
Get-UserProperty -Properties badpwdcount
{% endhighlight %}

The “Get-UserProperty” command can list all the user’s properties. We can use the “Properties” flag to show one property at a time. 

![PowerView](https://bohansec.com/assets/AD-Attack-3/11.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/12.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/13.PNG "PowerView")

{% highlight js %}
Get-NetComputer
Get-NetComputer -Fulldata | select operatingsystem
{% endhighlight %}

Show all the workstations on the domain with their properties.

![PowerView](https://bohansec.com/assets/AD-Attack-3/14.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/15.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/16.PNG "PowerView")

{% highlight js %}
Get-NetGroup -GroupName "Domain Admins"
Get-NetGroupMember -GroupName "Domain Admins"
{% endhighlight %}

Show the Group names and Group members.

![PowerView](https://bohansec.com/assets/AD-Attack-3/17.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/18.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/19.PNG "PowerView")

{% highlight js %}
Invoke-ShareFinder
{% endhighlight %}

Find all the shares on the domain. We can see the “Documents” and “Share” folder we created early are presented here. 

![PowerView](https://bohansec.com/assets/AD-Attack-3/20.PNG "PowerView")

{% highlight js %}
Get-NetGPO
Get-NetGPO | select displayname, whenchanged
{% endhighlight %}

Show the domain policy and use “select” to narrow down the property we are interested in. We can see the “Disable Windows Defender” Policy we set early presented here.

![PowerView](https://bohansec.com/assets/AD-Attack-3/21.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/22.PNG "PowerView")
![PowerView](https://bohansec.com/assets/AD-Attack-3/23.PNG "PowerView")

You can find more useful commands for PowerView [here](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993){:target="_blank"}.

### BloodHound

BloodHound can be used for enumerating the entire domain. Based on the domain data, it can help you to determine what’s the shortest path to get the domain admin. It is a fantastic tool that aids you to see a bigger picture of the entire domain. 

The BloodHound installation process is pretty straight forward.

Update your Kali APT repo: 

{% highlight js %}
sudo apt-get update
{% endhighlight %}

Install the BloodHound via the “apt-get”:

{% highlight js %}
sudo apt-get install bloodhound
{% endhighlight %}

![BloodHound](https://bohansec.com/assets/AD-Attack-3/24.PNG "BloodHound")


Setup the Neo4j once the installation process finished: 

{% highlight js %}
sudo neo4j console
{% endhighlight %}

![BloodHound](https://bohansec.com/assets/AD-Attack-3/25.PNG "BloodHound")
![BloodHound](https://bohansec.com/assets/AD-Attack-3/26.PNG "BloodHound")
![BloodHound](https://bohansec.com/assets/AD-Attack-3/27.PNG "BloodHound")

Now run the BloodHound: 

{% highlight js %}
sudo bloodhound
{% endhighlight %}

![BloodHound](https://bohansec.com/assets/AD-Attack-3/28.PNG "BloodHound")
![BloodHound](https://bohansec.com/assets/AD-Attack-3/29.PNG "BloodHound")

Now head over to one of your Windows 10 clients, download the “SharpHound Data Ingestor” [here](https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.ps1){:target="_blank"}. Import the Powershell module then generate the zip file contains the data in the domain. 

{% highlight js %}
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -Domain kudos.local -ZipFilename file.zip
{% endhighlight %}

![BloodHound](https://bohansec.com/assets/AD-Attack-3/30.PNG "BloodHound")

Move the zip file to the kali, and upload it to the BloodHound application.

![BloodHound](https://bohansec.com/assets/AD-Attack-3/31.PNG "BloodHound")

We can see the data have already uploaded successfully. 

![BloodHound](https://bohansec.com/assets/AD-Attack-3/32.PNG "BloodHound")

Examine through the various queries, we see it can show us the shortest path to the domain admins, the shortest path to the high-value targets, the shortest path to the Kerberoastable Users, etc. I highly recommend you to check out all the queries listed here to get a feel of what they could do for you :)

![BloodHound](https://bohansec.com/assets/AD-Attack-3/33.PNG "BloodHound")

![BloodHound](https://bohansec.com/assets/AD-Attack-3/34.PNG "BloodHound")

![BloodHound](https://bohansec.com/assets/AD-Attack-3/35.PNG "BloodHound")

![BloodHound](https://bohansec.com/assets/AD-Attack-3/36.PNG "BloodHound")

![BloodHound](https://bohansec.com/assets/AD-Attack-3/37.PNG "BloodHound")

![BloodHound](https://bohansec.com/assets/AD-Attack-3/38.PNG "BloodHound")

In this post, we learned how to use PowerView and BloodHound to do some basic enumeration of the domain. In the following posts, I will continue to discuss other ways to enumerate and attack the AD environment. Thank you for reading :)