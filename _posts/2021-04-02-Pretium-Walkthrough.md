---
layout: post
title: BTLO Pretium Walkthrough
---
![Alt](https://bohansec.com/assets/Pretium/coverpage.png "Security Blue Team")

The writeups will be a series to document how I solved each scenario on BTLO (Blue Team Labs Online), hope you will enjoy it :)

## Pretium Video Walkthough 

<div class="youtube-wrapper">
    <iframe 
            src="https://www.youtube.com/embed/qJATvmgXNhA"
            frameborder="0"
            allow="autoplay; encrypted-media"
            allowfullscreen></iframe>
</div>



## Scenario
***The Security Operations Center at Defense Superior are monitoring a customer’s email gateway and network traffic (Crimeson LLC). One of the SOC team identified some anomalous traffic from Josh Morrison’s workstation, who works as a Junior Financial Controller. When contacted Josh mentioned he received an email from an internal colleague asking him to download an invoice via a hyperlink and review it. The email read:***

***There was a rate adjustment for one or more invoices you previously sent to one of our customers. The adjusted invoices can be downloaded via this [link] for your review and payment processing. If you have any questions about the adjustments, please contact me.***

***Thank you.***

***Jacob Tomlinson, Senior Financial Controller, Crimeson LLC.***

***The SOC team immediately pulled the email and confirmed it included a link to a malicious executable file. The Security Incident Response Team (SIRT) was activated and you have been assigned to lead the way and help the SOC uncover what happened.***

***You have NetWitness and Wireshark in your toolkit to help find out what happened during this incident.***

## Tools
- Wireshark
- Netwitness
- Cyberchief
- Tshark

## Difficulty
- Medium

