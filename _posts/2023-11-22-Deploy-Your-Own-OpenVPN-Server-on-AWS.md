---
layout: post
title: Deploy Your Own OpenVPN Server on AWS
---
![Alt](https://bohansec.com/assets/vpn-blog/1.jpg)

After using Surfshark VPN and constantly experiencing their IP being blocked by Google reputation check, it sparked my interest to set up my own VPN server on AWS.

![image](https://bohansec.com/assets/vpn-blog/2.png "image")

As you can see in the screenshot, the majority of the Surfshark IP runs on Datacamp which is constantly abused by bad actors to conduct nefarious activities across the internet and has a poor reputation. I ran the IP 37.19.211.115 that I connected to through [AbuseIPDB](https://www.abuseipdb.com/){:target="_blank"} and [VirusTotal](https://www.virustotal.com/){:target="_blank"}, and both returned negative results.

![image](https://bohansec.com/assets/vpn-blog/5.png "image")

![image](https://bohansec.com/assets/vpn-blog/3.png "image")

![image](https://bohansec.com/assets/vpn-blog/4.png "image")

In this article, I will walk you through how to set up an OpenVPN access server on an AWS EC2 instance. 

If this is your first time using AWS, you need to sign up for an account on [aws.amazon.com](aws.amazon.com){:target="_blank"}. You will be logged in as an AWS root user however it’s the best practice to create a separate IAM account for day-to-day usage. 

![image](https://bohansec.com/assets/vpn-blog/6.png "image")

Here, since I am based in Canada, I will choose the “Canada ca-central-1” cloud region for the best VPN connection speed. But if you are located in another country, you can choose the cloud region that is closest to your location. 

![image](https://bohansec.com/assets/vpn-blog/7.png "image")

Next, let’s go to the AWS Marketplace and search for “OpenVPN” Amazon Machine Image. The first result “OpenVPN Access Server” is the result we are looking for. 

![image](https://bohansec.com/assets/vpn-blog/8.png "image")

Note that if you are a new user of AWS, you are eligible for 750 hours each month of free usage on a t2-micro instance which is the one we will use in this blog. You can find more information on the AWS free tier at [here](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all){:target="_blank"}. Please also note that the OpenVPN Access Server provides two concurrent connections on their free tier, so for my use case, I will just stick with the free tier to keep from being charged. 

![image](https://bohansec.com/assets/vpn-blog/9.png "image")

Next, let’s click on “Continue to Subscribe” with our OpenVPN Access Server then “Continue to Configuration”. Once you are on the “Configure this software” page, please choose the region that this EC2 instance should be launched, here I chose Canada. Then, go to “Continue to Launch”. 

![image](https://bohansec.com/assets/vpn-blog/10.png "image")

On the “Launch this software” page, change the “EC2 Instance Type” to “t2.micro” and “Security Group Settings” to the one OpenVPN has configured for you. I set the “Security Group” to the custom security group “OpenVPN-SG” I created earlier. Please refer to the OpenVPN official documentation with the following ports need that to be opened: [https://openvpn.net/vpn-server-resources/amazon-web-services-ec2-byol-appliance-quick-start-guide/](https://openvpn.net/vpn-server-resources/amazon-web-services-ec2-byol-appliance-quick-start-guide/){:target="_blank"}

> For the security group, we recommend using the default group for the marketplace instance but adjusting the sources for some ports to improve security. 

> TCP 22: For SSH to remotely administer your appliance. We recommend you restrict this port to trust IP addresses by entering a specific subnet in CIDR notation (e.g., 12.34.56.0/24 for a subnet or 11.22.33.44/32 for a single IP address).

> TCP 943: The Admin Web UI uses this port, which is also served on port 443 by default.

> TCP 945: The clustering functionality uses this port. If you don’t use this feature, you don’t need to open this port. If you do, ensure the cluster nodes can reach each other on their public addresses.

> TCP 443: For HTTPS, used by the Client Web UI, the interface where your users sign into the VPN server to retrieve client or config files. We recommend leaving this port open to the source as 0.0.0.0/0. The Admin Web UI is also default enabled on this port unless you turn off this setting. In multi-daemon mode, the OpenVPN TCP daemon shares this port with the Client Web UI, and your clients initiate TCP-based VPN sessions under this port number.

> UDP 1194: For the OpenVPN UDP port used by your clients to initiate UDP-based VPN sessions to the VPN server, the preferred way for clients to communicate. Keep this port open for all clients.

![image](https://bohansec.com/assets/vpn-blog/11.png "image")

If you want to further lock down the SSH access to your EC2 instance, you can set the SSH source to the specific IP range that is only allowed to connect from but here for simplicity, I will just use anywhere which is “0.0.0.0/0”.

If you want to SSH into the EC2 from your local machine, you also need to configure an SSH keypair and add it to the “Key Pair Settings”. I will use the key I have created earlier. Then, click on Launch. 

![image](https://bohansec.com/assets/vpn-blog/12.png "image")

To ensure the EC2 instance to stays on a static IP, let’s create an “Elastic IP address” and associate it with the EC2 instance we just launched. Please note that there will be no charge incurred if your elastic IP is assigned to a running EC2 instance. However, if you decide to stop your EC2 instance, please make sure to delete your elastic IP to avoid being charged. 

![image](https://bohansec.com/assets/vpn-blog/13.png "image")

![image](https://bohansec.com/assets/vpn-blog/14.png "image")

After I associate my elastic IP to my running EC2 instance, it will look like the following screenshot:

![image](https://bohansec.com/assets/vpn-blog/15.png "image")

Now, let’s connect to the EC2 instance through the Amazon SSH connect, which is a browser-based SSH connection experience. 

![image](https://bohansec.com/assets/vpn-blog/16.png "image")

Once we connect to the server, for simplicity, enter Yes and Default to all of the prompts, also remember to configure a strong password for your Admin WebUI account. The default account will be “openvpn”.

![image](https://bohansec.com/assets/vpn-blog/17.png "image")

After the configuration is finished, we can see my Admin portal is located at “https://3.99.38.103:943/admin" but you may have a different IP address than mine so just use whatever public IP address you have to log in. 

![image](https://bohansec.com/assets/vpn-blog/18.png "image")

![image](https://bohansec.com/assets/vpn-blog/19.png "image")

After you log into your admin portal, navigate to “VPN Settings” and toggle on the option “Should client Internet traffic be routed through the VPN?” to Yes. This will allow all the outbound traffic from your client to be encrypted and sent through the OpenVPN server before they are hit off to the internet. Additionally, we can toggle on “Have clients use specific DNS servers” and set the primary DNS server to CloudFlare DNS 1.1.1.1 and Google DNS 8.8.8.8 for improved privacy. For more information on why you shouldn’t use the default ISP DNS server, please refer to the following article:

[https://www.howtogeek.com/664608/why-you-shouldnt-be-using-your-isps-default-dns-server/#dns-is-not-private-without-doh](https://www.howtogeek.com/664608/why-you-shouldnt-be-using-your-isps-default-dns-server/#dns-is-not-private-without-doh){:target="_blank"}

![image](https://bohansec.com/assets/vpn-blog/20.png "image")

Optionally, you can also set a custom domain to the hostname and you can directly access your VPN with your domain instead of the IP address. 

Please refer to the following articles on how to configure an A record for your domain: 

[https://openvpn.net/vpn-server-resources/setting-up-your-openvpn-access-server-hostname/](https://openvpn.net/vpn-server-resources/setting-up-your-openvpn-access-server-hostname/){:target="_blank"}

[https://openvpn.net/vpn-server-resources/setting-up-your-openvpn-access-server-hostname/](https://www.namecheap.com/support/knowledgebase/article.aspx/9776/2237/how-to-create-a-subdomain-for-my-domain/){:target="_blank"}

Once you configured the A record then wait for 30 minutes to an hour for the DNS record to be propagated throughout the internet. 

![image](https://bohansec.com/assets/vpn-blog/21.png "image")

Next, depending on your operating system. we can download a VPN client at the following URL, and connect to the VPN server with the credentials we set earlier. 

[https://openvpn.net/client/](https://openvpn.net/client/){:target="_blank"}

![image](https://bohansec.com/assets/vpn-blog/22.png "image")

Doing a quick check on my public IP, it shows my IP is 3.99.38.103 which is the IP that I set up earlier in AWS. 

![image](https://bohansec.com/assets/vpn-blog/23.png "image")

![image](https://bohansec.com/assets/vpn-blog/24.png "image")

Let’s also do a speed test, while the speed dropped quite a bit compared to my original internet speed, it’s still a decent speed based on the internet test results.  

![image](https://bohansec.com/assets/vpn-blog/25.png "image")

We can also track down the client's real IP in the log Log Reports section, this will be especially helpful if you want to track your user’s connection activity or look out for any suspicious connections to your VPN server. 

![image](https://bohansec.com/assets/vpn-blog/26.png "image")

Lastly, let’s run a PCAP capture on my local network to verify my VPN connection, as you can see there are OpenVPN protocols in my network traffic which proves all my outbound traffic is being routed to my OpenVPN server. 

![image](https://bohansec.com/assets/vpn-blog/27.png "image")

To further harden your OpenVPN server, consider to review the following link:

[https://openvpn.net/vpn-server-resources/recommendations-to-improve-security-after-installation/](https://openvpn.net/vpn-server-resources/recommendations-to-improve-security-after-installation/){:target="_blank"}

Reference:

[https://openvpn.net/vpn-server-resources/recommendations-to-improve-security-after-installation/](https://openvpn.net/vpn-server-resources/amazon-web-services-ec2-byol-appliance-quick-start-guide/){:target="_blank"}

That’s all for today and thank you for reading. 




