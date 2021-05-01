---
layout: post
title: BTLO PEAK Walkthrough
---
![Alt](https://bohansec.com/assets/PEAK/cover.png "Security Blue Team")

The writeups will be a series to document how I solved each scenario on BTLO (Blue Team Labs Online), hope you will enjoy it :)

## Pretium Video Walkthough 

<div class="youtube-wrapper">
    <iframe 
            src="https://www.youtube.com/embed/vGdN4RW6DWY"
            frameborder="0"
            allow="autoplay; encrypted-media"
            allowfullscreen></iframe>
</div>


## Scenario

***Dwight works as a web developer at Mountain Top Solutions, Chicago. He reports unusual activity originating from the private network 10.x.x.x in the logs on the application development server. Dwight also added that the server should only be accessed directly from the console or from his laptop via ssh which is in the network 192.168.1.0/24. Can you investigate this anomaly?***

***You are provided with the following logs, already ingested into an ELK deployment:***
***1. apache2 access and error logs***
***2. auditd logs ( Auditd rules configured with https://github.com/bfuzzy/auditd-attack/blob/master/auditd-attack.rules)***
***3. auth.log***
***4. syslog***

## Tools
ELK

## Difficulty
- Medium 

## Reading Material
N/A

## Scenario Questions

### What is the hostname of the infected server?
Adjust the time range as shown in the image below, we will use the log source from Auth. The infected server is located in the 10.x.x.x range, we can see the hostname via the log entry.

![screenshot](https://bohansec.com/assets/PEAK/1.PNG "screenshot")
![screenshot](https://bohansec.com/assets/PEAK/2.PNG "screenshot")

Answer:

APPSERV-Chicago

### The attacker got into the server via what service/port?
Filter the message from the Auth log, we know the attacker is SSH bruteforcing the server. The SSH Service is running on port 44322 in this case.

![screenshot](https://bohansec.com/assets/PEAK/3.PNG "screenshot")

Answer:

SSH, 44322

### What is the tool to crack the password that he possibly used?
Lookup the log source from Apache2, filter the user agent, we see the attacker used Hydra for brute-forcing. 

![screenshot](https://bohansec.com/assets/PEAK/4.PNG "screenshot")

Answer:

Hydra

### What is the first command executed by the attacker? 
Set the display rows to 10000, then filter the log source auditd. Look carefully in the auditd log, sort the time from the earliest, we find the attacker execute Linux command "ls".

![screenshot](https://bohansec.com/assets/PEAK/5.PNG "screenshot")
![screenshot](https://bohansec.com/assets/PEAK/6.PNG "screenshot")

Answer:

ls

### What is the first domain the attacker connects to from the server, and what is the name of the file he downloads?
Searching through the auditd log source carefully. Filter the type as "EXECVE" and time from the earliest. 

![screenshot](https://bohansec.com/assets/PEAK/7.PNG "screenshot")

Answer:

raw.githubusercontent.com, linpeas.sh

### Attacker identifies version of a binary and tries to exploit it. What is the utility and what is the vulnerability he attempts to exploit? 
Filter the type as "EXECVE", look through the auditd log carefully, we find the attacker downloaded the 49521 exploits from exploit db, which is one of the Sudo vulnerabilities discovered early this year. 

![screenshot](https://bohansec.com/assets/PEAK/8.PNG "screenshot")
![screenshot](https://bohansec.com/assets/PEAK/9.PNG "screenshot")

Answer:

sudo, CVE-2021-3156

### The Exploit didn’t work. Attacker again downloads a script from a remote server to perform a different action. What is the domain name of the remote server and what is the file they downloaded?
Just above the Sudo command, we find the attacker downloaded the "upload_btlo.sh" from the remote server.

![screenshot](https://bohansec.com/assets/PEAK/10.PNG "screenshot")

Answer: 

134430fcb321.ngrok.io, upload_btlo.sh

### Attacker executes the downloaded script. What is the URL that the script connects to?
Narrow down the upload script, we see that script is trying to upload the local files to the attacker's remote server.

![screenshot](https://bohansec.com/assets/PEAK/11.PNG "screenshot")

Answer: 

https://134430fcb321.ngrok.io/upload

### What are the local files that have been downloaded in the malicious activity

![screenshot](https://bohansec.com/assets/PEAK/11.PNG "screenshot")

Answer:

/etc/passwd, /tmp/btlo.zip

### How many files did the attacker delete?
Filter 49512, we see the attacker removed 4 files.

![screenshot](https://bohansec.com/assets/PEAK/12.PNG "screenshot")

Answer:

4

That's all for today, hope you enjoyed the walkthrough and learned something new! I will keep posting more BTLO writeups as I make progress on the platform. Thank you for stopping by :) 




