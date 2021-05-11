---
layout: post
title: My eCPPT Review
---
![Alt](https://bohansec.com/assets/eCPPT/cover.jpg "Photo by Yannick Menard on Unsplash")

Since I passed my eCPPT exam, I would like to share my experience with the exam, and some helpful resources could help you ace the exam.

## What is eCPPT? 

eCPPT stands for "eLearnSecurity Certified Professional Penetration Tester". It is a highly hands-on Penetration Test exam designed to test your ability and knowledge to thoroughly assess a vulnerable network environment, as well as produce an excellent report. The following areas are covered and assessed in this course:

* Understanding a letter of engagement and the basics related to a penetration testing engagement

* Deep understanding of networking concepts

* Manual exploitation of Windows and Linux targets

* Performing vulnerability assessment of networks

* Using Metasploit for complex and multi-step exploitation of different systems and OS’s

* Web application Manual exploitation

* Ability in performing post-exploitation techniques

* Exploit development skills on x86 environment

* Outstanding reporting skills

## Exam Timeline

### April 12th

I started my exam on April 12 sometime around the afternoon. I felt good and confident that I can pass this exam sooner than a week. However, I quickly discovered that is not the case. The initial foothold is pretty straightforward, and I was being able to quickly get that done without much trouble. However, things became a challenge when there is post-exploitation and pivoting involved. I was stuck on pivoting for a while around 2 to 3 hours. And read some blogs and went back to the course material to get some ideas. Later on that night, I was being able to move to the second machine with some research on google and tried multiple different payloads. Around 9 pm, I wrapping up for the night. 

### April 13

I continued my exam in the afternoon. After some enumeration on the second machine, I found something interesting to lead me to move further. However, things got a bit challenging when comes to the exploit development. Especially I wasn't able to wrap my head around how to deliver my exploit when I have to pivot. And, I spend around 1 or 2 hours to find a proper working image as my testing environment. I was really stuck at this exploit thing, and I was getting nervous and thought I would fail. My exploit for some reason it just not working. Around 9 pm that night I called a wrap and went to sleep.

### April 14

Wednesday, I wasn’t able to move forward. And I was still struggling on how to get my exploit work. I was not sure why my code isn’t working. Later on that day, I did some scans on the network, and discovered few more machines, collected some basic information about them, and called a wrap.

### April 15

Things getting a bit nervous at this moment, I knew I was in serious trouble if I can't figure this exploit development thing out, and I thought I would fail. Continue from the previous day’s discovery, I was able to get a few more access to the network. However, I was still desperately in trouble with the exploit development thing.

### April 16

I got up early on Friday, put everything I supposed to do aside, brewed a couple of cups of coffee, and decided to knock out this exploit development thing. After some research, I decided to totally abandon my current exploit and just redo it from scratch. Because I feel there must be something wrong with my exploit. After some google-fu on blogs, I was able to find a way to work my exploit out. And tested it with the Metasploit in my local environment, it worked!! Happy I can move forward now. With the working exploit, I called for the day and went to sleep.

### April 17

Everything became clear and bright after my exploit worked. Although the way to get into the final stage is a bit tricky, with proper enumeration, course material, and a bit of google-fu, I was able to obtain root access at the final network around 1 PM. On the same day around 9 PM, I turned my report in and went to sleep.

### April 23

Unfortunately, I received an email says I failed my exam. I felt kinda disappointed and ready to give it another try. Since eLearnSecurity provides you excellent feedback, I was able to fix my report and submit it on the same day.

### May 10

After 20 days of waiting, I received an email informs me I am an eCPPT now!

## Tips and Resources perhaps can make your life easier

* Learn to pivot, Pivoting is the God for the exam. If you understand pivoting, you already halfway to success
 
* Post-Exploitation is important
 
* Self-doubt is normal, but be persistent and DO NOT GIVE UP
 
* Don't over complicated things, look what's in front of you, if you think that's the way, I am sure 90% chance that's the way to go
 
* If one exploit or payload not work, try another till it works

* Do TryHackMe Gatekeeper for the Exploit Development, the THM Gatekeeper is the Godsend to me
 
* Sleep well, relax, hacking and repeat
 
* Google is your friend
 
* RDP is helpful
 
* Metasploit all the way, makes your life easier, of course, for this exam

* Get a working Windows machine VM before the exam

* Focus on your report writing, all the steps you performed the attack should be documented in detail

* Take good notes


## Some Resources potentially helpful

1. [GateKeeper Walkthrough](https://www.youtube.com/watch?v=ALbNTOOqmsA){:target="_blank"}

2. [Gatekeeper Walkthrough](https://pencer.io/ctf/ctf-thm-gatekeeper/){:target="_blank"}

3. [Pivoting](https://pentest.blog/explore-hidden-networks-with-double-pivoting/){:target="_blank"}

4. [Pivoting](https://www.offensive-security.com/metasploit-unleashed/pivoting/){:target="_blank"}

5. [Post-Exploitation](https://laptrinhx.com/credential-dumping-applications-3748591012/){:target="_blank"}

6. [Report-Template](https://github.com/hmaverickadams/TCM-Security-Sample-Pentest-Report){:target="_blank"}

## Conclusion 

I really enjoyed eCPPT exam. It is a highly hands-on exam that requires practical knowledge with pivoting, post-exploitation, and good communication skills from the report. I highly recommend anyone to take this exam if you are looking for something fun and challenge to do. Thank you eLearnSecurity and INE for your wonderful course material, exam, and supportive community!

![eCPPT](https://bohansec.com/assets/eCPPT/1.PNG "eCPPT")