## Reading Material
[Link1](https://isc.sans.edu/forums/diary/An+Introduction+to+RSA+Netwitness+Investigator/18199/){:target="_blank"}
[Link2](https://resources.infosecinstitute.com/topic/pcap-analysis-basics-with-wireshark/){:target="_blank"}
[Link3](https://isc.sans.edu/forums/diary/Packet+Tricks+with+xxd/10306/]){:target="_blank"}

## Scenario Questions

### What is the full filename of the initial payload file?

Open the LAB.pcap file from Wireshark. 

![screenshot](https://bohansec.com/assets/Pretium/1.PNG "screenshot")

Since we know Josh downloaded a maclious "invoice" file, we will filter through HTTP traffic to see if we can find anything from there. We will also add the HTTP filter as the label for easier access later.

![screenshot](https://bohansec.com/assets/Pretium/2.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Pretium/3.PNG "screenshot")

We find the traffic "4502" has a name "INVOICE" with it, so we can conclude that "INVOICE_2021937.pdf.bat" is the inital payload Josh downloaded from his internal colleague. This could also indicate that the attacker comprimsed his colleague's computer first and got a foothold, then trying to pivot from there.

![screenshot](https://bohansec.com/assets/Pretium/4.PNG "screenshot")

Answer:
INVOICE_2021937.pdf.bat

### What is the name of the module used to serve the malicious payload?

Follow the TCP stream on traffic 4502, we find there is a line "Server: SimpleHTTP/0.6 Python/3.8.5" in the http header. If you ever have done any red team training, you would know that attacker served the maclious payload on a Python Web Server. 

![screenshot](https://bohansec.com/assets/Pretium/5.PNG "screenshot")

Python Web Server Command:

{% highlight js %}
python -m SimpleHTTPServer [PORT]
{% endhighlight %}

Answer:
SimpleHTTPServer

### Analysing the traffic, what is the attacker's IP address?

From the previous questions, we already know Josh at (192.168.1.8) downloaded a maclious payload from his colleague's comprimised machine at (192.168.1.9).

![screenshot](https://bohansec.com/assets/Pretium/2.PNG "screenshot")

Answer:
192.168.1.9

### Now that you know the payload name and the module used to deliver the malicious files, what is the URL that was embedded in the malicious email?

By looking at the traffic 4502, and expanding the "Hyper Text Transfer Protocol" tab, we can find the full request URL is 

{% highlight js %}
"http://192.168.1.9:443/INVOICE_2021937.pdf.bat"
{% endhighlight %}

![screenshot](https://bohansec.com/assets/Pretium/6.PNG "screenshot")

Answer:
{% highlight js %}
"http://192.168.1.9:443/INVOICE_2021937.pdf.bat"
{% endhighlight %}

### Find the PowerShell launcher string (you don’t need to include the base64 encoded script)

Back to the TCP stream we followed early, we can see the encoded maclious powershell script started from a batch file.

![screenshot](https://bohansec.com/assets/Pretium/7.PNG "screenshot")

Answer:
powershell -noP -sta -w 1 -enc

### What is the default user agent being used for communications?

Follow one of the TCP Stream after the traffic "4502", we find the communication between the client and server looks like a c2 server. Follow the "GET /news.php" TCP stream, we find that the user agent is "Mozilla/5.0".

![screenshot](https://bohansec.com/assets/Pretium/8.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Pretium/9.PNG "screenshot")

Answer:
Mozilla/5.0

### You are seeing a lot of HTTP traffic. What is the name of a process where malware communicates with a central server asking for instructions at set time intervals

The answer is "beaconing" but if you do not know the answer, you can alwasy google it.

![screenshot](https://bohansec.com/assets/Pretium/10.PNG "screenshot")

Answer:
beaconing

### What is the URI containing ‘login’ that the victim machine is communicating to?

Scrolling down the traffic from traffic "4502", we find the traffic contains "POST /login/process.php".

![screenshot](https://bohansec.com/assets/Pretium/11.PNG "screenshot")

Answer:
http://192.168.1.9/login/process.php

### What is the name of the popular post-exploitation framework used for command-and-control communication?

If you have ever worked with this C2 framework, you should be familiar with the traffic it produces. But if you have not worked with it before, you can search on google with the HTTP request we have observed from Wireshark. We find the c2 framework the attacker used is Empire.

![screenshot](https://bohansec.com/assets/Pretium/12.PNG "screenshot")

Answer:
Empire

### It is believed that data is being exfiltrated. Investigate and provide the decoded password

The last two questions require to read the [link](https://isc.sans.edu/forums/diary/Packet+Tricks+with+xxd/10306/){:target="_blank"} provided from BTLO. The attacker used covert channel to exfiltrated the credential through the ICMP packets from the network. 

We will use tshark to extract the exfiltrated data first. 

{% highlight js %}
tshark.lnk -r C:\Users\BTLOTest\Desktop\Investigation\LAB.pcap -T fields -e data > result.txt
{% endhighlight %}

![screenshot](https://bohansec.com/assets/Pretium/13.PNG "screenshot")

Open the result.txt file we just created, we find some data presented in hexadecimal. if you are familar with the scripting, you could automate the following steps and save a ton of time. But we will do it manually fow now.

![screenshot](https://bohansec.com/assets/Pretium/14.PNG "screenshot")

Go to [CyberChief](https://gchq.github.io/CyberChef/){:target="_blank"}, copy and paste the strings to the cyberchief, and convert the hex to ascii. we have 

{% highlight js %}
UUAABBhhAAHHMMAAccwwBB33AAGG88AAc
{% endhighlight %}

as the converted ascii.

![screenshot](https://bohansec.com/assets/Pretium/15.PNG "screenshot")

Let's try to decode it with Base64, the result does not make any sense. If you look closely, the character seems repeated itself twice. So, let's try to remove the extra character to see what we can find. Now we have "UABhAHMAcwB3AG8Ac". We see in Cyberchief the decoded text make sense now.

![screenshot](https://bohansec.com/assets/Pretium/16.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Pretium/17.PNG "screenshot")

Repeat the above steps for the next two sections of the hexadecimal. We got our result back as a full sentence. 

{% highlight js %}
P.a.s.s.w.o.r.d. .f.o.r. .m.y. .$.s.e.c.-.a.c.c.o.u.n.t.:. .Y.0.u.t.h.i.n.k.y.0.u.c.A.n.c.4.t.c.h.m.3.$.$.
{% endhighlight %}

![screenshot](https://bohansec.com/assets/Pretium/18.PNG "screenshot")

The full URL if you do not want to type: 
{% highlight js %}
https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)&input=VUFCaEFITUFjd0IzQUc4QWMgZ0JrQUNBQVpnQnZBSElBSUFCdEFIa0FJQUFrQUhNQVpRQmpBQzBBWVFCakEgR01BYndCMUFHNEFkQUE2QUNBQVdRQXdBSFVBZEFCb0FHa0FiZ0JyQUhrQU1BQjFBR01BUVFCdUFHTUFOQUIwQUdNQWFBQnRBRE1BSkFBa0FBPT0
{% endhighlight %}



Answer:
Y0uthinky0ucAnc4tchm3$$

### What is the account’s username?

Answer:
$sec-account


That's all for today, hope you enjoyed the walkthrough and learned something new! I will keep posting more BTLO writeups as I make progress on the platform. Thank you for stoping by :) 




