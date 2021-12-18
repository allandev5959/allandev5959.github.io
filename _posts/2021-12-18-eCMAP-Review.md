---
layout: post
title: My eCMAP Review
---
![Alt](https://bohansec.com/assets/ecmap/1.jpg "Photo by Qijin Xu on Unsplash")

A few days ago, I passed the eCMAP from eLearnSecurity. In this review, I would like to share my experience and thoughts on this course, and a few small tips on how you could pass the exam. 

## What is eCMAP? 

eCMAP stands for eLearnSecurity Certified Malware Analysis Professional. By obtaining the eCMAP, your skills in the following areas will be assessed and certified:

* Run malware and track its activity

* Reverse Engineering and/or unpacking malware
    
* Ability to debug malware step-by-step
    
* Identify how the malware achieves obfuscation
    
* Identify C2 channels and what they are used for
    
* Bypass anti-analysis techniques
    
* Locate and analyze dropped and downloaded malware as well as persistence mechanisms

The course itself contains a wide range of topics and gives you a great foundation on malware analysis. The following topics are being covered in this course. 

* Introduction of Malware Analysis

* Static Analysis Techniques

* Assembly Language

* Behavior Analysis

* Debugging and Disassembly Techniques

* Obfuscation Techniques

The Introduction of Malware Analysis section gives you a great overview on what is malware analysis and reverse engineering, different malware family types, and some techniques that would be used in malware analysis. In the Static Analysis Techniques part, you will dive into the basics of malware analysis, identify different file types, and learn how to utilize IOC to create your Yara rules. It is crucial to understand the basics of assembling when reverse engineering malware, that is why a dedicated section goes towards Assembly Language. Maybe you would feel bored or overwhelmed by the assembly part, but I recommend you to at least go through it once so you will have a good understanding when you doing debugging and disassembly at the later stage. The behavior analysis section perhaps is one of my favorite sections throughout the entire course, which covers how to dynamically analyze a malware sample. Tools include but are not limited to Sysinternal, ProcDot, Regshot, ApateDNS, Wireshark. All these tools can aid your dynamic malware analysis effort. Essentially, what that means is that by launching the malware sample and monitoring its behavior with different tools inside a sandbox environment, we would know what this malware exactly does.

The last two sections of the course, Debugging and Disassembly Techniques as well Obfuscation Techniques, are fun yet challenging. In these two sections, not only you will learn how to use the IDA PRO and x64 debugger, but also you will learn how to effectively use them to analyze several real-world malware samples such as WannCry, Redaman, and Locky. In addition, unpacking and patching malware are also presented in these sections. I highly recommend you to go through labs for unpacking and patching as these techniques will be very helpful throughout the entire malware analysis process. 

## Experience and Tips 

I enjoyed the final exam so much and I feel I learned so much from the exam itself. You will have a week to finish the entire malware sample analysis process and draft the final report for your findings. I would say the whole exam process is not hard if you did all the labs. The labs will well prepare you to conquer the exam. Below are a few tips for your exam. 

* Take a lot of screenshots for your findings, screenshots should reflect the steps you take

* Set a goal and have a plan for your exam, define what you should do each day during your exam

* Doing all the labs and understanding why they are doing what they are doing

* Take breaks during the exam as you have plenty of times, a full week!

* Having a nice and clean report will help

* Use your labs as a reference if you are stuck somewhere

* Relax and enjoy the process, don’t rush through it because the exam is fun and you can learn a lot from it

## Credits and Suggestions  

In this final section, I want to give a big shutout to the course instructor Ali Hadi and the entire INE team. I think this course gives you a good introduction to malware analysis. Although it is not super in-depth and advanced like the Zero2Automated, it gives you a good start for learning about malware analysis. I say this course opened the door for me to malware analysis and reverse engineering. It prepared me well so I can analyze a malware sample on my own both statically and dynamically. However, I think it would be beneficial for this course or future courses to include more types of malware such as malicious documents or MalDocs. More frequent updates on the course content would be also appreciated, such as including more recent malware samples. 

## Conclusion

Overall, I highly recommend this course to anyone who wants to start their malware analysis journey or is just curious about malware analysis. 

![eCMAP](https://bohansec.com/assets/ecmap/2.PNG "eCMAP")




